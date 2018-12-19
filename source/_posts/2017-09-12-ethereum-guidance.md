title: "以太坊私有链搭建指南"
date: 2017-09-12 0:0:0
tag:
- 以太坊
- Ethereum
- Solidity
- Blockchain
permalink: ethereum-guidance
---

## 说明

### 一、为什么用到私有链？

在以太坊的共有链上部署智能合约、发起交易需要花费以太币。而通过修改配置，可以在本机搭建一套以太坊私有链，因为与公有链没关系，既不用同步公有链庞大的数据，也不用花钱购买以太币，很好地满足了智能合约开发和测试的要求，开发好的智能合约也可以很容易地切换接口部署到以太坊公有链上。

### 二、需要用到哪些工具？

1. 以太坊客户端
  以太坊客户端用于接入以太坊网络，进行账户管理、交易、挖矿、智能合约相关的操作。目前有多种语言实现的客户端，常用的有 Go 语言实现的 go-ethereum 客户端 Geth，支持接入以太坊网络并成为一个完整节点，也可作为一个 HTTP-RPC 服务器对外提供 JSON-RPC 接口。
  其他的客户端有：
   * Parity：Rust 语言实现；
   * cpp-ethereum：C++ 语言实现；
   * ethereumjs-lib：JavaScript 语言实现；
   * Ethereum(J)：Java 语言实现；
   * ethereumH：Haskell 语言实现；
   * pyethapp： Python 语言实现；
   * ruby-ethereum：Ruby 语言实现；
2. 智能合约编译器
  以太坊支持两种智能合约的编程语言：Solidity 和 Serpent。Serpent 语言面临一些安全问题，现在已经不推荐使用了。Solidity 语法类似 JavaScript，它编译器 solc 可以把智能合约源码编译成以太坊虚拟机 EVM 可以执行的二进制码。
  现在以太坊提供更方便的在线 IDE —— Remix https://remix.ethereum.org 使用 Remix，免去了安装 solc 和编译过程，它可以直接提供部署合约所需的二进制码和 ABI。
3. 以太坊钱包
  以太坊提供了图形界面的钱包 Ethereum Wallet 和 Mist Dapp 浏览器。钱包的功能是 Mist 的一个子集，可用于管理账户和交易；Mist 在钱包基础上，还能操作智能合约。为了演示合约部署过程，本文使用了 Geth console 操作，没有用到 Mist，当然，使用 Mist 会更简单。

## 安装环境

这里以 Ubuntu 16.04 为例进行介绍。

### 一、安装以太坊客户端

两种方式可选：PPA 直接安装、源码编译安装。

1. PPA  直接安装
  安装必要的工具包：
  ```
  apt install software-properties-common
  ```
  添加以太坊源：
  ```
  add-apt-repository -y ppa:ethereum/ethereum
  apt update
  ```
  安装 go-ethereum：
  ```
  apt install ethereum
  ```

  安装完成后，可以使用 `geth version` 命令查看是否安装成功。

2. 源码安装
  因为 go-ethereum 使用 Go 语言编写，编译源码前需要配置好 Go 环境。

  **配置 Go 语言环境**
  参照 https://golang.org/doc/install
  简单地，以安装 go 1.9 为例：
  ```
  wget https://storage.googleapis.com/golang/go1.9.linux-amd64.tar.gz
  tar -C /usr/local -xzf go1.9.linux-amd64.tar.gz
  ```
  编辑 `/etc/profile` 或 `$HOME/.profile`，在文件最后添加环境变量：
  ```
  export PATH=$PATH:/usr/local/go/bin
  ```
  编辑 `$HOME/profile` 或 `$HOME/.bashrc`，添加 go 工作目录环境变量：
  ```
  export GOPATH=$HOME/gopath
  ```
  **下载和编译 Geth**
  安装 C 编译器：
  ```
  apt install -y build-essential
  ```
  下载最新源码：
  ```
  git clone https://github.com/ethereum/go-ethereum
  ```
  编译安装：
  ```
  cd go-ethereum
  make geth
  ```

  安装完成后，可以使用 `geth version` 命令查看是否安装成功。记得把生成的 geth 加入到系统的环境变量中。

### 二、安装 Solidity 编译器

Solidity 编译器也有多种方法安装，参照 http://solidity.readthedocs.io/en/latest/installing-solidity.html 这里介绍最简单快捷的安装方式：PPA 直接安装。

1.  PPA 直接安装

  ```
  add-apt-repository ppa:ethereum/ethereum
  apt update
  apt install solc
  ```
2.  官方推荐使用基于浏览器的 IDE 环境：Remix https://remix.ethereum.org

## 私有链搭建

### 一、配置初始状态

要运行以太坊私有链，需要定义自己的创世区块，创世区块信息写在一个 JSON 格式的配置文件中。首先将下面的内容保存到一个 JSON 文件中，例如 `genesis.json`。

```
{
  "config": {
    "chainID": 1024,
    "homesteadBlock": 0,
    "eip155Block": 0,
    "eip158Block": 0
  },
  "alloc": {},
  "coinbase": "0x0000000000000000000000000000000000000000",
  "difficulty": "0x400",
  "extraData": "0x0",
  "gasLimit": "0x2fefd8",
  "nonce": "0xdeadbeefdeadbeef",
  "mixhash": "0x0000000000000000000000000000000000000000000000000000000000000000",
  "parentHash": "0x0000000000000000000000000000000000000000000000000000000000000000",
  "timestamp": "0x00"
}
```

其中，`chainID` 指定了独立的区块链网络 ID。网络 ID 在连接到其他节点的时候会用到，以太坊公网的网络 ID 是 1，为了不与公有链网络冲突，运行私有链节点的时候要指定自己的网络 ID。不同 ID 网络的节点无法相互连接。配置文件还对当前挖矿难度 `difficulty`、区块 Gas 消耗限制 `gasLimit` 等参数进行了设置。

在 Geth 1.6+ 中，以太坊提供了一个生成创世块的向导工具：puppeth。并且提供了更适合在私有链中使用的 Clique PoA 共识算法。puppeth 的使用，可以参照《[利用puppeth搭建POA共识的以太坊私链网络][1]》

### 二、初始化：写入创世区块
准备好创世区块配置文件后，需要初始化区块链，将上面的创世区块信息写入到区块链中。首先要新建一个目录用来存放区块链数据，假设新建的数据目录为 `~/privatechain/data0`，`genesis.json` 保存在 `~/privatechain` 中，此时目录结构应该是这样的：
```
privatechain
├── data0
└── genesis.json
```

接下来进入 privatechain 中，执行初始化命令：
```
cd privatechain
geth --datadir data0 init genesis.json
```

上面的命令的主体是 `geth init`，表示初始化区块链，命令可以带有选项和参数，其中 `--datadir` 选项后面跟一个目录名，这里为 data0，表示指定数据存放目录为 data0，genesis.json 是 `init` 命令的参数。

运行上面的命令，会读取 `genesis.json` 文件，根据其中的内容，将创世区块写入到区块链中。如果看到以下的输出内容，说明初始化成功了。

```
WARN [09-12|04:01:09] No etherbase set and no accounts found as default
INFO [09-12|04:01:09] Allocated cache and file handles         database=/root/work/privatechain/data0/geth/chaindata cache=16 handles=16
INFO [09-12|04:01:09] Writing custom genesis block
INFO [09-12|04:01:09] Successfully wrote genesis state         database=chaindata                                    hash=84e71d…97246e
INFO [09-12|04:01:09] Allocated cache and file handles         database=/root/work/privatechain/data0/geth/lightchaindata cache=16 handles=16
INFO [09-12|04:01:09] Writing custom genesis block
INFO [09-12|04:01:09] Successfully wrote genesis state         database=lightchaindata                                    hash=84e71d…97246e
```

初始化成功后，会在数据目录 data0 中生成 `geth` 和 `keystore` 两个文件夹，此时目录结构如下：
```
privatechain
├── data0
│   ├── geth
│   │   ├── chaindata
│   │   │   ├── 000001.log
│   │   │   ├── CURRENT
│   │   │   ├── LOCK
│   │   │   ├── LOG
│   │   │   └── MANIFEST-000000
│   │   └── lightchaindata
│   │       ├── 000001.log
│   │       ├── CURRENT
│   │       ├── LOCK
│   │       ├── LOG
│   │       └── MANIFEST-000000
│   └── keystore
└── genesis.json
```

其中 `geth/chaindata` 中存放的是区块数据，`keystore` 中存放的是账户数据。

### 三、启动私有链节点

初始化完成后，就有了一条自己的私有链，之后就可以启动自己的私有链节点并做一些操作，在终端中输入以下命令即可启动节点：

```
geth --identity "TestNode" --rpc --rpcport "8545" --datadir data0 --port "30303" --nodiscover console
```

上面命令的主体是 `geth console`，表示启动节点并进入交互式控制台。
各选项含义如下：

* --identity：指定节点 ID；
* --rpc：表示开启 HTTP-RPC 服务；
* --rpcport：指定 HTTP-RPC 服务监听端口号（默认为 8545）；
* --datadir：指定区块链数据的存储位置；
* --port：指定和其他节点连接所用的端口号（默认为 30303）；
* --nodiscover：关闭节点发现机制，防止加入有同样初始配置的陌生节点。

运行上面的命令后，就启动了区块链节点并进入了该节点的控制台：

```
...
Welcome to the Geth JavaScript console!

instance: Geth/TestNode/v1.6.7-stable-ab5646c5/linux-amd64/go1.8.1
 modules: admin:1.0 debug:1.0 eth:1.0 miner:1.0 net:1.0 personal:1.0 rpc:1.0 txpool:1.0 web3:1.0
```

这是一个交互式的 JavaScript 执行环境，在这里面可以执行 JavaScript 代码，其中 `>` 是命令提示符。在这个环境里也内置了一些用来操作以太坊的 JavaScript 对象，可以直接使用这些对象。这些对象主要包括：

* eth：包含一些跟操作区块链相关的方法；
* net：包含一些查看p2p网络状态的方法；
* admin：包含一些与管理节点相关的方法；
* miner：包含启动&停止挖矿的一些方法；
* personal：主要包含一些管理账户的方法；
* txpool：包含一些查看交易内存池的方法；
* web3：包含了以上对象，还包含一些单位换算的方法。

## 控制台操作

进入以太坊 Javascript Console 后，就可以使用里面的内置对象做一些操作，这些内置对象提供的功能很丰富，比如查看区块和交易、创建账户、挖矿、发送交易、部署智能合约等。

常用命令有：

* personal.newAccount()：创建账户；
* personal.unlockAccount()：解锁账户；
* eth.accounts：枚举系统中的账户；
* eth.getBalance()：查看账户余额，返回值的单位是 Wei（Wei 是以太坊中最小货币面额单位，类似比特币中的`聪`，1 ether = 10^18 Wei）；
* eth.blockNumber：列出区块总数；
* eth.getTransaction()：获取交易；
* eth.getBlock()：获取区块；
* miner.start()：开始挖矿；
* miner.stop()：停止挖矿；
* web3.fromWei()：Wei 换算成以太币；
* web3.toWei()：以太币换算成 Wei；
* txpool.status：交易池中的状态；
* admin.addPeer()：连接到其他节点；

这些命令支持 `Tab` 键自动补全，具体用法如下。

### 一、创建账户
```
> personal.newAccount()
Passphrase:
Repeat passphrase:
"0x3443ffb2a5ce3f4b80080791e0fde16a3fac2802"
> INFO [09-12|05:55:44] New wallet appeared                     url=keystore:///root/work/privatech… status=Locked
```

输入两遍密码后，会生成账户地址。

再创建一个账户：
```
> personal.newAccount()
Passphrase:
Repeat passphrase:
"0x02bee2a1582bbf58c42bbdfe7b8db4685d4d4c62"
> INFO [09-12|06:10:45] New wallet appeared                     url=keystore:///root/work/privatech… status=Locked
```

查看刚刚创建的两个账户：
```
> eth.accounts
["0x3443ffb2a5ce3f4b80080791e0fde16a3fac2802", "0x02bee2a1582bbf58c42bbdfe7b8db4685d4d4c62"]
```

### 二、查看账户余额

```
> eth.getBalance(eth.accounts[0])
0
> eth.getBalance(eth.accounts[1])
0
```

### 三、启动&停止挖矿

启动挖矿：
```
> miner.start(1)
```

其中 start 的参数表示挖矿使用的线程数。第一次启动挖矿会先生成挖矿所需的 DAG 文件，这个过程有点慢，等进度达到 100% 后，就会开始挖矿，此时屏幕会被挖矿信息刷屏。

停止挖矿，在 console 中输入：
```
> miner.stop()
```

挖到一个区块会奖励5个以太币，挖矿所得的奖励会进入矿工的账户，这个账户叫做 coinbase，默认情况下 coinbase 是本地账户中的第一个账户，可以通过 miner.setEtherbase() 将其他账户设置成 coinbase。

### 四、发送交易

目前，账户 0 已经挖到了 3 个块的奖励，账户 1 的余额还是0：
```
> eth.getBalance(eth.accounts[0])
15000000000000000000
> eth.getBalance(eth.accounts[1])
0
```

我们要从账户 0 向账户 1 转账，所以要先解锁账户 0，才能发起交易：
```
> personal.unlockAccount(eth.accounts[0])
Unlock account 0x3443ffb2a5ce3f4b80080791e0fde16a3fac2802
Passphrase: 
true
```

发送交易，账户 0 -> 账户 1：
```
> amount = web3.toWei(5,'ether')
"5000000000000000000"
> eth.sendTransaction({from:eth.accounts[0],to:eth.accounts[1],value:amount})
INFO [09-12|07:38:12] Submitted transaction                    fullhash=0x9f5e61f3d686f793e2df6378d1633d7a9d1df8ec8c597441e1355112d102a6ce recipient=0x02bee2a1582bbf58c42bbdfe7b8db4685d4d4c62
"0x9f5e61f3d686f793e2df6378d1633d7a9d1df8ec8c597441e1355112d102a6ce"
```

此时如果没有挖矿，用 `txpool.status` 命令可以看到本地交易池中有一个待确认的交易，可以使用 `eth.getBlock("pending", true).transactions` 查看当前待确认交易。

使用 `miner.start()` 命令开始挖矿：
```
> miner.start(1);admin.sleepBlocks(1);miner.stop();
```

新区块挖出后，挖矿结束，查看账户 1 的余额，已经收到了账户 0 的以太币：
```
> web3.fromWei(eth.getBalance(eth.accounts[1]),'ether')
5
```

### 五、查看交易和区块

查看当前区块总数：
```
> eth.blockNumber
4
```

通过交易 Hash 查看交易（Hash 值包含在上面交易返回值中）：
```
> eth.getTransaction("0x9f5e61f3d686f793e2df6378d1633d7a9d1df8ec8c597441e1355112d102a6ce")
{
  blockHash: "0xdc1fb4469bf4613821c303891a71ff0d1f5af9af8c10efdd8bcd8b518533ee7d",
  blockNumber: 4,
  from: "0x3443ffb2a5ce3f4b80080791e0fde16a3fac2802",
  gas: 90000,
  gasPrice: 18000000000,
  hash: "0x9f5e61f3d686f793e2df6378d1633d7a9d1df8ec8c597441e1355112d102a6ce",
  input: "0x",
  nonce: 0,
  r: "0x4214d2d8d92efc3aafb515d2413ecd45ab3695d9bcc30d9c7c06932de829e064",
  s: "0x42822033225a2ef662b9b448576e0271b9958e1f4ec912c259e01c84bd1f6681",
  to: "0x02bee2a1582bbf58c42bbdfe7b8db4685d4d4c62",
  transactionIndex: 0,
  v: "0x824",
  value: 5000000000000000000
}
```

通过区块号查看区块：
```
> eth.getBlock(4)
{
  difficulty: 131072,
  extraData: "0xd783010607846765746887676f312e382e31856c696e7578",
  gasLimit: 3153874,
  gasUsed: 21000,
  hash: "0xdc1fb4469bf4613821c303891a71ff0d1f5af9af8c10efdd8bcd8b518533ee7d",
  logsBloom: "0x00000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000",
  miner: "0x3443ffb2a5ce3f4b80080791e0fde16a3fac2802",
  mixHash: "0x6df88079cf4fbfae98ad7588926fa30becddf4b8b55f93f0380d82ce0533338c",
  nonce: "0x39455ee908666993",
  number: 4,
  parentHash: "0x14fe27755d6fcc704f6b7018d5dc8193f702d89f2c7807bf6f0e402a2b0a29d9",
  receiptsRoot: "0xfcb5b5cc322998562d96339418d08ad8e7c5dd87935f9a3321e040344e3fd095",
  sha3Uncles: "0x1dcc4de8dec75d7aab85b567b6ccd41ad312451b948a7413f0a142fd40d49347",
  size: 651,
  stateRoot: "0xdd4a0ce76c7e0ff149853dce5bb4f99592fb1bc3c5e87eb07518a0235ffacd8c",
  timestamp: 1505202063,
  totalDifficulty: 525312,
  transactions: ["0x9f5e61f3d686f793e2df6378d1633d7a9d1df8ec8c597441e1355112d102a6ce"],
  transactionsRoot: "0xbb909845183e037b15d24fe9ad1805fd00350ae04841aa774be6af96e76fdbf9",
  uncles: []
}
```

### 六、连接到其他节点

可以通过 `admin.addPeer()` 方法连接到其他节点，两个节点要要指定相同的 chainID。

假设有两个节点：节点一和节点二，chainID 都是 1024，通过下面的步骤就可以从节点一连接到节点二。

首先要知道节点二的 enode 信息，在节点二的 JavaScript console 中执行下面的命令查看 enode 信息：
```
> admin.nodeInfo.enode
"enode://d465bcbd5c34da7f4b8e00cbf9dd18e7e2c38fbd6642b7435f340c7d5168947ff2b822146e1dc1b07e02f7c15d5ca09249a92f1d0caa34587c9b2743172259ee@[::]:30303"
```

然后在节点一的 JavaScript console 中执行 admin.addPeer()，就可以连接到节点二：

```
> admin.addPeer("enode://d465bcbd5c34da7f4b8e00cbf9dd18e7e2c38fbd6642b7435f340c7d5168947ff2b822146e1dc1b07e02f7c15d5ca09249a92f1d0caa34587c9b2743172259ee@127.0.0.1:30304")
```

addPeer() 的参数就是节点二的 enode 信息，注意要把 enode 中的 `[::]` 替换成节点二的 IP 地址。连接成功后，节点二就会开始同步节点一的区块，同步完成后，任意一个节点开始挖矿，另一个节点会自动同步区块，向任意一个节点发送交易，另一个节点也会收到该笔交易。

通过 `admin.peers` 可以查看连接到的其他节点信息，通过 `net.peerCount` 可以查看已连接到的节点数量。

除了上面的方法，也可以在启动节点的时候指定 `--bootnodes` 选项连接到其他节点。

## 智能合约操作

### 一、创建和编译智能合约

新建一个 Solidity 智能合约文件，命名为 `testContract.sol`，该合约包含一个方法 `multiply()`，将输入的两个数相乘后输出：
```
pragma solidity ^0.4.0;
contract TestContract
{
    function multiply(uint a, uint b) returns (uint)
    {
        return a * b;
    }
}
```

编译智能合约，获得编译后的 EVM 二进制码：
```
$ solc --bin testContract.sol

======= testContract.sol:TestContract =======
Binary:
6060604052341561000f57600080fd5b5b60b48061001e6000396000f30060606040526000357c0100000000000000000000000000000000000000000000000000000000900463ffffffff168063165c4a1614603d575b600080fd5b3415604757600080fd5b60646004808035906020019091908035906020019091905050607a565b6040518082815260200191505060405180910390f35b600081830290505b929150505600a165627a7a72305820b494a4b3879b3810accf64d4cc3e1be55f2f4a86f49590b8a9b8d7009090a5d30029
```

再用 solc 获取智能合约的 JSON ABI（Application Binary Interface），其中指定了合约接口，包括可调用的合约方法、变量、事件等：
```
$ solc --abi testContract.sol

======= testContract.sol:TestContract =======
Contract JSON ABI
[{"constant":false,"inputs":[{"name":"a","type":"uint256"},{"name":"b","type":"uint256"}],"name":"multiply","outputs":[{"name":"","type":"uint256"}],"payable":false,"stateMutability":"nonpayable","type":"function"}]
```

回到 Geth 的控制台，用变量 `code` 和 `abi` 记录上面两个值，注意在 code 前加上 `0x` 前缀：
```
> code = "0x6060604052341561000f57600080fd5b5b60b48061001e6000396000f30060606040526000357c0100000000000000000000000000000000000000000000000000000000900463ffffffff168063165c4a1614603d575b600080fd5b3415604757600080fd5b60646004808035906020019091908035906020019091905050607a565b6040518082815260200191505060405180910390f35b600081830290505b929150505600a165627a7a72305820b494a4b3879b3810accf64d4cc3e1be55f2f4a86f49590b8a9b8d7009090a5d30029"
> abi = [{"constant":false,"inputs":[{"name":"a","type":"uint256"},{"name":"b","type":"uint256"}],"name":"multiply","outputs":[{"name":"","type":"uint256"}],"payable":false,"stateMutability":"nonpayable","type":"function"}]
```

### 二、部署智能合约

这里使用账户 0 来部署合约，首先解锁账户：
```
> personal.unlockAccount(eth.accounts[0])
Unlock account 0x3443ffb2a5ce3f4b80080791e0fde16a3fac2802
Passphrase:
true
```

发送部署合约的交易：
```
> myContract = eth.contract(abi)
...
> contract = myContract.new({from:eth.accounts[0],data:code,gas:1000000})

INFO [09-12|08:05:19] Submitted contract creation              fullhash=0x0a7dfa9cac7ef836a72ed1d5bbfa65c0220347cde4efb067a0b03b15fb70bce1 contract=0x7cbe4019e993f9922b8233502d94890099ee59e6
{
  abi: [{
      constant: false,
      inputs: [{...}, {...}],
      name: "multiply",
      outputs: [{...}],
      payable: false,
      stateMutability: "nonpayable",
      type: "function"
  }],
  address: undefined,
  transactionHash: "0x0a7dfa9cac7ef836a72ed1d5bbfa65c0220347cde4efb067a0b03b15fb70bce1"
}
```

此时如果没有挖矿，用 `txpool.status` 命令可以看到本地交易池中有一个待确认的交易。使用下面的命令查看当前待确认的交易：
```
[{
    blockHash: "0xfedd6fef9f25e96a5a20b5ffcd152e9fe05d193ae0989c25d6197d2441c2c09b",
    blockNumber: 5,
    from: "0x3443ffb2a5ce3f4b80080791e0fde16a3fac2802",
    gas: 1000000,
    gasPrice: 18000000000,
    hash: "0x0a7dfa9cac7ef836a72ed1d5bbfa65c0220347cde4efb067a0b03b15fb70bce1",
    input: "0x6060604052341561000f57600080fd5b5b60b48061001e6000396000f30060606040526000357c0100000000000000000000000000000000000000000000000000000000900463ffffffff168063165c4a1614603d575b600080fd5b3415604757600080fd5b60646004808035906020019091908035906020019091905050607a565b6040518082815260200191505060405180910390f35b600081830290505b929150505600a165627a7a72305820b494a4b3879b3810accf64d4cc3e1be55f2f4a86f49590b8a9b8d7009090a5d30029",
    nonce: 3,
    r: "0xfe7139a31694a36f946e7182c35c52a21bf31d33994490815f63f6674e84dc93",
    s: "0x4701a984ee93d2a323fac55547b31534b11915599de75202a1d62061241fefbf",
    to: null,
    transactionIndex: 0,
    v: "0x823",
    value: 0
}]
```

使用 `miner.start()` 命令开始挖矿，一段时间后交易会被确认，随新区块进入区块链。

### 三、调用智能合约

使用以下命令发送交易，`sendTransaction` 方法的前几个参数应该与合约中 `multiply` 方法的输入参数对应。这种情况下，交易会通过挖矿记录到区块链中：
```
> contract.multiply(2, 4, {from:eth.accounts[0]})

INFO [09-12|08:24:14] Submitted transaction                    fullhash=0x29b47d580ba6ccb2445aa3ebdcb14567bd5cbc6004edef7a4064c36e0606bca2 recipient=0x7cbe4019e993f9922b8233502d94890099ee59e6
"0x29b47d580ba6ccb2445aa3ebdcb14567bd5cbc6004edef7a4064c36e0606bca2"
```

如果只是本地运行该方法查看返回结果，可以采用如下方式：
```
> contract.multiply(2，4)
8
```

## 术语说明

术语 |  | 说明
-----|-----
ABI | Application Binary Interface，应用二进制接口 | 其中指定了合约接口，包括可调用的合约方法、变量、事件等。
DApp | Decentralized App，去中心化的应用程序 | 基于智能合约的应用称为去中心化的应用程序。
EVM | Ethereum Virtual Machine，以太坊虚拟机 | 以太坊智能合约的运行环境。
Gas | （消耗的）汽油 | 在以太坊上发起交易、部署合约和调用合约都要消耗一定量的以太币，这些消耗的以太币称为 Gas。
Geth | - | 以太坊客户端 go-ethereum，使用 Go 语言编写，是最常用的以太坊客户端之一。
Solidity | - | 以太坊智能合约的一种编程语言，类似 JavaScript。
Remix IDE | https://remix.ethereum.org | 基于浏览器的 Solidity 集成开发环境，在浏览器中编写和调试智能合约。


## 参考内容

* 《区块链——原理、设计与应用》杨保华、陈昌编著，机械工业出版
* 以太坊学习笔记：私有链搭建操作指南 https://my.oschina.net/u/2349981/blog/865256
* 利用puppeth搭建POA共识的以太坊私链网络 https://github.com/xiaoping378/blog/blob/master/posts/%E4%BB%A5%E5%A4%AA%E5%9D%8A-%E7%A7%81%E6%9C%89%E9%93%BE%E6%90%AD%E5%BB%BA%E5%88%9D%E6%AD%A5%E5%AE%9E%E8%B7%B5.md
* 区块链技术指南 https://yeasy.gitbooks.io/blockchain_guide/ethereum/install.html
* Solidity 官方文档 http://solidity.readthedocs.io/en/latest/installing-solidity.html
* Go 官方文档 https://golang.org/doc/install

> 更多区块链资料，请访问 https://github.com/gymgle/blockchain-reference


  [1]: https://github.com/xiaoping378/blog/blob/master/posts/%E4%BB%A5%E5%A4%AA%E5%9D%8A-%E7%A7%81%E6%9C%89%E9%93%BE%E6%90%AD%E5%BB%BA%E5%88%9D%E6%AD%A5%E5%AE%9E%E8%B7%B5.md
