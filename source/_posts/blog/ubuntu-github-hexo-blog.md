---
title: "ubuntu下使用github搭建hexo博客"

date: 2020-10-11T05:00:00Z
# image: "/images/placeholder.png"
categories: ["Linux"]
author: "Qingfeng Zhang"
---

  * 使用github搭建博客应该先安装git和node.js，git已经安装了，现在安装node.js，node.js是运行在服务端的JavaScript（不是很懂）

# 1.安装node.js：

**1.1 安装nvm**

  * nvm是一个node版本管理工具，就像Python中的anaconda，先安装nvm，参考[Install & Update Script](https://github.com/nvm-sh/nvm?tab=readme-ov-file#install--update-script)

```shell    
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.1/install.sh | bash
```

**1.2 安装node：**

  * 查看所有可用的node版本：

```shell
nvm ls-remote  
```
  
  * 指定一个安装版本：

```shell
nvm install v11.11.0  
```

  * 查看已安装的版本：

```shell
nvm ls  
```
  
# 2\. 安装hexo：

  * hexo是一个基于node.js制作的一个博客工具，hexo将markdown编写的文件生成静态html页面，然后上传到服务器；  
使用命令安装hexo：

```shell
npm install -g hexo  
```
  
(这里的npm是和nodejs一起安装的包管理工具，能解决nodejs代码部署的很多问题)

  * 初始化一个文件夹：

```shell
hexo init my_blog  
```
  
  * 进入文件夹，运行命令：

```shell
npm install  
```

  * 生成静态网页文件：

```shell
hexo generate  
```

等价于`hexo g`；

  * 启动服务器：

```shell
hexo server  
```
  
等价于`hexo s`，显示进入`http://localhost:4000`可以查看，但是是在服务器端搭建的，直接在浏览器无法查看；

# 3\. 部署到github：

> 需要安装git并设置name和email，然后生成公钥与github建立ssh连接，这些步骤在另外一篇文章里写了，并且环境也已经配置好了；

  * 检查ssh是否已经配置好：

```shell
ssh -T git@github.com  
```
  
然后在github上建立一个repository，注意名称为your_name.github.io;  
然后打开my_blog文件夹中的_config.yml进行编辑，修改最下面的deploy部分：  
![](images/ubuntu-github-hexo-blog/Image1.jpg)

  * 执行命令部署博客

```shell
hexo clean  
hexo generate  
hexo deploy  
```

然后运行以下命令就可以在浏览器的qfzack.github.io查看

但是最后一步报错：`ERROR Deployer not found: git`  
是因为没有安装hexo-deployer-git插件，安装命令：

```shell    
npm install hexo-deployer-git --save  
```

然后就可以成功打开网页了

# 4\. 发布文章到博客：

  * 使用markdown格式编辑好文件，放到`/my_blog/source/_posts`路径下，然后依次运行（generate和deploy可以用g和d代替）：

```shell
hexo clean  
hexo generate  
hexo deploy  
```

分别是清理缓存文件，生成静态文件，部署到网站，生成新文件也可以使用：

```shell
hexo n filename  
```
  
# 5.安装hexo主题：

  * 如果觉得原始的界面不好看，可以更换主题，下载的主题是放在`/my_blog/themes`文件夹下，在hexo的官网会提供很多主题，并且在主题的github界面会提供下载命令，以一个主题为例：

```shell    
git clone https://github.com/klugjo/hexo-theme-clean-blog.git themes/clean-blog  
```

  * 将clean-blog下载到themes文件夹内，然后修改_config.yml文件：

```shell
# Extensions  
## Plugins: http://hexo.io/plugins/  
## Themes: http://hexo.io/themes/  
theme: clean-blog  
```

主题的github界面会有更详细的参数设置，注意是修改主题文件夹内的_config.yml文件

# 6.使用subline Text3编辑文件

  * 使用subline Text3安装sftp插件，可以直接连上服务器，然后直接编辑服务器的文件，并上传，但是遇到一个问题：保存并上传文件的时候显示没有读写权限；  
对于这个问题，可以修改对应文件夹的权限来解决：

```shell
chmod 777 -R my_blog  
```
  
对于无法插入图片的问题，运行命令：

```shell
npm install https://github.com/CodeFalling/hexo-asset-image --save  
```

然后在使用hexo新建文件时会生成同名文件夹，将图片放在文件夹中并引用;  
但是最后还是有问题：对于新建的文件，sublime还是没有写入到权限；后来问了师兄才知道，不能只在容器里面修改文件夹的权限，需要在容器外面修改，这样新建文件的权限都是修改后的。

# 7.使用VScode远程连接服务器

  * 刚刚尝试使用vscode来做一些工作，和sublime类似，都支持很多插件，可以远程浏览并编辑服务器文件，之前是用pycharm做这些；可以直接上传和下载文件，也就是可以当作flashfxtp，虽然没有那么方便；并且可以直接使用终端，可以做之前sublime+xshell完成的任务；
  * vscode在本地、服务器、容器内安装的插件都是独立的，在连接远程服务器时遇到了一些困惑：**ssh连接的是宿主机，而不是我的dockers**

一开始的设置本地C:\Users\zhangqf.ssh的config文件如下：

```
    Host 1008server  
      HostName 192.168.195.55  
      User ubuntu  
```
  
这样登录是以普通的ubuntu用户登录，而我需要指定容器的端口进行登录，因此修改如下：

```    
    Host 1008server  
      HostName 192.168.195.55  
      User root  
      Port 9022  
```
  
这样就可以直接连到我自己的容器里面了，注意ubuntu用户和root用户的密码不同；

  * 还有一个问题就是每次都要输入root用户的密码，感觉有点麻烦，解决方案就是先在本机目录`C:\Users\zhangqf\.ssh`下运行：

```shell
ssh-keygen -t rsa  
```

生成密钥对，包括私钥id_rsa和公钥id_rsa.pub，然后再将公钥的内容复制到服务器的容器文件`~/.ssh/authorized_keys`中，然后就可以根据本机`.ssh/config`文件连接服务器且不需要再输入密码；
