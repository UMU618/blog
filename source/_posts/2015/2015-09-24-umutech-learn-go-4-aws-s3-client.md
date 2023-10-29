---
layout: post
title: 学习 go 语言【4】简单的 AWS S3 客户端
date: 2015-09-24 17:24:28
categories: UMUTech
tags:
- dev
- go
---
## 需求

公司搞了一套兼容 Amazon S3 的云存储系统，用 C++ 写客户端很蛋疼，UMU 决定还是用 go 写一个。

## 开发过程

### 1. 找库

先找一个靠谱的开源项目，运行以下命令安装：

```sh
go get github.com/mitchellh/goamz
```

它内部还用到 `github.com/vaughan0/go-ini`。

### 2. 通过示例代码学习

看一下库带的例子：

```go
package main

import (
  "github.com/mitchellh/goamz/aws"
  "github.com/mitchellh/goamz/s3"
  "log"
  "fmt"
)

func main() {
  auth, err := aws.EnvAuth()
  if err != nil {
    log.Fatal(err)
  }
  client := s3.New(auth, aws.USEast)
  resp, err := client.ListBuckets()

  if err != nil {
    log.Fatal(err)
  }

  log.Print(fmt.Sprintf("%T %+v", resp.Buckets[0], resp.Buckets[0]))
}
```

### 3. 实现

要改的地方不多：

- 认证方式可以改为 aws.GetAuth，但这样容易暴露 AccessKey、SecretKey，所以 UMU 下面贴出的代码还是使用 aws.EnvAuth()。

- aws.USEast 改为我们自己的。

- UMU 尝试添加列出所有文件的功能。

## 代码

```go
package main

import (
	"fmt"
	"github.com/mitchellh/goamz/aws"
	"github.com/mitchellh/goamz/s3"
	"log"
)

func main() {
	auth, err := aws.EnvAuth()
	if err != nil {
		log.Fatal(err)
	}

	var cnc = aws.Region{
		"cnc", // Name
		"",    // EC2Endpoint
		"http://s3.bj.xs3cnc.com", // S3Endpoint
		"",    // S3BucketEndpoint
		false, // S3LocationConstraint
		false, // S3LowercaseBucket
		"",    // SDBEndpoint
		"",    // SNSEndpoint
		"",    // SQSEndpoint
		"",    // IAMEndpoint
		"",    // ELBEndpoint
		"",    // AutoScalingEndpoint
		"",    // RdsEndpoint
		"",    // Route53Endpoint
	}
	client := s3.New(auth, cnc)
	resp, err := client.ListBuckets()
	if err != nil {
		log.Fatal(err)
	}

	for _, b := range resp.Buckets {
		fmt.Printf("Bucket Name: %s\n", b.Name)
		bc, err := b.GetBucketContents()
		if err == nil {
			for _, key := range *bc {
				fmt.Printf("\t%s, %s, %d, %s, %s, %s\n",
					key.Key, key.LastModified, key.Size, key.StorageClass,
					key.Owner.ID, key.Owner.DisplayName)
			}
		}
	}
}
```
效果如下：

![xs3cnc](/images/2015/20150924-1.png)

参照对象：

![S3 Browser](/images/2015/20150924-2.jpg)
