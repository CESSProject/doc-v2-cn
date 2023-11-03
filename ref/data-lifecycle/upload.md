# 整体流程

![数据上传流程](../../assets/ref/data-lifecycle/upload-process.svg)

本节介绍了数据上传到 CESS 网络中的整体流程。

当您将数据上传到 CESS 时，它们会被分成固定大小的分片，然后对分片使用冗余编码技术 (基于 [Reed-Solomon 错误更正法](https://en.wikipedia.org/wiki/Reed%E2%80%93Solomon_error_correction)) 得到大小固定的 fragment，最后将数据分片分发到网络中的存储节点。

# 概述

要将数据上传到 CESS 网络，您必须执行以下步骤：

1. 租贷存储空间
2. 授权空间使用权予 [**DeOSS**](https://docs.cess.cloud/deoss)
3. 如需要，创建文件桶
4. 上传数据到 DeOSS
5. 根据返回的 Hash 值可以查看数据状态或者下载数据

具体的操作步骤参考 [DeOSS 使用指南](https://docs.cess.cloud/deoss).

# 上传流程

下图展示了用户上传数据的详细处理流程，其中 metadata 指的是元数据，包含了客户端计算好的数据大小、数据 hash、分段列表、冗余列表以及拥有者的账户、数据所在桶信息等，客户端在链上下达存储订单，链网络分配存储节点，客户端通过监听事件获取到链网络处理的结果，里面包含了每个分片对应的存储节点，随后客户端需要把所有的分片存储到这些存储节点中：

![数据上传流程](../../assets/ref/data-lifecycle/data-upload-flow.svg)

用户在上传数据时可以选择是否对数据加密，如果需要加密，还需用户输入加密密钥，SDK 使用用户输入的密钥进行加密，SDK 不会记录用户密钥，用户要妥善保管密钥，一旦丢失 CESS 网络也无法解密其数据。

用户收到数据的 hash 后，数据尚未存储到 CESS 网络中的存储节点上面，当数据的状态变为 ACTIVE 后，表示数据被成功存储。
