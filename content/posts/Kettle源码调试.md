---
title: "Kettle源码调试"
date: 2019-05-30T17:46:01+08:00
draft: false
tags: 
  - Kettle
---
## Kettle源码调试
继上次基于kettle源码编译Kettle之后,因为要开发kettle插件,所以尝试了下kettle源码的调试。

### 调试步骤
##### 调试环境
- 我使用的还是idea
- kettle源码版本9.0
##### 步骤
1.clone kettle源码保留以下文件即可：

![1](https://raw.githubusercontent.com/zy528/blog/hugo/static/images/kettle_debug/1.png)

- kettle的源码结构
- kettle-core：kettle的核心模块，包括一些数据处理等。
- kettle-dbdialog：kettle数据库连接界面逻辑。
- kettle-engine：kettle的引擎，负责执行kettle的具体作业和转换的逻辑，并会调用core模块。
- kettle-ui：用户界面模块，包括用户界面显示的xul文件，通过后端代码编写的Dialog以及国际化等。

根目录下的pom.xml 的modules对应进行修改只保留core、dbdialog、engine、ui

将编译好的kettle ```data-integration\ui```

2.目录下的所有文件复制到源码ui文件目录下：

![2](https://raw.githubusercontent.com/zy528/blog/hugo/static/images/kettle_debug/2.png)

3.因为我是在windows环境下调试的，所以要修改```ui/pom.xml```
里面的依赖，之前是linux的依赖修改为以下代码：

```
<dependency>
      <groupId>org.eclipse.swt</groupId>
      <artifactId>org.eclipse.swt.win32.win32.x86_64</artifactId>
</dependency>
```

4.运行```ui\src\main\java\org\pentaho\di\ui\spoon ```目录下的```Spoon.java```出现kettle可操作的界面即调试启动成功
![3](https://raw.githubusercontent.com/zy528/blog/hugo/static/images/kettle_debug/3.png)

5.之前在调试的时候遇到一个很坑的问题,启动页面都能显示，但操作界面就是不出来，控制台报错：```
java.lang.OutOfMemoryError: Java heap space kettle```
不管我怎么调运行内存都没用，后来打断点调试，发现是在加载
```db.cache-Unknown ```配置文件的时候内存溢出，于是清理了```C:\Users\Administrator\.kettle```目录下的文件，就可以正常启动了。

### 总结
调试kettle是为了进行转换的插件开发，接下来会介绍kettle准换插件的开发流程。
kettle的相关知识可以参考https://blog.csdn.net/inthat/article/details/84867066#_408
