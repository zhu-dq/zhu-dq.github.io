---
layout: post
title:  C++ 调用Python
date:   2015-08-24 15:48:00
category: "C++"
---
# C++ 调用Python

****
## 前提
> 已经配好Python环境。在c++项目中引入 <br> 
> **包含目录**：C:\Python27\include <br>
> **库目录**：  C:\Python27\libs  <br>
> 在源代码加入头文件 : #include <Python.h>

## 通用撸码步骤
- **第一步：初始化Python**

	![这里写图片描述](http://img.blog.csdn.net/20150824153740528)


- **第二步：引入Python模块**

	![这里写图片描述](http://img.blog.csdn.net/20150824153812581)

- **第三步：调用模块里的函数**
    
	![这里写图片描述](http://img.blog.csdn.net/20150824153838372)

- **第四步：调用函数并获取返回值**

	![这里写图片描述](http://img.blog.csdn.net/20150824153903525)

- **第五步：关闭python**

	![这里写图片描述](http://img.blog.csdn.net/20150824153927398)

##	解析复杂返回值

- **list的解析**

	![这里写图片描述](http://img.blog.csdn.net/20150824154011303)

- **Tuple的解析**

	![这里写图片描述](http://img.blog.csdn.net/20150824154046156)

##	实际撸码思路

> 因为调python大部分都是可重用的代码，所以我把每个步骤封装成宏的形式，这样可以使代码更简洁。

- **写成宏的形式**

	![这里写图片描述](http://img.blog.csdn.net/20150824154137097)

- **使用宏**


	![这里写图片描述](http://img.blog.csdn.net/20150824154155150)

>  **PS**: 我把第一步和第五步分别放在了构造函数和析构函数里，这里没有体现 <br>

## 实例

>  这个是我写的一个简单把Neo4j的python接口封装成c++接口的简单demo。<br>
	<https://github.com/zhu-dq/Neo4jCPP.git>
