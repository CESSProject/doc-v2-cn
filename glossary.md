<!--
# General

| Term                                             | Definition |
| ------------------------------------------------ | ---------- |
| Block                                            | -          |
| Blockchain                                       | -          |
| Content IDentifier (CID)                         | -          |
| Continuous Availability Proof of Storage (CAPoS) | -          |
| Data Chunk                                       | -          |
| Data Fragment                                    | -          |
| Data Segment                                     | -          |
| Epoch                                            | -          |
| Era                                              | -          |
| File ID (FID)                                    | -          |
| Hash                                             | -          |
| Merkle Root                                      | -          |
| Multi-format Data Rights Confirmation (MDRC)     | -          |
| Peer-to-peer Network                             | -          |
| Proof of Data Reduplication and Recovery (PoDR²) | -          |
| Proxy Re-encryption Technology (PReT)            | -          |
| Random Rotational Selection (R²S)                | -          |
| Reputation Rotational Consensus (R²C)            | -          |
| Slot                                             | -          |
| Smart Contract                                   | -          |
| Tag                                              | -          |
| TEE Worker                                       | -          |
| Transaction Hash                                 | -          |
| Transaction                                      | -          |
| Trusted Execution Environment (TEE)              | -          |
| WebAssembly (Wasm)                               | -          |

-->

# DeOSS

| 术语          | 定义 |
| ------------------------------------ | ---------- |
| Bucket（文件桶）    | 存储桶是用于存储对象的容器。每个私钥/账户最多可以管理1000个存储容器。 |
| Decentralized Object Storage Service（去中心化对象存储服务） | 存储服务将用户数据存储为对象，可通过CESS API访问。 |
| Object（对象） | 对象是存储数据的基本单元。与传统文件系统不同，对象没有文件目录的层级关系结构。一个对象包括数据、元数据和 fid。 |
| Data（数据）| 用户的数据包括但不限于网站和应用程序中的高清图片、音频、视频和备份文件。 |
| FID | 用于检索对象的文件哈希。服务器和用户可以通过Fid找到对象，而无需知道数据的物理地址。它是对象的全局唯一标识符（UID）。 |
| Metadata（元数据）| 元数据类似于数据标签。标签上的类型和项目数量没有限制，可以是对对象的各种描述。 |

# 存储矿工

| 术语            | 定义  |
| --------------- | ------------------------------- |
| Earning Account （收益账户）| 奖励发送到的账户   |
| Locked Space 锁定空间）   | CESS网络锁定的空间   |
| Peer ID  （节点ID）   | 用于节点通信的 ID   |
| Staking Account （质押账户） | 质押TCESS的账户，应至少有 2000 个 TCESS             |
| Staking Amount （质押数量） | 被质押的TCESS数量       |
| State  （状态）  | <p><strong>Positive（正常）</strong>：工作正常<br><strong>Frozen（冻结）</strong>：质押不足<br><strong>Exit（退出）</strong>：正在退出CESS网络<br><strong>Offline（离线）</strong>：被CESS网络踢出</p> |
| Used Space （已用空间） | 存储用户使用的空间                                                                                                                     |
| Validated Space （已验证空间）| 已验证用于存储用户数据的空间                 |

# 参考资料

* [DeOSS 词汇](https://docs.cess.cloud/deoss/introduction/terminologies)
