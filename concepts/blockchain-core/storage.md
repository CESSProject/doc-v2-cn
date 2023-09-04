本节介绍交易是如何被存储的以及 CESS 系统中与业务相关的元信息的存储方式。

在 Substrate 框架中，使用键值对来存储数据，这是数据库支持和修改的基本实现。Substrate 中所有高级存储抽象都基于这种简单的键值对实现。

# 键-值 数据库

Substrate 的存储数据库基于 RocksDB 实现。RocksDB 是一种适用于快速存储场景的持久化键值存储数据库，并支持 Parity 数据库（目前为测试版本）。

该数据库用于所有需要持久化存储的 Substrate 组件：

- Substrate 客户端
- Substrate 轻节点
- Off-chain worker

# CESS 使用的数据存储

CESS 网络使用 Substrate 提供的更高级的数据库抽象。

[**`StorageValue` 类型**](https://paritytech.github.io/substrate/master/frame_support/storage/types/struct.StorageValue.html) 用于存储任意单个值。以下是常见的用例：

- 元信息的单个值
- 单个结构体对象
- 单个相关项的集合

如果使用列表来存储项目，需要注意列表的长度。非常大的列表或结构会产生存储成本。在运行时处理大型列表或结构可能会影响网络性能，甚至导致区块停止生成。CESS 网络使用此类型来存储一些 TEE Worker 的公共证书或公钥。有关具体用法，请参阅[此页](https://paritytech.github.io/substrate/master/frame_support/storage/trait.StorageValue.html#required-methods)。

[**`StorageMap` 類型**](https://paritytech.github.io/substrate/master/frame_support/storage/types/struct.StorageMap.html) 存储了单个键对应的值。映射存储适用于需要随机访问存储项而不是整体顺序迭代的情况，例如特定账户的余额值。它用于存储 CESS 中节点和文件的元信息。

[**`StorageDoubleMap` 类型**](https://paritytech.github.io/substrate/master/frame_support/storage/types/struct.StorageDoubleMap.html) 存储了两个键对应的值。此存储的一个优势是可以有效地删除所有与第一个键相关的项。在 CESS 中，它用于存储位于存储节点中空闲文件的元信息。

[**`StorageNMap` 类型**](https://paritytech.github.io/substrate/master/frame_support/storage/types/struct.StorageNMap.html) 存储了任意键对应的值。目前，CESS 不使用此类型的存储。

对于映射类型的存储，可以根据存储数据的敏感性自定义映射算法。其提供了灵活的选择：选择计算速度较快但安全性较低的算法，或者计算速度较慢但安全性较高的算法。

# 存储查询

基于使用 Substrate 构建的区块链，任何节点都可以暴露一个远程过程调用（RPC）服务器。通过 RPC 访问存储对象时，只需提供与对象相关的主 Key 即可。在运行时逻辑中，开发者需要对存储信息进行严格的判断，并高效地使用它。

# 交易存储

Runtime 涉及了底层的键值数据库和内存中的存储覆盖层。它们将跟踪数据的状态和变化，直到值被提交到底层数据库。默认情况下，在运行时函数将值提交给主存储覆盖层之前，变更的状态将单独写入内存中的交易存储层。如果由于错误导致交易无法完成，则交易存储层中的变更将被丢弃。

您可以通过声明 [`#[transactional]`](https://paritytech.github.io/substrate/master/frame_support/attr.transactional.html) 宏来添加交易存储层。当然，这会消耗更多的计算资源。但出于安全原因，CESS 为每个交易明确声明并添加了交易存储层。接下来的 Substrate 版本中，将默认添加交易存储层。
