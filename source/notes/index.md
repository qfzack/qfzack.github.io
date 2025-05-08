> **A notebook for some basic environment configuration in WSL2 (Ubuntu).**

# **WSL**

**Windows Subsystem for Linux (WSL)** is a feature of Windows that allows you to run a Linux environment on your Windows machine, without the need for a separate virtual machine or dual booting.

- search and install WSL(windows subsystem for linux) in windows store
- search and install the Ubuntu version you needed，or list all available version with `wsl --list --online`
- execute `wsl --install -d <distribution-version>` to install specified distribution in Powershell
- and then the installation of Ubuntu subsystem is completed

> Node: upgrade wsl lunux kernel with `wsl --update`

- use with [VS Code](https://code.visualstudio.com/):
  - install plugin `remote-wsl` in VS Code
  - execute `code <path>` to open subsystem directory with VS Code
  - the path of `code` tool on the Windows machine is `/mnt/c/Users/Qingfeng_Zhang/AppData/Local/Programs/Microsoft VS Code`
  - if run `code` command to get error `command not found: code`:
    - find the position of `code` tool: `sudo find /mnt/c/ -name "code" -type f 2>/dev/null`
    - and then create symbolic links with: `sudo ln -s /mnt/c/Users/Qingfeng_Zhang/AppData/Local/Programs/Microsoft\ VS\ Code/bin/code /usr/local/bin/code`

# **Zsh**

- Zsh, or Z Shell, is an extended Unix shell that serves as both an interactive command interpreter and a powerful scripting language:

```shell
apt install zsh
chsh -s /bin/zsh
```

- `Oh My Zsh` is a popular framework for managing Zsh configuration:

```shell
sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
```

- modify `Oh My Zsh` configuration file `vi ~/.zshrc`, and suggest theme is `ZSH_THEME="ys"`

# **Golang**

## basic environment installation

- find a appropriate golang version in [https://go.dev/dl/](https://go.dev/dl/)
- download golang package with wget: `wget https://go.dev/dl/go1.21.3.linux-amd64.tar.gz --no-check-certificate`
- extract golang files with `tar -xzvf ./golang -zxvf go1.21.3.linux-amd64.tar.gz`
- configure the golang environment variables:

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

[GVM](https://github.com/moovweb/gvm) is a useful tool to manage Go versions

- install gvm refer to [https://github.com/moovweb/gvm](https://github.com/moovweb/gvm)

```bash
sudo apt-get install bison
bash < <(curl -s -S -L https://raw.githubusercontent.com/moovweb/gvm/master/binscripts/gvm-installer)
```

- install go1.23

```bash
# install new golang version
gvm install go1.23
# list all available golang versions
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

- create Python environment:

```shell
conda create --name python3.10 python=3.10
```

- if get error about SSL certificate，ignore with:

```shell
conda config --set ssl_verify no
```

- see all installed environments:

```shell
conda info --envs
```

- use specified environment:

```shell
conda activate python3.10
```

# **Java**

- install java in Ubuntu environment: [https://www.digitalocean.com/community/tutorials/install-maven-linux-ubuntu](https://www.digitalocean.com/community/tutorials/install-maven-linux-ubuntu)

# **Groovy**

- refer to [https://groovy-lang.org/install.html](https://groovy-lang.org/install.html) to install Groovy
- install java:

```shell
apt install default-jre
apt install default-jdk
```

- configure JAVA_HOME variable

```shell
export JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64
```

# **Git**

## Configuration

- configure git info

```shell
# param --global is for global git configuration
git config --global user.name <username>
git config --global user.email <email>
git config --global commit.template ~/.git-commit-template.txt
git config --global core.editor "vim"

# generate rsa key
ssh-keygen -t rsa -C <email>
```

- configure multiple RSA

```shell
ssh-keygen -t rsa -C <email> -f "~/.ssh/id_rsa_personal"
```

## Git Workflow

1. fork repo
2. clone repo
3. `git remote add upstream git@github.com:<user>/<repo-name>`
   - use `git remote -v` to see all remote configs
4. create a new branch in local: `git checkout -b feature/dev`
5. use `git commit -am "comment"` to submit code change
   - if commit template is configured, run `git commit -a` will into the commit message edit page
6. push local branch to origin(fork) repo: `git push origin feature/dev`
7. check commit history with `git log`
8. if there have 2 commits and want to merge to 1, use `git rebase -i HEAD~2`
9. if the local code is behind the upstream repo, use `git pull upstream master --rebase` to sync with upstream master branch
   - the param `--rebase` will not create new commit
   - or use `git fetch upstream master` and `git merge upstream/master`
10. create pull request in Github page
11. wait for the PR to be reviewed and merge

## Common Operations

- **update branch name**
  - `git branch -m old_branch new_branch`
  - if the old-branch is existed in origin repo, execute `git push --delete origin old_branch` to delete it first
  - push the new branch`git push origin new_branch`
  - connect upstream branch with local branch: `git branch --set-upstream-to=origin/origin-branch local-branch`

- **configure commit template**
  - `git config --global commit.template ./commit_template.txt`

- **rebase commits**
  - use `git log` to see commit history
  - use `git rebase -i HEAD~3` to edit the first 3 commits
  - update `pick` to `squash` or `s`, which means to merge with previous commit
  - save and to edit the rebase commit message

- **branch create and merge**
  - `git checkout -b new_branch` create and switch to this branch
  - `git merge new_branch` merge changes from new_branch to current branch

- **delete local and remote branches**
- `git branch -d branch_name`
- `git push origin --delete branch_name`
- **modify commit information**
  - `git commit --amend`

- **add tags to repo**
  - `git tag -l "v1.0.*"`
  - `git tag -a v1.0.0 -m "comment"`
  - `git tag -a v1.0.0 <commit_hash>`
  - `git tag v1.0.0 -n "comment"`
  - `git tag -d v1.0.0`
  - `git push origin v1.0.0`

- **local switch to upstream branch**
  - `git fetch upstream`
  - `git branch -a`
  - `git checkout -b <branch_name> upstream/<branch_name>`
  - `git branch -vv`

- **stash changes**
  - `git stash save "stash-info"` save current changes
  - `git stash list` view stash list
  - `git stash apply 0` use specified stash
  - `git stash drop 0` delete specified stash
  - `git stash show 0 -p` view specified stash content

- Disable SSL verification when using HTTPS:

```shell
git clone -c http.sslVerify=false <Repo_URL>
```

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

## Postgres

- start postgres with docker:
  - pull postgres image: `docker pull bitnami/postgresql`
  - start docker container: `docker run --name postgres -e POSTGRES_PASSWORD=postgrespw -p 5432:5432 -d bitnami/postgresql`
  - enter container: `docker exec -it <imageID> /bin/bash`
  - enter postgres: `psql -U postgres`
- start postgres client pgadmin4 with docker:
  - pull pgadmin image: `docker pull dpage/pgadmin4`
  - start docker container: `docker run -p 5050:80 -e 'PGADMIN_DEFAULT_EMAIL=pgadmin4@pgadmin.org' -e 'PGADMIN_DEFAULT_PASSWORD=admin' -d --name pgadmin4 dpage/pgadmin4`
- common commands:
  - view current database: `select current_database();` or `\l`
  - create database: `CREATE DATABASE dbname;`
  - enter database: `\c dbName`
  - view current user: `select user;` or `\du`
  - view all tables: `\d`
- postgres started in wsl:
  - name: postgres
  - password: postgrespw
- data dump and restore:

```shell
pg_dump -U postgres -t ingested_commits -f /devequ-snapshoot/ingested_commits.sql poxio_datalake
psql -h 127.0.0.1 -p 5432 -U postgres -d poxio_datalake -f /devequ-snapshoot/ingested_commits.sql
```

## Mongodb

- start mongodb with docker:
  - pull mongo image: `docker pull amaas-eos-drm1.cec.lab.emc.com:5033/vxraildevops/mongo:4.2.7`
  - start docker container: `docker run --name mongodb -p 27017:27017 -d mongo`
  - enter container: `docker exec -it mongo /bin/bash`
  - enter mongo: `mongo -u <userName> -p`
- common commands:
  - query databases: `show dbs`
  - view current database: `<dbName>`
  - switch database: `use <dbName>`
  - view collections: `show collections`
- use mongo in k8s:
  - `kubectl exec -i -t -n mongodb-cluster-prod deep-mongodb-cluster-prod-0 -c mongod -- sh -c "(bash || ash || sh)"`

```mongo
// Query document
db.getCollection('commit').find({"repo": "nano-service","ntId":{$exists: false}})

// Count documents
db.getCollection('commit').count({"repo": "nano-service","ntId":{$exists: false}})

// Update document field
db.getCollection('commit').update({"repo": "nano-service"},{ $set: { "workItemProject": "" } })
db.getCollection('commit').updateMany({"repo": "nano-service"},{ $set: { "workItemProject": "" } })

// Delete document field
db.getCollection('commit').update({"repo": "nano-service"},{ $unset: { "ntId": "" } })
db.getCollection('commit').updateMany({"repo": "nano-service"},{ $unset: { "ntId": "" } })

// Check if field exists
"ntId":{$exists: false}
// Check if field equals ""
"workItemKey":{$eq: ""}
// Check if field not equals ""
"workItemKey":{$ne: ""}
```

## Redis

- Search available docker images in docker hub with `docker search redis`

> check docker configuration file `cat ~/.docker/config.json`

- create redis container with docker image:

```shell
# redis-server --appendonly yes: enable AOF data persistence in container
docker run -d -p 6379:6379 \
  -v <local-path>:<container-path> \
  redis \
  redis-server --appendonly yes
```

- enter to redis container: `docker exec -it <container-id> bash`

- connect to redis service using redis cli

```shell
redis-cli -h <ip-address> -p <port>    
```

## Jenkins

- create docker container with Jenkins image `jenkins/jenkins:2.263.4`:

```shell
docker run -d \
  -p 9000:8080 \
  -p 50000:50000 \
  -v <local-path>:/var/jenkins_home \
  -v /etc/localtime:/etc/localtime \
  --name jenkins-2.263.4 \
  jenkins/jenkins:2.263.4
```

## Grafana

```shell
docker pull grafana/grafana
docker run --name grafana -p 3000:3000 -d grafana/grafana:latest
```

## Prometheus

- start node-exporter

```shell
docker pull prom/node-exporter
docker run --name node-exporter -p 9100:9100 -v "/proc:/host/proc:ro" -v "/sys:/host/sys:ro" -v "/:/rootfs:ro" -d prom/node-exporter
```

- create configuration file `/opt/prometheus/prometheus.yml`:

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

- start prometheus

```shell
docker pull bitnami/prometheus

docker run --name prometheus  -d \\
    -p 9090:9090 \\
    -v /opt/prometheus/prometheus.yml:/etc/prometheus/prometheus.yml  \\
    bitnami/prometheus
```

- config file path: `/etc/prometheus/prometheus.yml`

# **Kubernetes**

## minikube

- install minikube - https://minikube.sigs.k8s.io/docs/start/

```shell
curl -LO <https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64>
sudo install minikube-linux-amd64 /usr/local/bin/minikube
```

## kubectl

```shell
kubectl get events  #view cluster events
```

- get pods information

```shell
kubectl get pods
kubectl get pods --all-namespaces
#get detailed pod information
kubectl get pods -o wide
#get resource configuration in yaml format
kubectl get po <podname> -o yaml
```

- create resources

```shell
kubectl create deployment my-deployment --image=imageName  #create deployment
kubectl expose deployment my-deployment --type=LoadBalancer --port=8080  #create service, expose pod to public network
kubectl apply -f config.yaml  #create or update resources based on yaml configuration file
```

- delete resources

```shell
kubectl delete pod pod-name  #delete pod
kubectl delete pod pod-name -n namespace

kubectl delete service serviceName  #delete service
kubectl delete deployment deploymentName  #delete deployment

kubectl delete pod <pod-name> -n merico-prod --force --grace-period=0  #force delete
```

- describe

```shell
kubectl describe nodes my-node  #view detailed node information
kubectl describe pods my-node  #view detailed pod information
```

- scaling

```shell
kubectl scale deployment my-deployment --replicas=0
kubectl scale deployment my-deployment --replicas=1 -n namespace
```

- pod logs

```shell
kubectl logs podName
```

- modify configuration

```shell
kubectl set image -n nameSpace deployment/my-deployment my-deployment=imageName  #modify deployment image, options include image, resources, selector, subject
```

- clean up invalid pods

```shell
kubectl delete pods --field-selector=status.phase=Failed
```

## helm

- chart represents a helm package
- repository is used to store and share charts
- release is a chart instance running in kubernetes

```shell
wget <https://get.helm.sh/helm-v3.12.0-linux-amd64.tar.gz>

tar -xvf  helm-v3.12.0-linux-amd64.tar.gz
sudo mv linux-amd64  /usr/local/bin
```

## K9s

- k8s management tool, similar to Lens, but used in command line
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

- swagger tools in golang is [swaggo](https://github.com/swaggo/swag/blob/master/README_zh-CN.md)
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
