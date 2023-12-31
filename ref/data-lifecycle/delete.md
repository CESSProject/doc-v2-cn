本节介绍了 CESS 网络中闲置数据与服役数据的删除流程。

# 服役数据删除

服役数据是指用户上传到 CESS 网络中的数据，用户对自己上传的数据拥有所有控制权，所以只有用户自己能对数据进行删除。

用户删除了自己的数据后，如果该数据存在多个拥有者，则数据内容不会从 CESS 网络中删除；如果没有其他任何拥有者，CESS 节点会将数据从硬盘上删除。

整体流程如下图所示：

![服役数据删除](../../assets/ref/data-lifecycle/delete-active-data.png)

# 闲置数据删除

闲置数据是由共识节点在 TEE 环境中生成，存储在存储节点的数据，每个闲置数据具有全网唯一性。一般有两种情况需要删除闲置数据：

- 存储节点存储了新的服役数据，需要删除对应数量的闲置数据。
- 存储节点的 `Restore` 机制检测到数据被异常修改或者验证失败，然后报告数据丢失并删除数据。

整体流程如下图所示：

![Delete Idle Data](../../assets/ref/data-lifecycle/delete-idle-data.png)
