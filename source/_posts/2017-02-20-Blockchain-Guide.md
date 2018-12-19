title: "区块链学习指南"
date: 2017-02-20 22:18:00
tag:
- fabric
- Hyperledger
- Blockchain
- Bitcoin
permalink: Blockchain-Guide
---

## 0x00 说明

从 2016 年开始，进行区块链技术预研并选择 Hyperledger fabric 平台实现场景的落地，在这个过程中，通过对区块链技术的学习和总结，整理出本文的学习路线。以此作为一个入门参考，希望能够帮助到对区块链技术感兴趣，并想要深入学习它的人们。

## 0x01 从比特币开始

区块链技术脱胎于比特币，因此想要深入了解区块链技术，可以先从了解比特币入手。

《[精通比特币][1]》是目前对比特币机制讲解最为详细、分析最为彻底的书。但在阅读此书之前，你可以搜一下《易懂的比特币工作机理详解》，该文通俗简短，阅读的过程中记下自己的问题，然后在《精通比特币》中找到答案。

对于[比特币白皮书][2]，并未对比特币原理做详细介绍，略读即可。

## 0x02 入门区块链技术

这个阶段，比较推荐的书是《[区块链技术指南][3]》，本书由 IBM [杨宝华][4]撰写，能对区块链有如此透彻的理解，也源于他所在的 Hyperledger 项目。同时，本书也可以作为接下来要学习的区块链平台的参考书。

## 0x03 以太坊 or Hyperledger?

接下来，选择一个区块链平台去研究。当然，这里不是单选，以太坊和 Hyperledger 都可以研究。这两者的目标都是打造一个区块链平台，造福区块链应用的开发者，让其开发更简单，无需自己实现复杂的区块链底层。但以太坊更倾向于建设共有链，Hyperledger 则是联盟链。

在 Hyperledger 项目中，fabric 项目是开发和迭代比较迅速的一个。因背后有 IBM 的技术力量，fabric 也是最被看好的一个区块链平台。我们的项目也是基于此平台开发。

fabric 有一份中文的[协议规范][5]，可以由此开始 fabric 的学习。在学习过程中，可以参考 IBM 的 [Learn Chaincode][6] 项目上手练习，这个项目也有一份几个月前我翻译的[中文版本][7]说明。

同时，IBM 也提供了一份区块链[开发人员快速入门指南][8]，作为参考。

## 0x04 平台部署

Fabric 的部署可以选择 [IBM Bluemix Blockchain][9] 服务，亦或是本地部署。前者免去了本地部署的复杂步骤，而本地部署 fabric 便于调试和测试。

在编写完成自己的链码后，可以在单节点 fabric 网络中进行调试和测试，参照《[Fabric 单节点部署与链码测试][10]》。

部署多节点的 fabric 网络适合场景的演示，参照《[Fabric 多节点部署小结][11]》。

## 0x05 区块链相关资料

最后，[这里][12]总结了区块链相关的参考资料，会不定期更新。


  [1]: http://zhibimo.com/books/wang-miao/mastering-bitcoin
  [2]: http://www.8btc.com/wiki/bitcoin-a-peer-to-peer-electronic-cash-system
  [3]: https://www.gitbook.com/book/yeasy/blockchain_guide/details
  [4]: https://yeasy.github.io/
  [5]: https://github.com/hyperledger/fabric/blob/master/docs/protocol-spec_zh.md
  [6]: https://github.com/IBM-Blockchain/learn-chaincode
  [7]: https://github.com/IBM-Blockchain/learn-chaincode/blob/master/README_zh-cn.md
  [8]: https://www.ibm.com/developerworks/cn/cloud/library/cl-ibm-blockchain-101-quick-start-guide-for-developers-bluemix-trs/index.html
  [9]: https://console.ng.bluemix.net/catalog/services/blockchain
  [10]: https://g2ex.github.io/2016/11/26/Fabric-Deployment-and-Chaincode-Setup/
  [11]: https://g2ex.github.io/2017/02/17/Fabric-Deployment-with-Multi-Peers/
  [12]: https://github.com/gymgle/blockchain-reference