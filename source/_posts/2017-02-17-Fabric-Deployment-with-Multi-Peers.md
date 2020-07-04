title: "Fabric 多节点部署小结"
date: 2017-02-17 20:00:00
tag:
- Fabric
- Hyperledger
- Blockchain
permalink: Fabric-Deployment-with-Multi-Peers
---

## 0x00 说明

上篇文章《[Fabric 单节点部署与链码测试][1]》中，总结了单认证节点的 fabric 区块链网络的部署和测试。在 fabric 中，单认证节点的网络不使用共识算法，主要用于测试和调试链码。在实际应用场景的 demo 中，有必要使用多节点进行演示，节点之间使用的是推荐的 PBFT 共识算法。

Fabric 目前尚未发布 1.0 正式版。本文仍然基于 Ubuntu 平台，使用 fabric 的 0.6 版本 docker 镜像，部署有 4 个认证节点的区块链网络。（参考《[区块链技术指南][2]》）

## 0x01 获取 Docker 镜像

参照《[Fabric 单节点部署与链码测试][1]》安装 docker 和 docker-compose。

### 配置 docker

Docker 安装完成后需要进行配置。需要注意的是，Ubuntu 14.04 和 16.04 中，docker 的配置文件不同。

Ubuntu 14.04 中，配置文件位于 `/etc/default/docker`，Ubuntu 16.04 中，配置文件位于 `/etc/systemd/system/docker.service.d/override.conf` 中（如果配置文件不存在则创建）。

在 docker 的配置文件中，修改以下配置：
```
DOCKER_OPTS="$DOCKER_OPTS -H tcp://0.0.0.0:2375 -H unix:///var/run/docker.sock --api-cors-header='*'"
```

### 加速 docker 镜像

因为网络原因，直接从 docker 官方下载速度缓慢，或时而中断。阿里云容器 Hub 服务提供了加速功能。登录[阿里云容器 Hub 服务][3]，在**加速器**一栏中获取自己的加速器地址。

![docker-accelerator][4]

参照阿里云操作文档，修改 docker 的配置。

本文使用的是 Ubuntu 16.04，docker 版本为 1.13，修改方法如下。

创建 `/etc/docker/daemon.json` 文件，添加以下内容：
```
{
  "registry-mirrors": ["你的加速器地址"]
}
```

重启 docker：
```
$ sudo systemctl daemon-reload
$ sudo systemctl restart docker
```

### 获取 fabric 镜像

这里获取 fabric 官方的镜像：
```
$ docker pull hyperledger/fabric-peer:x86_64-0.6.1-preview \
&& docker pull hyperledger/fabric-membersrvc:x86_64-0.6.1-preview \
&& docker pull yeasy/blockchain-explorer:latest \
&& docker tag hyperledger/fabric-peer:x86_64-0.6.1-preview hyperledger/fabric-peer \
&& docker tag hyperledger/fabric-peer:x86_64-0.6.1-preview hyperledger/fabric-baseimage \
&& docker tag hyperledger/fabric-membersrvc:x86_64-0.6.1-preview hyperledger/fabric-membersrvc
```

## 0x02 启动集群并部署

### 使用 docker-compse 启动集群

使用 docker-compose 更方便的启动多容器节点，下载 Compose 模板文件：
```
$ git clone https://github.com/yeasy/docker-compose-files
```

进入 `hyperledger/0.6/pbft` 目录，以启动 4 个 PBFT 认证节点 + 1 个 CA 节点为例，启动集群：
```
$ docker-compose -f 4-peers-with-membersrvc.yml up
```

### 把链码复制到认证节点中

集群中 VP0 是 Root VP 节点，进入 VP0，可以看到 fabric 路径：
```
$ docker exec -it pbft_vp0_1 bash
root@vp0:/opt/gopath/src/github.com/hyperledger/fabric#
```

以示例 02 为例，把链码复制到 VP0 节点：
```
$ docker cp chaincode_example02/ pbft_vp0_1:/opt/gopath/src/github.com/
```

### 通过 REST API 部署链码

注册账户登录：
```
POST http://127.0.0.1:7050/registrar

{
  "enrollId": "jim",
  "enrollSecret": "6avZQLwcUe9b"
}
```

登录响应：
```
{
  "OK": "Login successful for user 'jim'."
}
```

部署链码：
```
POST http://127.0.0.1:7050/chaincode

{
  "jsonrpc": "2.0",
  "method": "deploy",
  "params": {
    "type": 1,
    "chaincodeID":{
        "path": "github.com/chaincode_example02"
    },
    "ctorMsg": {
        "args":["init"]
    },
    "secureContext": "jim"
  },
  "id": 1
}
```

部署响应：
```
{
  "jsonrpc": "2.0",
  "result": {
    "status": "OK",
    "message": "28a49ca95eb0e0dbf3ee7e41ff2172c174b13aa1539db3c4977bb0b7a78d26cb4a0ea26252358ddc184f8d44172af1284dc3f914caa13ccd2137611c70059b59"
  },
  "id": 1
}
```

记下 `message` 字段，在后续调用、查询链码的时候填入 `name` 字段。

## 0x03 总结

这种部署方法与单 VP 节点节点中编译运行部署差别：

1. REST API 部署的 JSON 数据 `chaincodeID` 字段不同，这里为 `path`，是链码在容器中的路径；单 VP 节点部署时 `chaincodeID` 内字段为 `name`，是链码编译后的可执行文件名；
2. 单 VP 节点部署是为了调试链码，做法是把链码复制到 VP 节点中并编译，通过 `CORE_CHAINCODE_ID_NAME=mycc CORE_PEER_ADDRESS=0.0.0.0:7051 ./chaincode_example02` 命令运行链码，在终端中查看调试信息，使用 REST API 完成部署操作；本文是把链码复制到 Root VP 节点中，使用 REST API 部署时广播到区块链网络中所有 VP 节点，自行编译链码。


  [1]: https://g2ex.github.io/2016/11/26/Fabric-Deployment-and-Chaincode-Setup/
  [2]: https://yeasy.gitbooks.io/blockchain_guide/content/hyperledger/install.html
  [3]: https://cr.console.aliyun.com/
  [4]: https://cdn.jsdelivr.net/gh/gymgle/imgur/2017-02-17_152821.webp
