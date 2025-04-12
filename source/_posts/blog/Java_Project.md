---
title: "记录我的第一个Java项目"

date: 2020-02-13T05:00:00Z
# image: "/images/placeholder.png"
categories: ["Java"]
author: "Qingfeng Zhang"
---

最近因为慕课网可以1元购买一些优质的视频，其中就有一个《新版本微服务时代Spring
Boot企业微信点餐系统》，是一个Java项目，正好一直想做一个Java项目，于是就想跟着学习，这个过程比较艰难，在这里记录一些学习到的一些东西以及踩过的一些坑。

根据课程的说明，这是一个前后端分离的项目，前后端使用的技术如下图所示（虽然我现在还看不懂，但是图先放上）：  
<img src="images/Java_Project/1.jpg" width="50%" height="50%" >
使用的关于SpringBoot的技术如下图所示：  
<img src="images/Java_Project/2.jpg" width="50%" height="50%" >
使用的一些软件以及版本如下图：
<img src="images/Java_Project/3.jpg" width="50%" height="50%" >
我并没有完全按照上述的版本进行操作的。

# 1.环境搭配

## 1.1 MySQL的安装配置

一开始下载的MySQL是zip压缩包，并且是最新版8.0.19的，没有安装程序，结果配置的时候一直出错，好像是需要安装Microsoft Visual
C++对应的版本，我的版本有问题，就没再折腾了。  
后来直接下载msi installer版的MySQL，版本是5.7.29，最后成功安装。

## 1.2 Java环境配置

因为本机上是没有Java环境的，就在官网下载1.8.0_241版本的JDK，配置完环境变量后可以在终端运行`java`和`javac`命令。

## 1.3 IDEA的安装配置

IDEA和PyCharm是同一家的，所以用起来还是有点熟悉的，不过进去官网是真的慢，要求下载专业版的IDEA，版本是2019.3.3，完成安装即可。

# 2.创建项目

使用IDEA新建project，然后选择Spring Initializr,然后设置项目名称，其他默认，最后勾选web即可；  
新建的SpringBoot项目是一些文件夹组成的目录结构和主类的java文件，但是没有找到这个文件的运行方式，就很奇怪，找了一下发现可以用maven启动这个项目，点击右上角的run/debug
configrations添加maven，在working directory里填写项目的路径，在command line里填写`spring-
boot:run`，保存就可以用maven运行这个项目，但是一开始运行的时候一直在下载很多jar包，非常慢，中途停掉了对maven进行换源：  
找到`maven\lib\maven3\conf\setting.xml`文件，在标签`<mirrors>`内添加以下内容：

```xml
<mirror>  
    <id>nexus-aliyun</id>  
    <mirrorOf>central</mirrorOf>  
    <name>Nexus aliyun</name>  
    <url>http://maven.aliyun.com/nexus/content/groups/public</url>  
</mirror>  
```
  
再次用maven启动项目，这次项目文件夹有一些变化，右上角的configurations的也多了通过SpringBoot运行项目的选项，通过SpringBoot运行的结果如下：  
![](images/Java_Project/4.jpg)  
在浏览器访问地址`localhost:8080`显示的结果：
<img src="images/Java_Project/5.jpg" width="50%" height="50%" >
在这之后在慕课上视频学习了maven和springboot的一些基础知识，对前面创建的springboot项目又有了更深的了解，之后可能会做一些总结；

# 3.买家端类目

前面三章的课程讲了开发环境的搭建、数据库表的设计、项目的建立以及日志的使用，第四章开始讲系统中买家端的构建。  
买家端的开发包括三部分：DAO层的设计与开发，Service层的设计与开发，Controller层的设计与开发；这是一个MVC结构。

# 4.课程笔记

## 第四章.买家类目

在ProductCategory类中创建的变量需要生成get和set方法，这样修改起来麻烦；可以在pom.xml中添加依赖：

```xml
<dependency>  
    <groupId>org.projectlombok</groupId>  
    <artifactId>lombok</artifactId>  
</dependency>  
```
  
然后搜索插件lombok并安装;然后在ProductCategory类中添加注解@Data，包含了get、set和toString的一些方法，然后就可以删掉这些方法的代码了，由@Data会自动生成；  
在ProducCategoryRepositoryTest类中向数据库添加记录，如果只是测试的话，可以在对应方法前加上注解@Transactional，使得添加记录之后回滚，不会把记录保存在数据库;  
如果代码在编译的时候就出错，看看哪里把代码写错了，或是少了注解；

## 第五章.买家商品

根据一开始就制定好的文档，对于商品的展示有一个层级的结构，因此定义了一个方法查询所有在架的商品，并将其信息按照文档的要求进行展示，此时可以通过`localhost:8080/sell2/buyer/product/list`进行访问，也可以通过虚拟机中已经构建好的前端来进行展示；如果直接访问虚拟机地址`192.168.1.105/#/`会是一个空白界面（前提是设置了cookie），因为前端没有内容，这是就需要编辑虚拟机的文件：`vim
/usr/local/nginx/conf/nginx.conf`，将里面的IP改为自己电脑的IP，就可以通过`192.168.1.103/#/`访问，并且显示了商品，并且可以设定`server_name=sell.com`然后通过`sell.com/#/`访问（也要设置cookie）；

## 第七章.微信授权

微信支付包括授权和支付，授权使用的测试号，在[微信公众平台](https://developers.weixin.qq.com/doc/offiaccount/Basic_Information/Requesting_an_API_Test_Account.html)里可以申请，然后就可以使用AppId和AppSecret（之前好像不行，后面在手动获取openid的过程中拿到了Secret）。按照[腾讯网页授权](https://developers.weixin.qq.com/doc/offiaccount/OA_Web_Apps/Wechat_webpage_authorization.html)文档的提示，先在用户同意授权获取code，然后使用code换取网页授权access_token，从而获取用户的openid，这一过程也可以通过SDK完成，通过在手机微信上点击生成的url，使用户授权登录，从而获取openid，并将网页重定向到指定url（可以设定为自己的域名）；  
如果想有一个外网可以访问的域名，可以在`https://natapp.cn/`申请，先购买一个VIP_1隧道，然后再购买一个二级域名，配置好之后，下载客户端，然后按照提示修改config.ini文件，官网有详细的说明，最后就可以通过所购买的域名`http://mysell.mynatapp.cc`访问本地localhost了；
