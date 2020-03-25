---
layout: post
title: MongoDB Shard ID hash 算法 std::hash 的跨平台性
date: 2018-04-24 23:18:11
categories: UMUTech
tags:
- dev
- cpp
- mongodb
---
```cpp
#include <functional>
#include <iomanip>
#include <iostream>
#include <string>


int main()
{
    std::string str = "Meet the new boss...";
    std::size_t str_hash = std::hash<std::string>{}(str);
    std::cout << "hash(" << std::quoted(str) << ") = " << str_hash << std::endl;

    str = "Meet the new boss..;";
    str_hash = std::hash<std::string>{}(str);
    std::cout << "hash(" << std::quoted(str) << ") = " << str_hash << std::endl;

    str = "Meet the new boss../";
    str_hash = std::hash<std::string>{}(str);
    std::cout << "hash(" << std::quoted(str) << ") = " << str_hash << std::endl;

    str = "Meet the new boss..,";
    str_hash = std::hash<std::string>{}(str);
    std::cout << "hash(" << std::quoted(str) << ") = " << str_hash << std::endl;

    return 0;
}
```
Windows,  VS 2017 的结果：

> hash("Meet the new boss...") = 5935324269489717502
>
> hash("Meet the new boss..;") = 5935347359233909933
>
> hash("Meet the new boss../") = 5935325369001345713
>
> hash("Meet the new boss..,") = 5935322070466461080

Ubuntu 16.04,  g++ 5.4.0 20160609 的结果：

> hash("Meet the new boss...") = 10656026664466977650
>
> hash("Meet the new boss..;") = 12509209616339026574
>
> hash("Meet the new boss../") = 6552276210272946664
>
> hash("Meet the new boss..,") = 15639609178671340058

还好我们不会在生产环境，使用 Windows 部署 MongoDB……

```
std::size_t ShardId::Hasher::operator()(const ShardId& shardId) const {
     return std::hash<std::string>()(shardId._shardId); 
} 
```
详见：[https://github.com/mongodb/mongo/blob/master/src/mongo/s/shard_id.cpp](https://github.com/mongodb/mongo/blob/master/src/mongo/s/shard_id.cpp)


这个 std::hash 在 x86 和 x64 下都不一样，所以，让我们看看 MongoDB 如何解决这个问题：

MongoDB 3.4 no longer supports 32-bit x86 platforms.

好样的！
