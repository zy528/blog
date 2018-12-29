---
title: "开源ETL工具Kettle编译安装"
date: 2018-12-29T15:00:01+08:00
draft: false
tags: 
  - Kettle
---

​	做数据开发一直都在用的一款开源的[Etl](https://baike.baidu.com/item/ETL/1251949?fr=aladdin)工具Kettle,今天记录一下基于[kettle](https://github.com/pentaho/pentaho-kettle)的源码,编译、安装过程。

​	kettle是用Java编写的一款etl工具，采用了Maven管理，官方的教程是在cmd里面执行maven命令编译，秉承着对工具的热爱，我尝试着在idea里面编译（其实和命令安装是一样的）。

​	kettle也有编译好的包，可以直接下载使用。[下载链接](https://sourceforge.net/projects/pentaho/files/latest/download?aliId=137249511)

### 编译步骤

##### 环境：

- Maven, version 3+
- Java JDK 1.8
- This [settings.xml](https://raw.githubusercontent.com/pentaho/maven-parent-poms/master/maven-support-files/settings.xml) in your /.m2 directory
- IDEA

------

##### 步骤：

1. 从github上clone源码到idea.https://github.com/pentaho/pentaho-kettle.git

![1](https://raw.githubusercontent.com/zy528/blog/hugo/static/images/1.png)

然后就是maven漫长的下载包的环节，kettle涉及到的依赖包很多很大，真是个漫长的过程。。。

   2.在漫长的下载过程后点击下图的Terminal窗口，输入命令

```
mvn install -DskipTests
```

![1546075627948](https://raw.githubusercontent.com/zy528/blog/hugo/static/images/2.png)

如果不出问题的话最后结果,如上图所示 BUILD SUCCESS

3.最后在 assemblies\client\target 目录下有一个zip格式的文件就是编译后的kettle

![1546076111642](https://raw.githubusercontent.com/zy528/blog/hugo/static/images/3.png)

4.解压该文件 运行Spoon.bat就可以使用kettle了 :raised_hands:

### 结语

​	说实话等待kettle下载依赖的过程实在有点久，如果没有特殊的二次开发的需求的话，还是建议直接下载官方编译好的包。

​	不过源码编译还是有一点好就是最新的kettle，我编译出来的是kettle9.0的版本，我去官网下载只能下到8.0的包（功能上没有太大的区别）。这个就看自己的喜好了。

​	如果编译的过程中出错了，最好检查下自己的maven环境。

​	kettle的使用我想在后面文章里记录下自己使用kettle遇到的问题和一些kettle的使用技巧，kettle的编译和安装就到这里了。

