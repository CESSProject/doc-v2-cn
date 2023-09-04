CESS 网络主要包括两种节点类型：**共识节点** 和 **存储节点**。

# 共识节点

共识节点对于 CESS 区块链中的验证者选举和创作区块至关重要。所有共识节点都具有以下特点：

- 记录并存储交易结果和状态变化
- 以去中心化的方式在形成点对点网络的节点之间进行通信
- 执行共识算法，确保链上数据安全和持续增长
- 包含用于签名和交易验证的加密算法

共识节点采用 [Substrate 框架](https://substrate.io/)开发。

## CESS 节点

CESS 节点是 CESS 网络的核心组件。是基于 Substrate 框架开发的区块链节点程序。

{% hint style="info" %}
如果同时运行 TEE Worker，则运行在全节点模式的 CESS 节点将被选举为验证者，负责生产和验证区块。
{% endhint %}

## **TEE Worker**

TEE Worker 的主要任务是标记数据并生成文件以验证存储节点上的存储。[TEE](https://en.wikipedia.org/wiki/Trusted_execution_environment) 中完成的每一项作业都是防篡改、可验证的，可以有效保证数据的真实性。

TEE Worker 基于 [Gramine 库](https://gramineproject.io/)开发，目前仅支持 [Intel 系列芯片](https://www.intel.com/content/www/us/en/developer/articles/tool/intel-trusted-execution-technology.html)。

TEE Worker 绑定共识节点，只有用共识节点的账户签名注册交易后才能工作。对硬件性能要求较高，需要 TEE 功能的硬件支持，需要强大的 CPU 才能快速完成打标任务。因此，这虽然成本较高，但也为矿工赚取了更多奖励。

如果您有兴趣运行共识节点，请参阅 [**生态角色：共识矿工**](https://www.notion.so/o/FsIgz09ubmGQUNERT1yp/s/Amt4KS3sOQ22dgD6uOuE/consensus-node)。

# 存储节点

存储节点在 CESS 网络的分布式存储系统中发挥着另一个至关重要的作用。所有节点都是对等体，采用基于 P2P 通信技术形成全球分布式存储网络[libp2p](https://github.com/libp2p/go-libp2p) 执行。

存储节点负责提供存储空间、存储数据、提供下载、计算数据证明。他们还控制使用哪个磁盘以及为 CESS 网络提供服务的最大存储容量。提供的存储量越大，获得的奖励就越高。

CESS 网络激励存储节点提供 4TB 及以上磁盘容量，最好采用 SSD，网络带宽不低于 2Mbps。满足这一要求的存储节点可以加速产生更高的奖励。

## 链客户端组件

链客户端组件基于开源的 [go-substrate-rpc-client](https://github.com/centrifuge/go-substrate-rpc-client) 实现，指定了存储节点如何与区块链节点交互，并提供查看链状态、交易、监听事件等功能。

## 数据库组件

数据库组件采用基于开源 [**goleveldb**](https://github.com/syndtr/goleveldb) 实现的高性能 [LevelDB](https://en.wikipedia.org/wiki/LevelDB)。用于提供链上元数据缓存并加速读取。

## P2P 通信组件

P2P 通信组件基于 [**libp2p**](https://libp2p.io/) 开发，实现存储节点之间形成 P2P 网络。有关详细信息，请参阅 GitHub 存储库 [CESSProject/p2p-go](https://github.com/CESSProject/p2p-go)。

如果您有兴趣运行存储节点，请参阅[**生态角色：存储矿工**](https://www.notion.so/o/FsIgz09ubmGQUNERT1yp/s/Amt4KS3sOQ22dgD6uOuE/storage-node) 部分。
