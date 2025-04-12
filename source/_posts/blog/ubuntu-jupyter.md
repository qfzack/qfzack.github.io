---
title: "ubuntu环境下配置远程访问jupyter notebook"

date: 2020-10-11T05:00:00Z
# image: "/images/placeholder.png"
categories: ["Linux"]
author: "Qingfeng Zhang"
---

之前一直不知道怎么在服务器上配置远程远程访问的jupyter，于是就用学院的机器学习平台试了一下，在这里记录一下配置过程。

**1\. 在平台上创建的一个容器：**  
![](images/ubuntu-jupyter/Image2.jpg)  
第二列的10.1.13.239可能就是我的容器IP

**2\. 创建端口映射**  
![](images/ubuntu-jupyter/Image3.jpg)  
创建一个映射到8888端口的校内端口，分配到的是30175，这也是之后在浏览器上使用的端口

**3\. 连接服务器：**  
![](images/ubuntu-jupyter/Image4.jpg)  
使用命令`jupter notebook --generate-config`生成配置文件  
进入ipython：  
![](images/ubuntu-jupyter/Image5.jpg)  
ipython是python的交互式shell；  
在ipython中导入`notebook.auth`的`passwd`，通过`passwd()`命令设置密码  
注意保存输出的password字符；

**4\. 修改配置文件：**  
![](images/ubuntu-jupyter/Image6.jpg)  
运行`vim ~/.jupyter/jupyter_notebook_config.py`编辑配置文件，在文件前面加上：

```
c.NotebookApp.ip='0.0.0.0'  
c.NotebookApp.password = u'sha1:bdb932c5e9f9:b9123eb02cb230ad601e4fb98a85c2bbe70c18fb'  
c.NotebookApp.open_browser = False  
c.NotebookApp.port =8888  
c.IPKernelApp.pylab = 'inline'  
```
  
这个可以根据需要修改

**5\. 启动jupyter notebook：**  
![](images/ubuntu-jupyter/Image7.jpg)  
我是使用命令`jupyter notebook --ip=0.0.0.0 --allow-root`启动  
然后在浏览器中访问`10.0.4.235:30175`

  * 使用`jupyter notebook list`查看正在运行的server；
  * 使用`jupyter notebook stop 8888`停止8888端口的server

**6\. jupyterlab的配置使用：**  
之后又尝试使用notebook的升级版jupyterlab，结果发现pip安装之后，其配置与notebook一样，也就是不需要配置了，直接`jupyter
lab`启动，然后进浏览器打开就行；  
jupyter notebook与jupyter lab使用的是同一个端口，那么如果我想打开notebook怎么办？  
其实两者的区别只在于地址后面的`tree`和`lab`;

  * jupyter notebook的地址是`http://10.0.4.235:30175/tree`
  * jupyter lab的地址是`http://10.0.4.235:30175/lab`

**7\. 使jupyter后台运行：**  
shell里启动jupyter后，如果ctrl+c终止会使网页端也断掉；  
`nohup`是不挂断地运行命令，`&`表示在后台运行，使用`nohup jupyter lab &`可以保持jupyter一直运行  
![](images/ubuntu-jupyter/Image8.jpg)  
如果想关闭服务：

  * `ps -ef`查看后台进程： ![](images/ubuntu-jupyter/Image9.jpg)
  * 找到jupyterlab对应的进程号，使用`kill -9 12765`关掉进程

**8.安装jupyterlab的扩展**  
先安装一个生成目录的插件`jupyter labextension install @jupyterlab/toc`，提示要先安装nodejs和npm：  
![](images/ubuntu-jupyter/Image10.jpg)  
尝试运行：`jupyter serverextension enable --py jupyterlab --user`  
和`conda install -c conda-forge nodejs`之后，再次安装，并使用`jupyter labextension
list`查看已安装的插件：  
![](images/ubuntu-jupyter/Image11.jpg)

  * jupyterlab配置：<https://www.cnblogs.com/lskreno/p/10844315.html>
