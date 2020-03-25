---
layout: post
title: 从 pthread_t 获得 PID 和 TID
date: 2017-11-24 10:37:23
categories: UMUTech
tags:
- c
- dev
- linux
---
pthread_t 其实是一个结构体指针，里面包含了 TID 和 PID，找出它的偏移量就行。在 Ubuntu 16.04.3 x64 测试通过。

```c
int get_tid_from_pthread(pthread_t t)
{
	struct pthread_fake {
		void *nothing[90];
		pid_t tid;
	};

	struct pthread_fake* f = (struct pthread_fake*)t;
	return f->tid;
}

int get_pid_from_pthread(pthread_t t)
{
	struct pthread_fake {
		void *nothing[90];
		pid_t tid;
		pid_t pid;
	};

	struct pthread_fake* f = (struct pthread_fake*)t;
	return f->pid;
}
```
