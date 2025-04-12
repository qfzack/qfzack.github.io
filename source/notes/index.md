# Notebook

## WSL

- windows subsystem for linux
- 先在softcenter安装wsl
- 在MS store安装ubuntu，或者powershell执行`wsl --install -d ubuntu`
- 安装完启动即可

> 踩坑：第一次安装完vscode无法连接wsl，于是卸载，卸载之后就再也安装不上了
解决：申请了临时管理员权限，升级wsl lunux内核，wsl --update，然后就可以了
> 
- 配合vscode使用：
    - 安装插件remote-wsl
    - 在ubuntu中创建文件夹，然后可以打开跳转到vscode，`mkdir dir`，`code dir`
    - vscode也自动连接上了wsl，可以在wsl ubuntu中进行开发了
    - 本地安装的vscode在wsl中的路径是`/mnt/c/Users/Qingfeng_Zhang/AppData/Local/Programs/Microsoft VS Code`
    - 当出现command not found: code时：
        - 先找到code的位置：`sudo find /mnt/c/ -name "code" -type f 2>/dev/null`
        - 创建软连接：`sudo ln -s /mnt/c/Users/Qingfeng_Zhang/AppData/Local/Programs/Microsoft\ VS\ Code/bin/code /usr/local/bin/code`

## Golang

### installation

- apt-get只能下载13版本的go，有点太旧了
- 下载go：`wget https://go.dev/dl/go1.21.3.linux-amd64.tar.gz --no-check-certificate`
- 使用`tar -xzvf ./golang -zxvf go1.21.3.linux-amd64.tar.gz`解压文件，然后配置环境变量

```bash
export GO111MODULE=on
export GOPROXY=https://goproxy.cn,direct

export GOROOT=/home/qingfeng_zhang/golang/go
export PATH=$PATH:$GOROOT/bin

export GOPATH=/home/qingfeng_zhang/golang/gopath
export PATH=$PATH:$GOPATH/bin
```

- vscode安装go插件，然后ctrl+shift+p搜索go: install安装go tools

### version management

- install gvm refer to

https://github.com/moovweb/gvm

```bash
sudo apt-get install bison
bash < <(curl -s -S -L https://raw.githubusercontent.com/moovweb/gvm/master/binscripts/gvm-installer)
```

- install go1.20

```bash
gvm install go1.20
gvm list
gvm use
```

## Nodejs

- `wget https://registry.npmmirror.com/-/binary/node/v20.13.0/node-v20.13.0-linux-x64.tar.xz`注意版本
- `tar -xvf node-v20.13.0-linux-x64.tar.xz`
- 设置node和npm的软连接，注意路径是否正确:
- `sudo ln -s /home/qingfeng_zhang/app/node-v20.13.0-linux-x64/bin/node /usr/local/bin/`
- `sudo ln -s /home/qingfeng_zhang/app/node-v20.13.0-linux-x64/bin/npm /usr/local/bin/`
- 更换node版本：
    - 安装n模块：`npm install -g n`
    - 升级到最新的稳定版本：`n stable`
    - 也可以安装指定版本`n v14.17.5`，如果找不到n则进入到对应的安装目录执行

## Python

- version management with pyenv

```
curl -L <https://raw.githubusercontent.com/pyenv/pyenv-installer/master/bin/pyenv-installer> | bash

export PATH=$HOME/.pyenv/bin:$PATH
eval "$(pyenv init -)"
eval "$(pyenv virtualenv-init -)"

pyenv install 3.11
pyenv versions
pyenv global 3.11
```

## 5. Java

- https://www.digitalocean.com/community/tutorials/install-maven-linux-ubuntu

## 5.Docker

- docker可以安装在windows上，这里选择安装在wsl(ubuntu)上
- Refer to guideline  https://docs.docker.com/engine/install/ubuntu/#install-using-the-convenience-script

```bash
# install docker
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh

# start docker daemon
sudo service docker start

# fix non-root user can not connect to docker daemon
# https://www.digitalocean.com/community/questions/how-to-fix-docker-got-permission-denied-while-trying-to-connect-to-the-docker-daemon-socket
sudo groupadd docker
sudo usermod -aG docker ${USER}
su -s ${USER}

# configure ca cert
wget --no-check-certificate  https://amaas-eos-drm1.cec.lab.emc.com/artifactory/vxrail-binaries-virtual/certs/EMC_CA_ROOT.crt -O /usr/local/share/ca-certificates/EMC_CA_ROOT.crt

# restart docker daemon
sudo service docker restart
```

## Build image

- build with docker file Dockerfile.python3.11 in current dir

```
docker build -t amaas-eos-drm1.cec.lab.emc.com:5033/vxraildevops/ci/blackduck-drp-scanner:python3.11 -f Dockerfile.python3.11 .

```

- push to image hub

```
docker push amaas-eos-drm1.cec.lab.emc.com:5033/vxraildevops/ci/blackduck-drp-scanner:python3.11

```

### Redis Container

- docker hub查询可用的docker镜像`docker search redis`
- 报错，修改docker镜像地址：`sudo vim /etc/docker/daemon.json`

```
{
    "registry-mirrors": "<https://dftbcros.mirror.aliyuncs.com>"
}

```

- 重启docker服务`systemctl restart docker`，继续pull将镜像拉到本地
- 根据docker镜像创建docker容器：

```
docker run -p 6379:6379 -v /home/mystic/Containers/data:/data -d redis redis-server --appendonly yes

```

> -p：将主机(VDI)的端口映射到容器的端口
-v：将主机的指定目录挂载到容器的指定目录下
redis-server --appendonly yes：在容器中执行redis-server启动服务，并开启redis持久化
> 
- 进入docker容器中`docker exec -it 95 bash`

### Container In VDI

- 在VDI中`ifconfig -a`查看ip为**20.1.206.197**
- 但是VDI不是直接连接的，中间经过可以一个server的转发，在[lab portal](http://10.124.108.208/nvp/#/vdi)中可以查看相关的主机信息
- 可以看到如果是用ubuntu的ssh，中间经过的ip端口号就是**10.124.95.16:22587**
- 而VDI中docker容器中redis服务的6379端口已经映射到了VDI中的6379端口
- 因此，对于VDI容器中的redis服务，要先将VDI和中间的这个机器的6379端口连接起来
    
    ```
    ssh mystic@10.124.95.16 -P 22587 -L 6379:127.0.0.1:6379
    
    ```
    
- 然后，再在本地连接中间的这个server的6379端口`./redis-cli -h 10.124.95.16 -p 6379`

> 访问顺序是本地->中间机器->VDI->docker容器，是不是有点太麻烦...
> 

### VDI User/Password

- Ubuntu ：mystic/mystic
- Windows：user2/Password123!

## 6.Jenkins

- 以docker容器的方式启动jenkins，因为原来使用的jenkins版本是2.263.4，因此拉取相同版本的镜像
- 添加一个本地的挂载目录`/var/jenkins_mount`，给777权限，方便查看容器中的配置文件
- 创建docker容器：
    
    ```
    docker run -d -p 9000:8080 -p 50000:50000 -v /var/jenkins_mount:/var/jenkins_home -v /etc/localtime:/etc/localtime --name jenkins-2.263.4 jenkins/
    jenkins:2.263.4
    
    ```
    

## 7.Python

### Install Anaconda

- 下载Anaconda的安装文件Anaconda3-2021.05-Linux-x86_64.sh：
    
    ```
    curl -L -O <url>
    
    ```
    
- 安装Anaconda：
    
    ```
    chmod +x Anaconda3-2023.03-1-Linux-x86_64.sh
    
    ./Anaconda3-2023.03-1-Linux-x86_64.sh
    
    ```
    

### Create Python Env

- 使用conda创建python3.10的环境：

```
conda create --name python3.10 python=3.10

```

- 如果出现SSL证书的错误，先忽略SSL认证：
    
    ```
    conda config --set ssl_verify no
    ```
    

### Version Control

- 查看已安装的环境：
    
    ```
    conda info --envs
    
    ```
    
- 使用指定的环境：
    
    ```
    conda activate python3.10
    
    ```
    

## Groovy

- refer to https://groovy-lang.org/install.html to install
- install java

```
apt install default-jre
apt install default-jdk

```

- config JAVA_HOME

```
export JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64

```

## zsh

```
apt install zsh
chsh -s /bin/zsh

sh -c "$(curl -fsSL <https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh>)"
```

- suggest theme: `ys`

# Summary

# 1.Git

- 一般是先把原始的repo fork为自己的repo，然后clone自己的repo进行开发，开发完提交到远程repo，然后再pull request到原始repo上
    - upstream repo：原始repo
    - 远程repo：从原始repo fork过来的
    - 本地repo：从远程repo clone下来的

### Configuration

```
git config --global user.name <username>
git config --global user.email <email>

ssh-keygen -t rsa -C <email>

```

- configure multi RSA

```
ssh-keygen -t rsa -C <email> -f "~/.ssh/id_rsa_personal"

```

### Workflow

1. fork repo
2. clone repo
3. `git remote add upstream <https://eos2git.cec.lab.emc.com/vxrail/poxio.git`>
    - `git remote -v`查看远程仓库，没有连接upstream之前就只有origin
    - 连接upstream之后可以看到有upstream
    - 然后可以从upstream repo拉去最新的改动：`git fetch upstream`（习惯使用`git pull upstream master --rebase`）
4. `git checkout -b feature/poxio-ui/select-option`
    - 切换到本地分支进行开发
5. `git commit -am "comment"`
    - 将分支提交到本地仓库
    - 配置commit模板之后就可以`git commit -a`提交所有变动，然后进入commit信息编辑页面
6. `git push origin feature/poxio-ui/select-option`
    - 将分支推送到远程origin
7. `git log`
    - 查看commit历史
8. `git rebase -i HEAD~3`
    - 如果branch里有多条commit记录，就合并为一条
9. `git pull upstream master --rebase`
    - 从upstream拉去更新，且不产生新的commit记录
10. submit pull request
11. PR is reviewed and merged

### Common Operation

- **修改branch name**
    - `git branch -m oldName newName`
    - 如果已经推送到远程删除远程分支`git push --delete origin oldName`再推送
    - 推送新分支`git push origin newName`
    - 将本地分支与远程分支关联`git branch --set-upstream-to origin/newName`
- **配置commit模板**
    - https://eos2git.cec.lab.emc.com/vxrail/poxio/blob/master/.github/template_guide.md
    - `git config --global commit.template ../commit_template.txt`
- **rebase合并commit**
    - https://juejin.cn/post/6844903600976576519
    - `git log`查看提交历史
    - `git rebase -i HEAD~3` 编辑前三条commit
    - 进入到编辑界面：将`pick`修改为`squash`或`s`，表示和前一个commit进行合并
    - 保存退出之后会编辑commit信息，编辑完保存即可
- **分支创建与合并**
    - `git checkout -b new_branch`创建并切换到此分支
    - `git merge new_branch`将new_branch的变更合并到当前分支上
- **删除本地和远程分支**
- `git branch -d branch_name`
- `git push origin --delete branch_name`
- **修改commit信息**
    - `git commit --amend`
- **tag为repo加标签**
    - `git tag -l "v1.0.*"`
    - `git tag -a v1.0.0 -m "comment"`
    - `git tag -a v1.0.0 <commit_hash>`
    - `git tag v1.0.0 -n "comment"`
    - `git tag -d v1.0.0`
    - `git push origin v1.0.0`
- **local切换upstream的branch**
    - `git fetch upstream`
    - `git branch -a`
    - `git checkout -b <branch_name> upstream/<branch_name>`
    - `git branch -vv`
- **stash暂存修改**
    - `git stash save "stash-info"` 保存当前的修改
    - `git stash list` 查看stash列表
    - `git stash apply 0` 使用指定的stash
    - `git stash drop 0` 删除指定的stash
    - `git stash show 0 -p` 查看指定stash内容
    ![[Pasted image 20230516113453.png]]
- 使用HTTPS时禁用SSL验证：
    
    ```
    git clone -c http.sslVerify=false <Repo_URL>
    
    ```
    

## 2.Docker

### Postgres

- docker启动postgres：
    - 拉去postgres镜像：`docker pull bitnami/postgresql`
    - 启动docker容器：`docker run --name postgres -e POSTGRES_PASSWORD=postgrespw -p 5432:5432 -d bitnami/postgresql`
    - 进入容器：`docker exec -it <imageID> /bin/bash`
    - 进入postgres：`psql -U postgres`
- docker启动postgres客户端pgadmin4：
    - 拉去pgadmin镜像：`docker pull dpage/pgadmin4`
    - 启动docker容器：`docker run -p 5050:80 -e 'PGADMIN_DEFAULT_EMAIL=pgadmin4@pgadmin.org' -e 'PGADMIN_DEFAULT_PASSWORD=admin' -d --name pgadmin4 dpage/pgadmin4`
- 常用命令：
    - 查看当前数据库：`select current_database();`或者`\l`
    - 创建数据库：`CREATE DATABASE dbname;`
    - 进入数据库：`\c dbName`
    - 查看当前用户：`select user;`or `\du`
    - 查看所有数据表：`\d`
- wsl启动的postgres：
    - name: postgres
    - password: postgrespw
- data dump and restore:
    
    ```
    pg_dump -U postgres -t ingested_commits -f /devequ-snapshoot/ingested_commits.sql poxio_datalake
    psql -h 127.0.0.1 -p 5432 -U postgres -d poxio_datalake -f /devequ-snapshoot/ingested_commits.sql
    
    ```
    

### Mongodb

- docker启动mongodb：
    - 拉取mongo的镜像：`docker pull amaas-eos-drm1.cec.lab.emc.com:5033/vxraildevops/mongo:4.2.7`
    - 启动docker容器：`docker run --name mongodb -p 27017:27017 -d mongo`
    - 进入容器：`docker exec -it mongo /bin/bash`
    - 进入mongo：`mongo -u <userName> -p`
- 常用命令：
    - 查询数据库：`show dbs`
    - 查看当前数据库：`<dbName>`
    - 切换数据库：`use <dbName>`
    - 查看collections：`show collections`
- 使用k8s中的mongo：
    - `kubectl exec -i -t -n mongodb-cluster-prod deep-mongodb-cluster-prod-0 -c mongod -- sh -c "(bash || ash || sh)"`

```
// 查询文档
db.getCollection('commit').find({"repo": "nano-service","ntId":{$exists: false}})

// 查询数量
db.getCollection('commit').count({"repo": "nano-service","ntId":{$exists: false}})

// 更新文档的字段
db.getCollection('commit').update({"repo": "nano-service"},{ $set: { "workItemProject": "" } })
db.getCollection('commit').updateMany({"repo": "nano-service"},{ $set: { "workItemProject": "" } })

// 删除文档的字段
db.getCollection('commit').update({"repo": "nano-service"},{ $unset: { "ntId": "" } })
db.getCollection('commit').updateMany({"repo": "nano-service"},{ $unset: { "ntId": "" } })

//判断字段存在
"ntId":{$exists: false}}
//判断字段等于""
"workItemKey":{$eq: ""}}
//判断字段不等于""
"workItemKey":{$ne: ""}}

```

### Redis

- docker pull and run Redis image

```
docker pull redis
docker run --name redis -p 6379:6379 -d redis
```

- 使用redis cli连接redis服务
    
    ```
    redis-cli -h <ip-address> -p <port>
    
    ```
    

### Grafana

```
docker pull grafana/grafana
docker run --name grafana -p 3000:3000 -d grafana/grafana:latest
```

### Prometheus

- 启动node-exporter

```
docker pull prom/node-exporter
docker run --name node-exporter -p 9100:9100 -v "/proc:/host/proc:ro" -v "/sys:/host/sys:ro" -v "/:/rootfs:ro" -d prom/node-exporter
```

- 创建配置文件`/opt/prometheus/prometheus.yml`:

```yaml
global:
  scrape_interval:     60s
  evaluation_interval: 60s

scrape_configs:
  - job_name: prometheus
    static_configs:
      - targets: ['localhost:9090']
        labels:
          instance: prometheus

  - job_name: linux
    static_configs:
      - targets: ['127.0.0.1:9100']
        labels:
          instance: localhost

```

- 启动prometheus

```
  docker pull bitnami/prometheus

  docker run --name prometheus  -d \\
  -p 9090:9090 \\
  -v /opt/prometheus/prometheus.yml:/etc/prometheus/prometheus.yml  \\
  bitnami/prometheus
```

- config file path:
    - `\\\\wsl.localhost\\Ubuntu\\home\\qingfeng_zhang\\wsl_project\\vxrail-docker-library\\projects\\_\\vxraildevops\\prometheus\\deploy\\etc\\prometheus.yml`
    - /etc/prometheus/prometheus.yml

## 3.Kubernetes

### minikube

- 安装minikube - https://minikube.sigs.k8s.io/docs/start/

```
curl -LO <https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64>
sudo install minikube-linux-amd64 /usr/local/bin/minikube
```

### kubectl

```
kubectl get events  #查看集群事件
```

- 获取pods信息
    
    ```bash
    kubectl get pods
    kubectl get pods --all-namespaces
    #获取pod详细信息
    kubectl get pods -o wide
    #获取yaml格式的资源配置
    kubectl get po <podname> -o yaml
    ```
    
- 创建资源
    
    ```
    kubectl create deployment my-deployment --image=imageName  #创建deployment
    kubectl expose deployment my-deployment --type=LoadBalancer --port=8080  #创建service，将pod暴露给公网
    kubectl apply -f config.yaml  #根据yaml配置文件创建或更新资源
    
    ```
    
- 删除资源

```
kubectl delete pod pod-name  #删除pod
kubectl delete pod pod-name -n namespace

kubectl delete service serviceName  #删除service
kubectl delete deployment deploymentName  #删除deployment

kubectl delete pod <pod-name> -n merico-prod --force --grace-period=0  #强制删除
```

- describe
    
    ```
    kubectl describe nodes my-node  #查看node的详细信息
    kubectl describe pods my-node  #查看pod的详细信息
    ```
    
- 扩容

```
  kubectl scale deployment my-deployment --replicas=0
  kubectl scale deployment my-deployment --replicas=1 -n namespace
```

- pod日志
    
    ```
    kubectl logs podName
    ```
    
- 修改配置
    
    ```
    kubectl set image -n nameSpace deployment/my-deployment my-deployment=imageName  #修改deployment的image,可选image,resources,selector,subject
    ```
    
- 清理无效的pod
    
    ```
    kubectl delete pods --field-selector=status.phase=Failed
    ```
    

### helm

- chart代表helm包
- repository用来存放和共享chart
- release是运行在kubernetes中的chart实例

```
wget <https://get.helm.sh/helm-v3.12.0-linux-amd64.tar.gz>

tar -xvf  helm-v3.12.0-linux-amd64.tar.gz
sudo mv linux-amd64  /usr/local/bin
```

### K9s

- k8s管理工具，类似Lens，但是是命令行使用的
- install:

```
wget <https://github.com/derailed/k9s/releases/download/v0.27.4/k9s_Linux_amd64.tar.gz>
tar -zxf k9s_Linux_amd64.tar.gz -C /usr/local/bin
```

### OCP tool

```bash
mkdir ~/bin && cd ~/bin
curl -kO https://mirror.openshift.com/pub/openshift-v4/x86_64/clients/ocp/4.13.8/openshift-client-linux.tar.gz
tar xzvf openshift-client-linux.tar.gz
rm openshift-client-linux.tar.gz
# rm kubectl if you don't want the kubectl that comes with oc
```

## 4.Swagger

- swagger tools in golang is swaggo
- https://github.com/swaggo/swag/blob/master/README_zh-CN.md
- update swagger docs

```
swag init -g hooks.go -d ./internal/pkg/api-machinery/api/v3/hooks -o ./api/api-machinery/docs --parseDependency true
```

## 5.Sonarqube

### sonar scan

- install path: `/opt/sonar-scanner`
- test with local sonar cli
    
    ```
    sonar-scanner \\
    '-Dsonar.host.url=https://sonarqube1.cec.lab.emc.com/' \\
    '-Dsonar.projectKey=vxrail-mcp-fermion-vmw' \\
    '-Dsonar.projectName=VxRail :: mcp-fermion-vmw' \\
    '-Dsonar.sources=.' \\
    '-Dsonar.tests=.' \\
    '-Dsonar.test.inclusions=**/*_test.go' \\
    '-Dsonar.go.coverage.reportPaths=../coverage.out' \\
    '-Dsonar.go.tests.reportPaths=report.xml'
    ```
    

### Sonar API

- api/measures/component
    - https://sonarqube1.cec.lab.emc.com/api/measures/component?additionalFields=metrics&component=vxrail-kgs-svc&metricKeys=ncloc,coverage,complexity,code_smells,new_coverage,duplicated_lines_density&branch=rel/vxrail-berwick&from=2023-06-07
- api/project_branches/list
    - https://sonarqube1.cec.lab.emc.com/api/project_branches/list?project=vxrail-kgs-svc
- api/measures/search_history
    - https://sonarqube1.cec.lab.emc.com/api/measures/search_history?additionalFields=metrics&component=vxrail-kgs-svc&metrics=ncloc,coverage,complexity,code_smells,new_coverage,duplicated_lines_density&branch=rel/vxrail-berwick&from=2023-06-07
- api/issues/search
    - https://sonarqube1.cec.lab.emc.com/api/issues/search?componentKeys=vxrail-kgs-svc&resolved=false&types=BUG&facets=severities&createdBefore=2023-06-07&branch=master
- api/components/show
    - https://sonarqube1.cec.lab.emc.com/api/components/show?component=vxrail-dm-common
- api/components/search
    - [https://sonarqube1.cec.lab.emc.com/api/components/search?q=az-lcm :: lcm-upgrade-manager&qualifiers=TRK&ps=500](https://sonarqube1.cec.lab.emc.com/api/components/search?q=az-lcm%20::%20lcm-upgrade-manager&qualifiers=TRK&ps=500)

## 6.MinIO

- MinIO is an object storage solution that provide an Amazon web service S3-compatible API and supports all core S3 feature
- `mc` commandline tool is built to operate Minio
- install: https://min.io/docs/minio/linux/reference/minio-mc.html#install-mc

```
curl <https://dl.min.io/client/mc/release/linux-amd64/mc> \\
  --create-dirs \\
  -o $HOME/minio-binaries/mc

chmod +x $HOME/minio-binaries/mc
export PATH=$PATH:$HOME/minio-binaries/

```

- create alias for minio service

```
mc alias set swarm-minio <http://rduvxrbldprd001.isus.emc.com:8080> kkhQ40VmQGwq1zzf M8ciRJfBTZSHf5EApMagFo0zLbBNvQBU

```

- usage
    
    ```
    mc ls swarm-minio                     #list buckets in minio
    mc ls swarm-minio/deep-merico-backup  #list buckets objects in minio
    
    ```
    

```
mc rm --recursive <ALIAS>/<BUCKET>/<FOLDER> --force --versions #force delete all version folder and files recursively

```

# OCP Cluster

- Ingress: *.nginx.demo-partners.k8s.cec.delllabs.net

```
  oc login -u < login id   >   -p  <password >  api.demo-partners.k8s.cec.delllabs.net:6443

```

```
oc login --token=sha256~AlCjDxGwmfYTMS-1_jYXevKCKwEQNT0Zb-ld42RIKEw --server=https://api.primarystorage-stage-drm.k8s.cec.delllabs.net:6443
oc projects

```
