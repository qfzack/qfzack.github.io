> **A notebook for some basic enviroment configuration in WSL2 (Ubuntu).**


# **WSL**

**Windows Subsystem for Linux (WSL)** is a feature of Windows that allows you to run a Linux environment on your Windows machine, without the need for a separate virtual machine or dual booting.

- search and install WSL(windows subsystem for linux) in windows store
- search and install the Ubuntu version you needed，or you can execute `wsl --install -d ubuntu` in Powershell
- and then the installation of Ubuntu subsystem is completed

> Node: upgrade wsl lunux kernel with `wsl --update`

- use with [VS Code](https://code.visualstudio.com/):
  - install plugin `remote-wsl` in VS Code
  - execute `code <path>` to open subsystem directory with VS Code
  - the path of `code` tool on the Windows machine is `/mnt/c/Users/Qingfeng_Zhang/AppData/Local/Programs/Microsoft VS Code`
  - if run `code` command to get error `command not found: code`:
    - find the position of `code` tool: `sudo find /mnt/c/ -name "code" -type f 2>/dev/null`
    - and then create symbolic links with: `sudo ln -s /mnt/c/Users/Qingfeng_Zhang/AppData/Local/Programs/Microsoft\ VS\ Code/bin/code /usr/local/bin/code`

# **Golang**

## basic enviroment installation

- find a appropriate golang version in [https://go.dev/dl/](https://go.dev/dl/)
- download golang package with wget: `wget https://go.dev/dl/go1.21.3.linux-amd64.tar.gz --no-check-certificate`
- extract golang files with `tar -xzvf ./golang -zxvf go1.21.3.linux-amd64.tar.gz`
- configure the golang enviroment variables:

```bash
export GO111MODULE=on
export GOPROXY=https://goproxy.cn,direct

export GOROOT=/home/qingfeng_zhang/golang/go
export PATH=$PATH:$GOROOT/bin

export GOPATH=/home/qingfeng_zhang/golang/gopath
export PATH=$PATH:$GOPATH/bin
```

- install some go development tools in VS Code:
  - use shortcut key `ctrl+shift+p`to open the commands palette, and type in `> go: install/Update Tools`to search and install go tools for development

## golang version management

[GVM](https://github.com/moovweb/gvm) is a usefule tool to manage Go versions

- install gvm refer to [https://github.com/moovweb/gvm](https://github.com/moovweb/gvm)

```bash
sudo apt-get install bison
bash < <(curl -s -S -L https://raw.githubusercontent.com/moovweb/gvm/master/binscripts/gvm-installer)
```

- install go1.23

```bash
# install new golang version
gvm install go1.23
# list all available golang vesions
gvm list
# use specific golang version
gvm use go1.23
```

# **Nodejs**

Node.js is a JavaScript runtime environment that allows you to run JavaScript code outside of a web browser.
NPM (Node Package Manager) is the default package manager for Node.js

- install node with version v20.13.0:

```bash
# download node package with specified version 
wget https://registry.npmmirror.com/-/binary/node/v20.13.0/node-v20.13.0-linux-x64.tar.xz
# extract the node files, contain the npm tool
tar -xvf node-v20.13.0-linux-x64.tar.xz
```

- set the symbolic link of `node` and `npm`:
  - `sudo ln -s /home/qingfeng_zhang/app/node-v20.13.0-linux-x64/bin/node /usr/local/bin/`
  - `sudo ln -s /home/qingfeng_zhang/app/node-v20.13.0-linux-x64/bin/npm /usr/local/bin/`

- `n` is a `node` version manager:
  - install with npm: `npm install -g n`
  - upgrade to the latest stable node version: `n stable`
  - or install the specified node version: `n v14.17.5`

# **Python**

## version management with [pyenv](https://github.com/pyenv/pyenv)

```shell
curl -L <https://raw.githubusercontent.com/pyenv/pyenv-installer/master/bin/pyenv-installer> | bash

export PATH=$HOME/.pyenv/bin:$PATH
eval "$(pyenv init -)"
eval "$(pyenv virtualenv-init -)"

pyenv install 3.11
pyenv versions
pyenv global 3.11
```

## version management with Anaconda

- download install file `Anaconda3-2021.05-Linux-x86_64.sh`:

```shell
curl -L -O <url>
```

- install Anaconda:

```shell
chmod +x Anaconda3-2023.03-1-Linux-x86_64.sh

./Anaconda3-2023.03-1-Linux-x86_64.sh
```

- create Python enviroment:

```shell
conda create --name python3.10 python=3.10
```

- if get error about SSL certificate，ignore with:

```shell
conda config --set ssl_verify no
```

- see all installed enviroments:

```shell
conda info --envs
```

- use specified enviroment:

```shell
conda activate python3.10
```

# **Java**

- install java in Ubuntu enviroment: [https://www.digitalocean.com/community/tutorials/install-maven-linux-ubuntu](https://www.digitalocean.com/community/tutorials/install-maven-linux-ubuntu)

# **Docker**

## Installation

- refer to guideline [install-using-the-convenience-script](https://docs.docker.com/engine/install/ubuntu/#install-using-the-convenience-script) to install Docker in WSL subsystem

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

# restart docker daemon
sudo service docker restart
```

## Build Image

- build with docker file in current directory

```shell
docker build -t <image_name>:<image-tag> -f <dockerfile> .
```

- push to docker image registry

```shell
docker push <image-registry-url>/<image-name>:<image-tag>
```

## Redis with Container

- search available docker images in docker hub with `docker search redis`

> check docker configuration file `cat ~/.docker/config.json`

- create redis container with docker image:

```shell
#  --appendonly yes: enable AOF data persistence in container
docker run -p 6379:6379 -v <local-path>:<container-path> -d redis redis-server --appendonly yes
```

- enter to redis container: `docker exec -it <container-id> bash`

# **Jenkins**

- 以docker容器的方式启动jenkins，因为原来使用的jenkins版本是2.263.4，因此拉取相同版本的镜像
- 添加一个本地的挂载目录`/var/jenkins_mount`，给777权限，方便查看容器中的配置文件
- 创建docker容器：

```shell
docker run -d -p 9000:8080 -p 50000:50000 -v /var/jenkins_mount:/var/jenkins_home -v /etc/localtime:/etc/localtime --name jenkins-2.263.4 jenkins/jenkins:2.263.4
```

# **Groovy**

- refer to https://groovy-lang.org/install.html to install
- install java

```shell
apt install default-jre
apt install default-jdk
```

- config JAVA_HOME

```shell
export JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64

```

# **zsh**

```shell
apt install zsh
chsh -s /bin/zsh

sh -c "$(curl -fsSL <https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh>)"
```

- suggest theme: `ys`

# **Git**

- 一般是先把原始的repo fork为自己的repo，然后clone自己的repo进行开发，开发完提交到远程repo，然后再pull request到原始repo上
  - upstream repo：原始repo
  - 远程repo：从原始repo fork过来的
  - 本地repo：从远程repo clone下来的

## Configuration

```shell
git config --global user.name <username>
git config --global user.email <email>

ssh-keygen -t rsa -C <email>
```

- configure multi RSA

```shell
ssh-keygen -t rsa -C <email> -f "~/.ssh/id_rsa_personal"
```

## Git Workflow

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

## Common Operation

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

- 使用HTTPS时禁用SSL验证：

```shell
git clone -c http.sslVerify=false <Repo_URL>
```

# **Docker Container**

## Postgres

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

```shell
pg_dump -U postgres -t ingested_commits -f /devequ-snapshoot/ingested_commits.sql poxio_datalake
psql -h 127.0.0.1 -p 5432 -U postgres -d poxio_datalake -f /devequ-snapshoot/ingested_commits.sql
```

## Mongodb

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

```mongo
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
"ntId":{$exists: false}
//判断字段等于""
"workItemKey":{$eq: ""}
//判断字段不等于""
"workItemKey":{$ne: ""}
```

## Redis

- docker pull and run Redis image

```shell
docker pull redis
docker run --name redis -p 6379:6379 -d redis
```

- 使用redis cli连接redis服务

```shell
redis-cli -h <ip-address> -p <port>    
```

## Grafana

```shell
docker pull grafana/grafana
docker run --name grafana -p 3000:3000 -d grafana/grafana:latest
```

## Prometheus

- 启动node-exporter

```shell
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

```shell
docker pull bitnami/prometheus

docker run --name prometheus  -d \\
    -p 9090:9090 \\
    -v /opt/prometheus/prometheus.yml:/etc/prometheus/prometheus.yml  \\
    bitnami/prometheus
```

- config file path:
  - `\\\\wsl.localhost\\Ubuntu\\home\\qingfeng_zhang\\wsl_project\\vxrail-docker-library\\projects\\_\\vxraildevops\\prometheus\\deploy\\etc\\prometheus.yml`
  - /etc/prometheus/prometheus.yml

# **Kubernetes**

## minikube

- 安装minikube - https://minikube.sigs.k8s.io/docs/start/

```shell
curl -LO <https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64>
sudo install minikube-linux-amd64 /usr/local/bin/minikube
```

## kubectl

```shell
kubectl get events  #查看集群事件
```

- 获取pods信息

```shell
kubectl get pods
kubectl get pods --all-namespaces
#获取pod详细信息
kubectl get pods -o wide
#获取yaml格式的资源配置
kubectl get po <podname> -o yaml
```

- 创建资源

```shell
kubectl create deployment my-deployment --image=imageName  #创建deployment
kubectl expose deployment my-deployment --type=LoadBalancer --port=8080  #创建service，将pod暴露给公网
kubectl apply -f config.yaml  #根据yaml配置文件创建或更新资源
```

- 删除资源

```shell
kubectl delete pod pod-name  #删除pod
kubectl delete pod pod-name -n namespace

kubectl delete service serviceName  #删除service
kubectl delete deployment deploymentName  #删除deployment

kubectl delete pod <pod-name> -n merico-prod --force --grace-period=0  #强制删除
```

- describe

```shell
kubectl describe nodes my-node  #查看node的详细信息
kubectl describe pods my-node  #查看pod的详细信息
```

- 扩容

```shell
kubectl scale deployment my-deployment --replicas=0
kubectl scale deployment my-deployment --replicas=1 -n namespace
```

- pod日志

```shell
kubectl logs podName
```

- 修改配置

```shell
kubectl set image -n nameSpace deployment/my-deployment my-deployment=imageName  #修改deployment的image,可选image,resources,selector,subject
```

- 清理无效的pod

```shell
kubectl delete pods --field-selector=status.phase=Failed
```

## helm

- chart代表helm包
- repository用来存放和共享chart
- release是运行在kubernetes中的chart实例

```shell
wget <https://get.helm.sh/helm-v3.12.0-linux-amd64.tar.gz>

tar -xvf  helm-v3.12.0-linux-amd64.tar.gz
sudo mv linux-amd64  /usr/local/bin
```

## K9s

- k8s管理工具，类似Lens，但是是命令行使用的
- install:

```shell
wget <https://github.com/derailed/k9s/releases/download/v0.27.4/k9s_Linux_amd64.tar.gz>
tar -zxf k9s_Linux_amd64.tar.gz -C /usr/local/bin
```

## OCP tool

```shell
mkdir ~/bin && cd ~/bin
curl -kO https://mirror.openshift.com/pub/openshift-v4/x86_64/clients/ocp/4.13.8/openshift-client-linux.tar.gz
tar xzvf openshift-client-linux.tar.gz
rm openshift-client-linux.tar.gz
# rm kubectl if you don't want the kubectl that comes with oc
```

# **Swagger**

- swagger tools in golang is swaggo
- https://github.com/swaggo/swag/blob/master/README_zh-CN.md
- update swagger docs

```shell
swag init -g hooks.go -d ./internal/pkg/api-machinery/api/v3/hooks -o ./api/api-machinery/docs --parseDependency true
```

# **Sonarqube**

## sonar scan

- install path: `/opt/sonar-scanner`
- test with local sonar cli

```shell
sonar-scanner \\
    '-Dsonar.host.url={sonar_server}' \\
    '-Dsonar.projectKey={project_key}' \\
    '-Dsonar.projectName={project_name}' \\
    '-Dsonar.sources=.' \\
    '-Dsonar.tests=.' \\
    '-Dsonar.test.inclusions=**/*_test.go' \\
    '-Dsonar.go.coverage.reportPaths=../coverage.out' \\
    '-Dsonar.go.tests.reportPaths=report.xml'
```

# **MinIO**

- MinIO is an object storage solution that provide an Amazon web service S3-compatible API and supports all core S3 feature
- `mc` commandline tool is built to operate Minio
- install: https://min.io/docs/minio/linux/reference/minio-mc.html#install-mc

```shell
curl <https://dl.min.io/client/mc/release/linux-amd64/mc> \\
  --create-dirs \\
  -o $HOME/minio-binaries/mc

chmod +x $HOME/minio-binaries/mc
export PATH=$PATH:$HOME/minio-binaries/
```

- create alias for minio service

```shell
mc alias set swarm-minio <http://rduvxrbldprd001.isus.emc.com:8080> kkhQ40VmQGwq1zzf M8ciRJfBTZSHf5EApMagFo0zLbBNvQBU

```

- usage

```shell
mc ls swarm-minio                     #list buckets in minio
mc ls swarm-minio/deep-merico-backup  #list buckets objects in minio
```

```shell
mc rm --recursive <ALIAS>/<BUCKET>/<FOLDER> --force --versions #force delete all version folder and files recursively
```

# **OCP Cluster**

- Ingress: *.nginx.demo-partners.k8s.cec.delllabs.net

```shell
oc login -u <login_id> -p <password> api.demo-partners.k8s.cec.delllabs.net:6443
```

```shell
oc login --token=sha256~AlCjDxGwmfYTMS-1_jYXevKCKwEQNT0Zb-ld42RIKEw --server=https://api.primarystorage-stage-drm.k8s.cec.delllabs.net:6443
oc projects
```
