title: "Fabric 单节点部署与链码测试"
date: 2016-11-26 23:25:00
tag:
- Fabric
- Hyperledger
- Blockchain
permalink: Fabric-Deployment-and-Chaincode-Setup
---

## 0x00 说明

作为区块链平台 fabric 项目的代码贡献者，IBM 在自己的 Bluemix 上提供了免费的区块链服务。在该服务中，我们可以一键创建 4 个认证节点的区块链网络，使用 REST API 方便地测试自己的 fabric 链码而不用在本地部署 fabric 区块链网络。

**为什么要本地部署 fabric？**

1. 在 Bluemix 上部署链码，需要把链码提交到 Github 上。在编写链码过程中，需要多次测试，每次都要上传到 Github。而本地部署链码只需要把链码在本地 Docker 中的认证节点上注册即可，不需多次上传 Github；
2. 可以方便地查看链码的打印日志；
2. 最近编写和测试链码的过程中，发现 Bluemix 提供的 fabric 服务中对链码方法参数的长度判断与本地部署 fabric 有所差异。目前在做的项目区块链网络最终要部署在本地。

在上一篇文章[《Ubuntu 中使用 Docker 部署 Hyperledger fabric》][1]中，总结了本地基于 Docker 的一键式部署方案，Docker 使用的镜像是由 [Baohua Yang][2] 构建。本文中总结使用 fabric 官方的 Docker 镜像在本地部署区块链平台，并在认证节点中注册链码（以 fabric [示例链码 02][3] 为例），使用 REST API 与链码交互。

## 0x01 准备工作

### 安装 docker

> 参照 https://docs.docker.com/engine/installation/linux/ubuntulinux/

这里以 `Ubuntu 16.04 LTS` 为例：

```sh
# 添加 PGP Key
sudo apt-key adv --keyserver hkp://ha.pool.sks-keyservers.net:80 --recv-keys 58118E89F3A912897C070ADBF76221572C52609D

# 添加更新源，其他 Linux 发行版请参照上面链接更换相应更新源
$ echo "deb https://apt.dockerproject.org/repo ubuntu-xenial main" | sudo tee /etc/apt/sources.list.d/docker.list

# 安装 Docker
sudo apt-get update
sudo apt-get install docker-engine

# 建立 docker 组
# 默认情况下，docker 命令会使用 Unix socket 与 Docker 引擎通讯。而只有 root 用户和 docker 组的用户才可以访问 Docker 引擎的 Unix socket。出于安全考虑，一般 Linux 系统上不会直接使用 root 用户。因此，更好地做法是将需要使用 docker 的用户加入 docker 用户组
$ sudo groupadd docker

# 将当前用户加入 docker 组
$ sudo usermod -aG docker $USER
```

### 安装 docker-compose

> 参照 https://yeasy.gitbooks.io/docker_practice/content/compose/intro.html

Docker-compose 可以很方便地实现 fabric 的一键部署。

因为 Ubuntu 仓库中的 docker-compose 版本较低，建议到 [这里][4] 自己下载最新版，此时最新版为 `1.9.0`。

以下命令使用 root 权限执行：

```sh
$ su # 使用 root 权限
$ curl -L https://github.com/docker/compose/releases/download/1.9.0/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose
$ chmod +x /usr/local/bin/docker-compose
```

### 获取官方 fabric 镜像

```sh
$ docker pull hyperledger/fabric-peer:latest
$ docker pull hyperledger/fabric-membersrvc:latest
```

## 0x02 启动 fabric 与测试链码

> 参考 https://github.com/hyperledger/fabric/blob/master/docs/Setup/Chaincode-setup.md

### 启动 fabric

把以下内容保存到 `docker-compose.yml` 文件，使用 `docker-compose up` 命令启动一个认证节点和成员管理服务：

```sh
membersrvc:
  image: hyperledger/fabric-membersrvc
  command: membersrvc
vp0:
  image: hyperledger/fabric-peer
  environment:
    - CORE_PEER_ADDRESSAUTODETECT=true
    - CORE_VM_ENDPOINT=http://172.17.0.1:2375
    - CORE_LOGGING_LEVEL=DEBUG
    - CORE_PEER_ID=vp0
    - CORE_PEER_PKI_ECA_PADDR=membersrvc:7054
    - CORE_PEER_PKI_TCA_PADDR=membersrvc:7054
    - CORE_PEER_PKI_TLSCA_PADDR=membersrvc:7054
    - CORE_SECURITY_ENABLED=true
    - CORE_SECURITY_ENROLLID=test_vp0
    - CORE_SECURITY_ENROLLSECRET=MwYpmSRjupbT
  links:
    - membersrvc
  command: sh -c "sleep 5; peer node start --peer-chaincodedev"
```

### 在节点上注册链码

进入 docker，在认证节点上注册链码，后续与链码交互过程中，可以在终端中查看链码的打印日志。

查看 docker 中当前进程：
```sh
$ docker ps
```

输出结果：
```sh
CONTAINER ID        IMAGE                           COMMAND                  CREATED             STATUS              PORTS               NAMES
4e864338032e        hyperledger/fabric-peer         "sh -c 'sleep 5; peer"   54 minutes ago      Up 54 minutes                           fabricdockercompse_vp0_1
92e4c255c47b        hyperledger/fabric-membersrvc   "membersrvc"             54 minutes ago      Up 54 minutes                           fabricdockercompse_membersrvc_1
```

这时，可以使用 `docker cp` 命令把链码复制到节点中：
```sh
docker cp chaincode_example02.go mycontainer:/opt/gopath/src/github.com/chaincode_example02/chaincode_example02.go
```

进入认证节点，启动节点的终端：
```
# -t 选项让 docker 分配一个伪终端（pseudo-tty）并绑定到容器的标准输入上， -i 则让容器的标准输入保持打开
docker exec -it fabricdockercompse_vp0_1 bash
```

编译链码：
```sh
cd $GOPATH/src/github.com/chaincode_example02
go build
```

在节点上注册链码：
```sh
# mycc 为自定义链码的 ID，后续与链码交互时会用到
# chaincode_example02 为编译后的链码可执行文件
CORE_CHAINCODE_ID_NAME=mycc CORE_PEER_ADDRESS=0.0.0.0:7051 ./chaincode_example02
```

### 使用 REST 与链码交互

使用 Postman 或类似工具完成下面的测试。

用户登录：
```
POST http://172.17.0.3:7050/registrar

{
  "enrollId": "jim",
  "enrollSecret": "6avZQLwcUe9b"
}
```

部署链码，会调用链码的 `Init()` 方法，创建 a、b 两个角色，余额分别为 100 和 200 单位：
```
POST http://172.17.0.3:7050/chaincode

{
  "jsonrpc": "2.0",
  "method": "deploy",
  "params": {
    "type": 1,
    "chaincodeID":{
      "name": "mycc"
    },
    "ctorMsg": {
       "args":["init", "a", "100", "b", "200"]
    },
    "secureContext": "jim"
  },
  "id": 1
}
```

从 a 向 b 转账 10 个单位，会调用链码的 `Invoke()` 方法：
```
POST http://172.17.0.3:7050/chaincode

{
  "jsonrpc": "2.0",
  "method": "invoke",
  "params": {
      "type": 1,
      "chaincodeID":{
        "name":"mycc"
      },
      "ctorMsg": {
        "args":["invoke", "a", "b", "10"]
      },
      "secureContext": "jim"
  },
  "id": 3
}
```

查询 a 的余额，会调用链码的 `Query()` 方法：
```
POST http://172.17.0.3:7050/chaincode

{
  "jsonrpc": "2.0",
  "method": "query",
  "params": {
      "type": 1,
      "chaincodeID":{
        "name":"mycc"
      },
      "ctorMsg": {
        "args":["query", "a"]
      },
      "secureContext": "jim"
  },
  "id": 5
}
```

## 0x03 参考内容
1. https://github.com/hyperledger/fabric
2. https://docs.docker.com/engine/installation/linux/ubuntulinux/
3. https://yeasy.gitbooks.io/docker_practice/content/compose/intro.html
4. https://github.com/hyperledger/fabric/blob/master/docs/Setup/Chaincode-setup.md


  [1]: https://g2ex.github.io/2016/10/14/Deploy-Hyperledger-Fabric-with-Docker/
  [2]: http://yeasy.github.io/
  [3]: https://github.com/hyperledger/fabric/tree/master/examples/chaincode/go/chaincode_example02
  [4]: https://github.com/docker/compose/releases