title: "以太坊开发填坑指北"
date: 2018-03-11 12:50:00
tag:
- ethereum
- bitcoin
- blockchain
permalink: ethereum-guide

---

甩锅提醒：本文更新于 `2018.3.31`，未来这些内容肯定会过时，务必善用搜索引擎去获取知识和技能。本文内容杂多，可能需要对区块链有一些深入了解的同学才能看懂。

## 0x00 一些 Tips

1. 关于私有链搭建。直接用 puppeth。生成创世块 json 配置文件后，可以手动修改其中的参数。如果使用 PoW 共识，账本数据目录下 keystore/ 目录不需要放置挖矿账户的 keystore 文件。如果选择 PoA 共识，则要把记账账户的 keystore 放到这个目录下，因为 PoA 记账需要解锁这个账户。

2. 以太坊区块最多可以容纳多少笔交易？
  比特币把块大小限制到了 1M 或 nM。和比特币不同，在以太坊里，块中容纳的交易数由共识节点的 gasLimit 设置。当前公网的 gasLimit 可以从 https://ethstats.net 查询到，大约 8000000 左右。理论上，gasLimit 可以设置无限大，这样一个块中可以打包无限笔交易。问题讨论请参考 https://forum.ethereum.org/discussion/1757/maximum-block-size
  但是，在创世块配置信息里改 gasLimit 是无效的！需要在启动 geth 的时候用 `--targetgaslimit` 设置。

3. Ethereum Wallet 和 Mist 有什么区别？
  https://github.com/ethereum/mist/releases 这里提供了这两个软件的下载，有人会把这两者都称为钱包。区别是 Mist 是一个去中心化应用浏览器。可以用 Mist 浏览器打开任何 Ethereum Dapp 应用。  Ethereum Wallet 是 Mist 浏览器 + 以太坊钱包 Wallet Dapp 应用。
  参考：https://ethereum.stackexchange.com/questions/2690/what-is-the-relationship-between-mist-and-ethereum-wallet

4. Mist 如何连接私有链？
  ``` shell
  // 把 127.0.0.1:8545 替换成你的私有链地址
  mist --rpc http://127.0.0.1:8545 --swarmurl="http://swarm-gateways.net"
  // 不启动 rpc 的话，也可以如下直接通过 ipc 通信
  mist --rpc /path/to/geth.ipc --node-networkid your_network_id --node-datadir /path/to/ethereum/dir/
  ```

5. Ubuntu 16.04 x64 启动 mist 时缺少 libXss.so.1
  ```
  // 错误描述
  mist: error while loading shared libraries: libXss.so.1: cannot open shared object file: No such file or directory

  // 在 64 位系统中，需要安装 32 位的依赖库
  apt install libxss1:i386
  apt install libgconf2-4:i386
  apt install libasound2:i386
  ```

6. 启动 mist 提示不能启动 swarm
  错误描述：[ERROR] main - Error starting up node and/or syncing Error: Couldn't start swarm process.
  终端/命令行启动时加上 `--swarmurl="http://swarm-gateways.net"` 参数。如果在 Windows 中使用，右键 Mist 快捷方式属性，在 `快捷方式` 选项卡，`目标` 里加上上述参数。

7. MyEtherWallet 的坑。MyEtherWallet 是一个离线钱包，开源，你可以下载这个项目本地打开 index.html 文件离线运行。生成的 keystore 与标准 keystore 存在一个字母大小写的差别，它把 keystore 文件中的 `Crypto` 的首字母大写了，以至 imToken 等钱包无法导入。
修改源码 js\etherwallet-master.js 文件，把 `Crypto:` 改为 `crypto:` 即可。

8. 以太坊账户的疑问。不少人会问到这个问题：账户钱包地址都是离线创建，以太坊网络怎么知道你的地址和余额？以太坊是基于账户的设计，和比特币的 UTXO 不一样。比特币网络不保存某个账户的余额，这是由钱包或者其他中心和的服务通过账本进行了遍历和索引。以太坊网络中则保存了账户地址对应的余额。那么这个账户地址的余额是什么时候出现到了链上呢？答案是 **当有交易从发送者的账户转移价值到接收者账户时，如果接收账户还不存在，则在区块链中创建此账户。** 这时，你离线创建的账号地址才真正出现在链上。

## 0x01 以太坊钱包开发

通过 geth 客户端创建账户时，不会有助记词，是直接通过密码生成了 keystone 文件。以太坊中的助记词是从比特币钱包拿过来用的。助记词、种子、私钥的关系，可以参考《精通比特币》 http://zhibimo.com/read/wang-miao/mastering-bitcoin/Chapter04.html 中的 **确定性（种子）钱包** 章节。

1. Android 钱包开发，可以参考 https://github.com/p-acs/ethereum-offline-signer
2. iOS 钱包开发，可以参考 https://github.com/ethers-io/ethers.objc
3. Web 钱包开发，参照 MyEtherWallet https://github.com/kvhnuke/etherwallet

移动端 Android 和 iOS 开发，还可以选择 React Native，使用 web3js，毕竟 web3js 已经把 JSON-RPC 接口封装好了。

## 0x02 以太坊接口开发

Geth、Parity 对外都提供了标准的 HTTP JSON-RPC 接口。更方便的是，Node 有 web3js 库可以用，Java 和 Android 有 web3j 库可以用。

### HTTP JSON-RPC 的说明

JONS-RPC 中提供了 `eth_sendTransaction` 、 `eth_sendRawTransaction` 、 `personal_sendTransaction`。这三个都是向节点发起交易，可以用来转账、调用合约。那么，它们的的区别是？

**eth_sendTransaction**
发送未签名的交易，但是在发送这笔交易之前，需要调用 `personal_unlockAccount` 解锁交易发起方的账户。开放 `personal_unlockAccount` 容易引起安全问题，例如这样的攻击手段 https://mp.weixin.qq.com/s/Kk2lsoQ1679Gda56Ec-zJg

这种交易构造起来是这样的：
```json
{
  "jsonrpc": "2.0",
  "method": "eth_sendTransaction",
  params: [{
  "from": "0xb60e8dd61c5d32be8058bb8eb970870f07233155",
  "to": "0xd46e8dd67c5d32be8058bb8eb970870f07244567",
  "gas": "0x76c0", // 30400
  "gasPrice": "0x9184e72a000", // 10000000000000
  "value": "0x9184e72a", // 2441406250
  "data": "0xd46e8dd67c5d32be8d46e8dd67c5d32be8058bb8eb970870f072445675058bb8eb970870f072445675"
}],
  "id": "dee5ca0b-8be5-4705-94a0-cab3659df089"
}
```

**eth_sendRawTransaction**
发送签名后的交易，没有安全风险。但是需要自己把合约方法名、所有参数按照以太坊规定编码成 ABI 十六进制形式。编码方式参照 https://github.com/ethereum/wiki/wiki/Ethereum-Contract-ABI 
好消息是，以太坊提供这样的库来编码函数名和参数编码。go 使用 go-ethereum 的 github.com/ethereum/go-ethereum/accounts/abi 中提供的 Pack() 方法，JS 使用 https://github.com/ethereumjs/ethereumjs-abi 提供的 rawEncode() 方法。

这种交易构造起来是这样的：
```json
{
  "jsonrpc": "2.0",
  "method": "eth_sendTransaction",
  params: ["0xd46e8dd67c5d32be8d46e8dd67c5d32be8058bb8eb970870f072445675058bb8eb970870f072445675"],
  "id": "dee5ca0b-8be5-4705-94a0-cab3659df089"
}
```

**personal_sendTransaction**
发送未签名交易，但附带上账户的密码。

这种交易构造起来是这样的，`123456` 是账户密码：
```json
{
  "jsonrpc": "2.0",
  "method": "personal_sendTransaction",
  "params": [
    {
      "from": "0x25976d6ab66b5d99d71fce5c4b9273e66d642713",
      "to": "0x0236fB10031eCc5aF22E5bAE45fEb1Eb555DCAee",
      "vaule": "1000000000000000000"
    },
    "123456"
  ],
  "id": 1
}
```

Mist 是通过这种方式发起交易的。`personal_sendTransaction` 并不能用在高并发上，因为节点解锁账号需要消耗大量时间。

### go 开发说明

因为 go-ethereum 以太坊客户端使用 go 语言编写，因此可以直接借鉴 go-ethereum 中调用智能合约的方法来用。

**使用 abigen 工具可以把 Solidity 写的智能合约编译成 go 语言文件，然后就可以使用了。**

参照 https://github.com/ethereum/go-ethereum/wiki/Native-DApps:-Go-bindings-to-Ethereum-contracts

会用到：
```go
"github.com/ethereum/go-ethereum/ethclient"
"github.com/ethereum/go-ethereum/accounts/abi/bind"
```

ethclient 可用于连接以太坊节点，RPC 或 IPC 都能用，用法：
```go
conn, err := ethclient.Dial("http://localhost:8545/") 或
conn, err := ethclient.Dial("/home/user/.ethereum/testnet/geth.ipc")
```

然后在 abigen 生成的 `xxx.go` 合约中可以看到合约名字 `XXX`。利用合约名子、合约地址、keystore 和密码去调用合约，发起交易：
```go
contract, err := XXX(common.HexToAddress("合约地址"), conn)
# 如果只是调用合约查询，不发起交易，可以不用这一步
auth, err := bind.NewTransactor(strings.NewReader(keystore), "password")
```

利用这个 contract 和 auth 就可以调用合约了：
```
# 如果只是调用合约查询，不发起交易，不用 auth 参数
trans, err := contract.合约方法名(auth, 合约方法的参数)
```

> 注意！
本质上，go 的使用方法也是调用了 JSON-RPC。调用合约封装了 `eth_sendRawTransaction` 方法。这个方法比 `eth_sendTransaction` 复杂一些，调用之前会自行处理 nonce、gasPrice、gasLimit 等参数。nonce 这个参数使用 `eth_getTransactionCount` 方法获取（参数为账号地址和 "pending"）。这就会导致一个问题，**当一个账号刚刚提交了一个交易，该账号立刻又要发起另一笔交易时，上一笔交易还未被 "pending" ，此时 `eth_getTransactionCount` 的结果不变，导致构造新交易的 nonce 和上一笔交易 nonce 相同。会提示 "replacement transaction underpriced" 这种错误。**而使用
 JSON-RPC 的 `eth_sendTransaction` 或 `personal_sendTransaction` 不会出现这个问题。Mist 即使用的后者。

### Node 开发说明

web3js 库封装了 JSON-RPC API。只需要知道合约 ABI 和合约地址，就可以很方便地调用合约。

### Java / Android 开发说明

web3j 封装了 JSON-RPC API。合约操作也有捷径，把 Solidity 合约编译成 Java 类去使用。

* 安装 Solidity 编译工具 solc
* 下载 web3j 工具 https://github.com/web3j/web3j

```
# 把合约 test.sol 编译为 bin 和 abi 文件, 输出到当前目录下的 output 文件夹中
solc test.sol --bin --abi --optimize -o ./output
# 使用 web3j 生成 java 合约, 输出目录 ./src/main/java
./web3j solidity generate ./output/test.bin ./output/test.abi -o ./src/main/java -p com.your.organisation.name
```

## 0x03 关于

区块链参考资料 https://github.com/gymgle/blockchain-reference