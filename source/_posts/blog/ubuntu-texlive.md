---
title: "通过jupyterlab使用远程服务器的Texlive"

date: 2020-10-11T05:00:00Z
# image: "/images/placeholder.png"
categories: ["Linux"]
author: "Qingfeng Zhang"
---

在jupyter lab上编写文档非常方便，并且jupyter lab上也有大量的插件，其中就包括Latex，可以将编写的tex文件编译成pdf文件  
**Latex文章的编译过程分为四步：**  
1.使用latex和xelatex/pdflatex编译main.tex；  
2.使用bibtex编译得到的main.aux文件；  
3.使用xelatex/pdflatex编译得到的main.tex文件；  
4.重复步骤3，完善tex文件；

**1\. Ubuntu环境安装TeXlive：**  
Latex有很多发行版，一般在linux上使用TeXlive，使用命令安装TeXlive：

```shell
apt-get install texlive-full  
```
  
安装xelatex编译引擎：

```shell
apt-get install texlive-xetex  
```

（我是使用`apt-get install texlive-xetex latex-cjk-all`直接安装，后面还有个中文包）

**2\. jupyter lab安装Latex插件**  
直接搜插件Latex安装，按照插件的github的提示进行操作，但是报错显示找不到引用的文献；  
查了一下资料，发现BibTeX是一种用于LaTeX参考文献处理的格式和程序，BibTex文件后缀名为.bib的文件，应该是要安装texlive-
full（插件的github界面没有直说）  
![](images/ubuntu-texlive/tex.jpg)

(一直报错，未完待续)
