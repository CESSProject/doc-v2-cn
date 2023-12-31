上传到 CESS 网络的用户数据将经历以下生命周期，我们将在本章节详细介绍每一个流程和环节。

* [**数据上传**](upload.md): 用户通过 CESS 客户端或网关将数据上传到购买的存储空间。CESS 使用纠删码将数据存储在多个存储节点上，具有 1.5 倍的冗余度。

* [**数据下载**](download.md): CESS 数据可供下载。文件元信息保存在链上。用户可以使用客户端或网关在本地检索数据。

* [**数据删除**](delete.md): CESS 节点根据其所有者的请求删除数据，并智能地处理用闲置数据替换服役数据。

* [**数据恢复**](restore.md): CESS 通过存储挑战机制确保用户数据的完整性。当发现数据块丢失时，CESS 立即修复该数据块。

* [**数据审计**](audit.md): CESS 开发了一系列审计算法和奖励机制，以确保存储在存储节点中的数据得到诚实、客观的审计服务。
