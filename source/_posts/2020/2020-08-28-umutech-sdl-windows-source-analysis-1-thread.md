---
layout: post
title: SDL/Windows 代码分析【1】thread
date: 2020-08-28 15:11:55
categories: UMUTech
tags:
- c
- debug
- dev
- windows
---
## 1. SDL_semaphore

代码：src\thread\windows\SDL_syssem.c

别名：SDL_sem

```c
typedef struct SDL_semaphore SDL_sem;
```

基于 WinAPI 匿名 Semaphore 封装。MaximumCount 硬编码为 32 * 1024。

```c
struct SDL_semaphore
{
    HANDLE id;
    LONG count;
};


/* Create a semaphore */
SDL_sem *
SDL_CreateSemaphore(Uint32 initial_value)
{
    SDL_sem *sem;

    /* Allocate sem memory */
    sem = (SDL_sem *) SDL_malloc(sizeof(*sem));
    if (sem) {
        /* Create the semaphore, with max value 32K */
#if __WINRT__
        sem->id = CreateSemaphoreEx(NULL, initial_value, 32 * 1024, NULL, 0, SEMAPHORE_ALL_ACCESS);
#else
        sem->id = CreateSemaphore(NULL, initial_value, 32 * 1024, NULL);
#endif
        sem->count = initial_value;
        if (!sem->id) {
            SDL_SetError("Couldn't create semaphore");
            SDL_free(sem);
            sem = NULL;
        }
    } else {
        SDL_OutOfMemory();
    }
    return (sem);
}
```

## 2. SDL_mutex

代码：src\thread\windows\SDL_sysmutex.c

基于 WinAPI CriticalSection 封装。SpinCount 硬编码为 2000，即在多处理器系统上，如果无法立刻进入临界区，则会自旋最多 2000 次，然后等待 CriticalSection 内部关联的信号量。只要在自旋过程中其它线程退出临界区，则无需进入等待状态。这么做是提高效率，自旋时当前线程还占着 CPU，如果进入等待状态，就是交出 CPU 时间片了，而 CPU 调度是个消耗型操作。

```c
struct SDL_mutex
{
    CRITICAL_SECTION cs;
};

/* Create a mutex */
SDL_mutex *
SDL_CreateMutex(void)
{
    SDL_mutex *mutex;

    /* Allocate mutex memory */
    mutex = (SDL_mutex *) SDL_malloc(sizeof(*mutex));
    if (mutex) {
        /* Initialize */
        /* On SMP systems, a non-zero spin count generally helps performance */
#if __WINRT__
        InitializeCriticalSectionEx(&mutex->cs, 2000, 0);
#else
        InitializeCriticalSectionAndSpinCount(&mutex->cs, 2000);
#endif
    } else {
        SDL_OutOfMemory();
    }
    return (mutex);
}
```

## 3. SDL_cond

代码：src\thread\windows\SDL_syscond.c

基于 SDL_mutex 和 SDL_sem 封装。

```c
struct SDL_cond
{
    SDL_mutex *lock;
    int waiting;
    int signals;
    SDL_sem *wait_sem;
    SDL_sem *wait_done;
};

/* Create a condition variable */
SDL_cond *
SDL_CreateCond(void)
{
    SDL_cond *cond;

    cond = (SDL_cond *) SDL_malloc(sizeof(SDL_cond));
    if (cond) {
        cond->lock = SDL_CreateMutex();
        cond->wait_sem = SDL_CreateSemaphore(0);
        cond->wait_done = SDL_CreateSemaphore(0);
        cond->waiting = cond->signals = 0;
        if (!cond->lock || !cond->wait_sem || !cond->wait_done) {
            SDL_DestroyCond(cond);
            cond = NULL;
        }
    } else {
        SDL_OutOfMemory();
    }
    return (cond);
}
```

SDL_CondWaitTimeout 实现较长，本文忽略。重点是：为了避免死锁，它进入等待前，会先解锁第二个参数 mutex。如果不这么做，其它线程也要 Lock 这个 mutex 就会发生死锁。

以下代码是典型用法，线程 A **先**进入临界区后，SDL_CondWait（内部调用 SDL_CondWaitTimeout）会调用 `SDL_UnlockMutex(lock);` 使得线程 B 可以进入临界区调用 `SDL_CondSignal(cond);`。

```c
// Typical use

// Thread A:
SDL_LockMutex(lock);
while (!condition) {
    SDL_CondWait(cond, lock);
}
SDL_UnlockMutex(lock);

// Thread B:
SDL_LockMutex(lock);
condition = true;
SDL_CondSignal(cond);
SDL_UnlockMutex(lock);
```

## 4. SDL_Atomic

代码：src\atomic\SDL_atomic.c

基于 \_Interlocked API 封装。此类原子操作一般底层实现都是相应平台的汇编指令（比如 x86 平台是 lock cmpxchg 之类），但在不同平台下会有不同的封装集，所以 SDL_atomic.c 里有很多平台相关的宏判断。

## 5. SDL_MemoryBarrier

代码：src\atomic\SDL_atomic.h

内存屏障。参考文章：[Acquire and Release Semantics](https://preshing.com/20120913/acquire-and-release-semantics/)。

在 Windows x86 环境下等价于 \_ReadWriteBarrier：

```c
void _ReadWriteBarrier(void);
#pragma intrinsic(_ReadWriteBarrier)
#define SDL_CompilerBarrier()   _ReadWriteBarrier()

/* This is correct for the x86 and x64 CPUs, and we'll expand this over time. */
#define SDL_MemoryBarrierRelease()  SDL_CompilerBarrier()
#define SDL_MemoryBarrierAcquire()  SDL_CompilerBarrier()
```

- **Acquire semantics** is a property that can only apply to operations that read from shared memory, whether they are read-modify-write operations or plain loads. The operation is then considered a **read-acquire**. Acquire semantics prevent memory reordering of the read-acquire with any read or write operation that follows it in program order.

- **Release semantics** is a property that can only apply to operations that write to shared memory, whether they are read-modify-write operations or plain stores. The operation is then considered a **write-release**. Release semantics prevent memory reordering of the write-release with any read or write operation that precedes it in program order.

生硬的翻译如下：

- 获取语义是一个属性，它只能应用于从共享内存读取的操作，无论是“读取-修改-写入”操作还是普通加载。该操作将被视为“读取获取”。获取语义可防止“读取获取”和它之后的读取或写入操作发生内存重新排序。

- 释放语义是一个属性，它只能应用于写入共享内存的操作，无论是“读取-修改-写入”操作还是普通存储。该操作将被视为“写入释放”。释放语义可防止“写入释放”和它之前，它之前的任何读取或写入操作发生内存重新排序。

以 x86 内存模型为例说明：

- Loads are not reordered with other loads.
- Stores are not reordered with other stores.
- Stores are not reordered with older loads.
- Loads **may be** reordered with older stores to different locations.

因为 store-load 可以被重排，所以 x86 不是顺序一致。但是其他三种读写顺序不能被重排，所以 x86 是 acquire/release 语义。

aquire 语义：load 之后的读写操作无法被重排至 load 之前。即 load-load, load-store 不能被重排。

release 语义：store 之前的读写操作无法被重排至 store 之后。即 load-store, store-store 不能被重排。

## 6. SDL_TLSData

意义：TLS，即 Thread Local Storage（线程局部存储）。

代码：src\thread\SDL_thread_c.h 和 src\thread\windows\SDL_systls.c

基于 SDL_Atomic、SDL_MemoryBarrier 和 WinAPI Tls API 封装。

以下结构体包含一个析构函数的指针，非空时，SDL_TLSCleanup() 会调用它。

```c
/* This is the system-independent thread local storage structure */
typedef struct {
    unsigned int limit;
    struct {
        void *data;
        void (SDLCALL *destructor)(void*);
    } array[1];
} SDL_TLSData;
```

## 7. SDL_Thread

代码：src\thread\SDL_thread.c 和 src\thread\windows\SDL_systhread.c

创建线程的 API 是 SDL_CreateThread 和 SDL_CreateThreadWithStackSize，导出函数 SDL_CreateThread 的定义如下，记为【X】：

```c
SDL_DYNAPI_PROC(SDL_Thread*,SDL_CreateThread,(SDL_ThreadFunction a, const char *b, void *c, pfnSDL_CurrentBeginThread d, pfnSDL_CurrentEndThread e),(a,b,c,d,e),return)
```

下面会有递归展开宏的过程。首先，用 SDL_DYNAPI_PROC 的定义：

```c
#define SDL_DYNAPI_PROC(rc,fn,params,args,ret) \
    static rc SDLCALL fn##_DEFAULT params { \
        SDL_InitDynamicAPI(); \
        ret jump_table.fn args; \
    }
```

展开【X】得到：

```c
static SDL_Thread* __cdecl SDL_CreateThread(SDL_ThreadFunction a, const char *b, void *c, pfnSDL_CurrentBeginThread d, pfnSDL_CurrentEndThread e) {
    SDL_InitDynamicAPI();
    return jump_table.SDL_CreateThread(a,b,c,d,e);
}
```

其中 jump_table.SDL_CreateThread 是【X】被 SDL_dynapi_procs.h 的：

```c
/* The jump table! */
typedef struct {
    #define SDL_DYNAPI_PROC(rc,fn,params,args,ret) SDL_DYNAPIFN_##fn fn;
    #include "SDL_dynapi_procs.h"
    #undef SDL_DYNAPI_PROC
} SDL_DYNAPI_jump_table;

/* The actual jump table. */
static SDL_DYNAPI_jump_table jump_table = {
    #define SDL_DYNAPI_PROC(rc,fn,params,args,ret) fn##_DEFAULT,
    #include "SDL_dynapi_procs.h"
    #undef SDL_DYNAPI_PROC
};
```

展开得到，为：

```c
typedef struct {
    /* ... */
    SDL_DYNAPIFN_SDL_CreateThread SDL_CreateThread;
    /* ... */
} SDL_DYNAPI_jump_table;

static SDL_DYNAPI_jump_table jump_table = {
    /* ... */
    SDL_CreateThread_DEFAULT,
    /* ... */
};
```

【X】又被 SDL_dynapi.c 的 initialize_jumptable 函数的：

```c
    /* Init our jump table first. */
    #define SDL_DYNAPI_PROC(rc,fn,params,args,ret) jump_table.fn = fn##_REAL;
    #include "SDL_dynapi_procs.h"
    #undef SDL_DYNAPI_PROC
```

展开为：

```c
    /* ... */
    jump_table.SDL_CreateThread = SDL_CreateThread_REAL;
    /* ... */
```

所以，调用 SDL_CreateThread 最终调用的就是 SDL_CreateThread_REAL，又由于 `src\dynapi\SDL_dynapi_overrides.h` 中的： 

```c
#define SDL_CreateThread SDL_CreateThread_REAL
```

所以调用的是 src\thread\SDL_thread.c 中的：

```c
#ifdef SDL_PASSED_BEGINTHREAD_ENDTHREAD
DECLSPEC SDL_Thread *SDLCALL
SDL_CreateThread(int (SDLCALL * fn) (void *),
                 const char *name, void *data,
                 pfnSDL_CurrentBeginThread pfnBeginThread,
                 pfnSDL_CurrentEndThread pfnEndThread)
#else
DECLSPEC SDL_Thread *SDLCALL
SDL_CreateThread(int (SDLCALL * fn) (void *),
                 const char *name, void *data)
#endif
{
    /* !!! FIXME: in 2.1, just make stackhint part of the usual API. */
    const char *stackhint = SDL_GetHint(SDL_HINT_THREAD_STACK_SIZE);
    size_t stacksize = 0;

    /* If the SDL_HINT_THREAD_STACK_SIZE exists, use it */
    if (stackhint != NULL) {
        char *endp = NULL;
        const Sint64 hintval = SDL_strtoll(stackhint, &endp, 10);
        if ((*stackhint != '\0') && (*endp == '\0')) {  /* a valid number? */
            if (hintval > 0) {  /* reject bogus values. */
                stacksize = (size_t) hintval;
            }
        }
    }

#ifdef SDL_PASSED_BEGINTHREAD_ENDTHREAD
    return SDL_CreateThreadWithStackSize(fn, name, stacksize, data, pfnBeginThread, pfnEndThread);
#else
    return SDL_CreateThreadWithStackSize(fn, name, stacksize, data);
#endif
}
```

可见 `SDL_CreateThread` 调用了 `SDL_CreateThreadWithStackSize`，而 `SDL_CreateThreadWithStackSize` 又调用 src\thread\windows\SDL_systhread.c 中的 `SDL_SYS_CreateThread`，因为 Windows 平台有 `_beginthreadex` 和 `_endthreadex`，所以最后是调用 `_beginthreadex`：

```c
    /* thread->stacksize == 0 means "system default", same as win32 expects */
    if (pfnBeginThread) {
        unsigned threadid = 0;
        thread->handle = (SYS_ThreadHandle)
            ((size_t) pfnBeginThread(NULL, (unsigned int) thread->stacksize,
                                     RunThreadViaBeginThreadEx,
                                     pThreadParms, flags, &threadid));
    } else {
        DWORD threadid = 0;
        thread->handle = CreateThread(NULL, thread->stacksize,
                                      RunThreadViaCreateThread,
                                      pThreadParms, flags, &threadid);
    }
```

其中 `RunThreadViaBeginThreadEx` 实际上是调用 `RunThread`:

```c
static DWORD
RunThread(void *data)
{
    pThreadStartParms pThreadParms = (pThreadStartParms) data;
    pfnSDL_CurrentEndThread pfnEndThread = pThreadParms->pfnCurrentEndThread;
    void *args = pThreadParms->args;
    SDL_free(pThreadParms);
    SDL_RunThread(args);
    if (pfnEndThread != NULL)
        pfnEndThread(0);
    return (0);
}

static DWORD WINAPI
RunThreadViaCreateThread(LPVOID data)
{
  return RunThread(data);
}

static unsigned __stdcall
RunThreadViaBeginThreadEx(void *data)
{
  return (unsigned) RunThread(data);
}
```

从代码可见 `RunThread` 调用 `SDL_RunThread`，而 `SDL_RunThread` 内部由 `SDL_TLSCleanup()` 来调用析构函数：

```c
void
SDL_RunThread(void *data)
{
    thread_args *args = (thread_args *) data;
    int (SDLCALL * userfunc) (void *) = args->func;
    void *userdata = args->data;
    SDL_Thread *thread = args->info;
    int *statusloc = &thread->status;

    /* Perform any system-dependent setup - this function may not fail */
    SDL_SYS_SetupThread(thread->name);

    /* Get the thread id */
    thread->threadid = SDL_ThreadID();

    /* Wake up the parent thread */
    SDL_SemPost(args->wait);

    /* Run the function */
    *statusloc = userfunc(userdata);

    /* Clean up thread-local storage */
    SDL_TLSCleanup();

    /* Mark us as ready to be joined (or detached) */
    if (!SDL_AtomicCAS(&thread->state, SDL_THREAD_STATE_ALIVE, SDL_THREAD_STATE_ZOMBIE)) {
        /* Clean up if something already detached us. */
        if (SDL_AtomicCAS(&thread->state, SDL_THREAD_STATE_DETACHED, SDL_THREAD_STATE_CLEANED)) {
            if (thread->name) {
                SDL_free(thread->name);
            }
            SDL_free(thread);
        }
    }
}
```
