# TEE Worker 运行指南

## 简介

TEE Worker 的主要任务是为用户在 PoDR²（Proof of Data Reduplication and Recovery）中使用的文件标记数据（生成文件标签），并为存储矿工在 PoIS（Proof of Idle Space）证明中提供的空间验证占位的闲置文件。在 TEE Worker 中完成的每项工作都是防篡改和可验证的，可以有效确保数据的真实性。

TEE Worker 基于 [Gramine 库](https://gramineproject.io/)开发，目前仅支持 [Intel 系列芯片](https://www.intel.com/content/www/us/en/developer/articles/tool/intel-trusted-execution-technology.html)。TEE Worker可分为三种类型：

1. **Marker**：专用于服役数据打标、闲置空间认证和闲置空间替换等工作；
2. **Verifier**：专用于服役数据和闲置空间的随机挑战验证，只能绑定共识节点注册；
3. **Full**：全节点，及 Marker 和 Verifier 功能的并集，只能绑定共识节点注册；

## 收益介绍

运行 TEE Worker 本身并不直接获得任何收益，而是通过执行存储网络的有效安全服务来为其相关节点获取更多的收益，TEE Worker有两种部署方式，分别可以获得不同的增益效果：

- 与共识节点绑定运行，只有在注册了共识节点的帐户签名的交易后才能工作。它需要相对较高的硬件要求，但其绑定的共识矿工也会获得更高的奖励。

- 独立注册运行，可运行特定类型的TEE Worker，无需与共识节点绑定，专门为特定存储矿工标记数据和验证空间，以从每次随机挑战中获得更高的奖励。

## 为什么部署独立注册的 TEE Worker 能为存储矿工带来更多的收益？

随机挑战时会根据存储矿工有效存储的用户数据（服役数据）和已认证的闲置空间在全网的占比来瓜分奖励。用户数据需经过 TEE Worker 打标后才能通过随机挑战，同样的，存储矿工批量生成的闲置空间也需要经过 TEE Worker 验证。但全网公开的 TEE Worker 资源有限，需要排队才能获得一次服务，对于性能较高的存储矿工，这往往是制约其生产效率的主要瓶颈之一，且由于区域网络差异的限制，分散在全球各地的存储节点获得 TEE Worker 服务的效率并不均等。因此，为消除这种差异，突破效率瓶颈，加速全网数据的验证，CESS 鼓励拥有数量较多存储矿工的用户独立注册运行若干个 TEE Worker 为自己节点服务。

## 操作指南

以 **Marker** 运行 TEE Worker 需要一个账户：即 **Controller 帐户**，以及仅需一笔用于注册交易的Gas费。请参阅[创建 CESS 账户](../community/cess-account.md)，及前往[CESS 水龙头](https://cess.cloud/faucet.html)获取 TCESS，或[联系我们](../introduction/contact.md)获取 TCESS 代币进行质押。

创建钱包帐户后，导航至[CESS 浏览器](https://polkadot.js.org/apps/?rpc=wss%3A%2F%2Ftestnet-rpc0.cess.cloud%2Fws%2F#/explorer)。运行其他类型的 TEE Worker 请参考[共识节点部署指南](../consensus-miner/running.md)。

### 安装 CESS 客户端

cess-nodeadm 是一个 CESS 节点部署和管理程序，有助于部署和管理存储节点、共识节点和全节点，以简化所有 CESS 矿工的开发运营。

```bash
wget https://github.com/CESSProject/cess-nodeadm/archive/v0.5.x.tar.gz
tar -xvf v0.5.x.tar.gz
cd cess-nodeadm-0.5.x
sudo ./install.sh
```

请检查您是否使用的是最新版本的 cess-nodeadm。目前是 v0.5.3。
如果出现 Install cess nodeadm success 消息，则表示安装成功。
如果安装失败，请查看故障排除步骤。

### 配置 CESS 客户端

运行`cess config set` 命令，参照以下方式进行配置：

```shell
cess config set
Enter cess node mode from 'authority/storage/watcher' (current: authority, press enter to skip): authority
Intel SGX is already enabled on this system
Enter cess node name (current: cess-test-node2-526824, press enter to skip): cess-test-node2
Enter cess chain ws url (current: wss://testnet-rpc0.cess.cloud/ws/, press enter to skip): ws://129.226.81.243:9944
Enter listener port for kaleido (current: 10010, press enter to skip):
Enter the kaleido endpoint (current: http://test.dm.com, press enter to skip): http://45.195.74.43:10010
Enter cess validator stash account (current: , press enter to skip): null
Your Tee worker will work as 'Marker'!
Enter cess validator controller phrase (current: level course inflict raise giant hammer blur run seed adjust ice goddess, press enter to skip):
Set configurations successfully
```

在设置 validator stash account 账户时，填入 null 后自动配置为 Marker 模式，然后再输入 controller 账户（用于 TEE Worker 注册，发送交易等功能的工作账户）助记词（账户seed）即可完成配置。然后输入 `cess start` 命令启动 Marker 型 TEE Worker。

**请注意**：上述配置中 "kaleido endpoint" 代表您部署的 TEE Worker 的访问地址，TEE Worker 运行后，您可以在运行存储矿工时将该地址加入到优先访问列表中，如使用CESS客户端配置存储矿工时，在 "Enter the reserved TEE woker endpoints" 中配置；如果您是直接手动运行的存储矿工，则请在存储矿工的配置文件中的 "TeeList:" 项中以列表形式进行配置。

## 工作原理

Marker 型 TEE Worker 工作原理如下图所示：

![Marker TEE worker workflow](../assets/storage-miner/teeworker/tee-workflow.jpeg)

TEE Worker 通过 SGX 可信执行环境保护 PoDR² 密钥，用于为用户服役文件的分片打标（多备份可恢复存储证明机制），以及验证并签名闲置空间认证或替换证明的结果。PoDR² 密钥在可信环境中生成，通过安全密钥交换通道传递到其他 TEE Worker 的可信环境中，不会泄露到外部，从而保证了算法的安全性；可信环境还对内部代码进行封装，并需要通过 Intel 远程认证，远程认证报告还需要在 TEE Worker 注册时被校验，以保证 SGX 内运行的代码是 CESS 官方公开且未被恶意篡改的，从而保障了服务的正确性。

此外，任何进入到 SGX 内部的用户请求参数，都需要进行合法性和完整性验证，以保证数据中途不会被篡改。
