title: "Ubuntu 中使用 Docker 部署 Hyperledger Fabric"
date: 2016-10-14 12:00:00
tag:
- Fabric
- Hyperledger
- Blockchain
- Docker
permalink: Deploy-Hyperledger-Fabric-with-Docker
---

本文总结如何使用 Docker 部署 Hyperledger 的 fabric。并使用 fabric 的 CLI 接口命令部署链码和进行交易。

## 准备工作

准备一台 Ubuntu 14.04 LTS 主机，规格如下：

| Role       | RAM         | Disk            | CPUs       | IP Address |
|------------|-------------|-----------------|------------|------------|
| hyperledger| 4 GB        | 40 GB           | 双核       | 172.16.1.78|

使用以下命令安装 `Docker Engine`：
```bash
$ sudo apt-get install -y python-pip git
$ curl http://files.imaclouds.com/scripts/docker_install.sh | sh
```

接下来，使用以下命令安装 `docker-compose`：
```bash
$ sudo pip install --upgrade pip
$ sudo pip install docker-compose
```

注意！如果上述命令安装的 `docker-compose` 不是最新版，在运行 `docker-compose up` 命令时会出现错误。可以按照 https://github.com/docker/compose/releases 的方法手动安装：
```bash
curl -L https://github.com/docker/compose/releases/download/1.8.0-rc2/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose
chmod +x /usr/local/bin/docker-compose
```

## 安装 Hyperledger

安装前首先通过 git 下载 Hyperledger compose：
```bash
$ git clone https://github.com/yeasy/docker-compose-files.git
$ cd docker-compose-files/hyperledger
```

执行以下步骤来下载 docker 镜像：
```bash
$ docker pull openblockchain/baseimage:0.0.9
$ docker pull yeasy/hyperledger:latest
$ docker tag yeasy/hyperledger:latest hyperledger/fabric-baseimage:latest
$ docker pull yeasy/hyperledger-peer:noops
$ docker pull yeasy/hyperledger-peer:pbft
$ docker pull yeasy/hyperledger-membersrvc:latest
```

执行 `docker compose` 来部署四个节点的 Hyperledger：
```bash
$ docker-compose up
Attaching to hyperledger_vp0_1, hyperledger_vp3_1, hyperledger_vp1_1, hyperledger_vp2_1
vp0_1  | 07:42:07.870 [crypto] main -> INFO 001 Log level recognized 'info', set to INFO
...
```
 
## 验证 Hyperledger peer

进入到第一个 Hyperledger 容器：
```bash
$ docker exec -ti hyperledger_vp0_1 bash
```

查看目前所有建立的节点：
```bash
$ peer node status
08:09:14.715 [crypto] main -> INFO 001 Log level recognized 'info', set to INFO
08:09:14.715 [logging] LoggingInit -> DEBU 002 Setting default logging level to DEBUG for command 'node'
08:09:14.715 [peer] func1 -> INFO 003 Auto detected peer address: 172.17.0.2:30303
08:09:14.716 [peer] func1 -> INFO 004 Auto detected peer address: 172.17.0.2:30303
08:09:14.716 [peer] func1 -> INFO 005 Auto detected peer address: 172.17.0.2:30303
status:STARTED
```
 
进入容器后，首先部署一个链码（chaincode）：
```bash
$ peer chaincode deploy -p github.com/hyperledger/fabric/examples/chaincode/go/chaincode_example02 \
-c '{"Function":"init", "Args": ["Alice","100", "Bob", "200"]}'
...
81a73fa1fabe6e385f3c609cef8915a732ee74179abde55f4ac7addf4e7c35ac4a669a7d9a17b2c9a6b3c28b45565b97dc69f4c8f53381ba13251adf5ac6d23d
```
> 上面会获得一个 128 数字字母组成的哈希字符串，该字符串是链码 ID，在后续的交易和查询会使用到。

首先查询 Alice 的账户余额：
```bash
$ my_key="81a73fa1fabe6e385f3c609cef8915a732ee74179abde55f4ac7addf4e7c35ac4a669a7d9a17b2c9a6b3c28b45565b97dc69f4c8f53381ba13251adf5ac6d23d"
$ peer chaincode query -n ${my_key} \
-c '{"Function": "query", "Args": ["Alice"]}'
...
100
```

接着发起一笔交易，让 Alice 支付 10 个单位给 Bob：
```bash
$ peer chaincode invoke -n ${my_key} \
-c '{"Function": "invoke", "Args": ["Alice", "Bob", "10"]}'
```

确认完成交易后，可以查看 Bob 的账户余额：
```bash
$ peer chaincode query -n ${my_key} \
-c '{"Function": "query", "Args": ["Bob"]}'
```

## 参考内容

https://yeasy.gitbooks.io/blockchain_guide/content/hyperledger/install.html
https://github.com/kairen/blockchain-tutorial/blob/master/hyperledger/hyperledger-docker.md
https://github.com/docker/compose/releases