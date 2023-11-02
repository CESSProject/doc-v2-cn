# 开始以前

在这个部分，我们将对 CESS 区块链的源代码和架构进行中级描述。主要围绕的是 [CESSProject/cess 代码库](https://github.com/CESSProject/cess)。这里的 "中级" 意味着它比高级描述具有更多细节。阅读文档的某些部分时，建议您同时打开源代码。您就会明白不同模块是如何连接的。您不需要本页面上的知识来使用 CESS 产品，但如果您是开发人员并希望更深入地了解 CESS 区块链的工作原理，建议您阅读本部分。

# 概述

CESS 区块链建立在 Parity 开发的 Substrate 区块链 SDK 之上。因此，我们采用与基于 Substrate 的区块链类似的架构，如下图所示：

![Substrate 架构](../../assets/developer/guides/src-overview/substrate-architecture.png)

在外层，有一个节点（又称客户端）。它包含区块链的所有低级构建块，例如点对点网络、外部接口的 JSON-RPC API 以及存储和数据库系统。

在节点内部，有一个 **Runtime**，链逻辑便是发生在其中。这是识别用户帐户并存储其余额的地方。基于 Substrate 的区块链还拥有完善的治理机制，可以提出提案并通过全民投票或理事会通过。这些功能都是在链的 Runtime 中指定。

也许听起来很复杂，但 Runtime 提供的功能被进一步细分为多个模块 (Pallet)。一个（有时是几个）Pallet 一起工作来实现一组功能。由于这些 Pallet 是独立且模块化的，那就是由 Runtime 工程师决定在 Runtime 内把不同的 Pallet 配置组合起来，从而决定区块链最终提供什么功能。

这种架构使得 CESS 区块链具有模块化、可组合性和可扩展性的特性。

# 深入理解

要了解 CESS 区块链的作用，我们可以查看它的运行时并了解它由哪些 Pallet 组成。我们在这里查看的 CESS 区块链是：**[CESSProject/cess v0.7.4](https://github.com/CESSProject/cess/tree/0.7.4)**。

让我们看看它的 runtime `runtime/src/libs.rs`，特别是代码里的 [`construct_runtime!()`](https://github.com/CESSProject/cess/blob/fbb64d37fab2b1de90e96a8cac7e95d95737096d/runtime/src/lib.rs#L1506-L1570)：


```rs
construct_runtime!(
  pub enum Runtime where
    Block = Block,
    NodeBlock = opaque::Block,
    UncheckedExtrinsic = UncheckedExtrinsic
  {
    // First group - foundational pallets
    System: frame_system = 0,
    RandomnessCollectiveFlip: pallet_randomness_collective_flip = 1,
    Timestamp: pallet_timestamp = 2,
    Sudo: pallet_sudo = 3,
    Scheduler: pallet_scheduler = 4,
    Preimage: pallet_preimage = 5,
    Mmr: pallet_mmr = 6,

    // Second group - Account, balances, and transaction fees
    Indices: pallet_indices = 7,
    Balances: pallet_balances = 10,
    TransactionPayment: pallet_transaction_payment = 11,
    Assets: pallet_assets = 12,
    AssetTxPayment: pallet_asset_tx_payment = 13,

    // Third group - Managing Session, Validators, Block Production, and Block Finalization
    Authorship: pallet_authorship = 20,
    Babe: pallet_rrsc = 21,  // retain Babe alias for Polka-dot official browser
    Grandpa: pallet_grandpa = 22,
    Staking: pallet_cess_staking = 23,
    Session: pallet_session = 24,
    Historical: pallet_session_historical = 25,
    Offences: pallet_offences = 26,
    ImOnline: pallet_im_online = 27,
    AuthorityDiscovery: pallet_authority_discovery = 28,
    VoterList: pallet_bags_list = 29,
    ElectionProviderMultiPhase: pallet_election_provider_multi_phase = 30,

    // Forth group - Chain Governance
    Council: pallet_collective::<Instance1> = 40,
    TechnicalCommittee: pallet_collective::<Instance2> = 41,
    TechnicalMembership: pallet_membership::<Instance1> = 42,
    Treasury: pallet_treasury = 43,
    Bounties: pallet_bounties = 44,
    ChildBounties: pallet_child_bounties = 45,

    // Fifth group - Smart contracts capability
    Contracts: pallet_contracts = 50,
    Ethereum: pallet_ethereum = 51,
    EVM: pallet_evm = 52,
    DynamicFee: pallet_dynamic_fee = 53,
    BaseFee: pallet_base_fee = 54,

    // Sixth group - CESS specific pallets
    FileBank: pallet_file_bank = 60,
    TeeWorker: pallet_tee_worker = 61,
    Audit: pallet_audit = 62,
    Sminer: pallet_sminer = 63,
    StorageHandler: pallet_storage_handler = 64,
    SchedulerCredit: pallet_scheduler_credit = 65,
    Oss: pallet_oss = 66,
    Cacher: pallet_cacher = 67,
    CessTreasury: pallet_cess_treasury = 68,
  }
);
```

如上所示，超过 40 个模块正在集成到 Runtime 中。他们可以大致分为六组。

1. **基础模块**：这些模块提供对核心类型和交叉性实用程序的低级访问。它们充当与 Substrate 框架组件和其他 Pallet 交互的基础层。这包括跟踪链块编号、获取当前块的父哈希等。这些模块 包含 System、RandomnessCollectiveFlip、Timestamp、Sudo、Scheduler、Preimage 和 MMR 模块。

2. **帐户、余额、和费用模块**：这些模块记录着现有帐户及其在区块链中各自的余额。用户余额进一步分为[自由余额和锁定余额](https://support.polkadot.network/support/solutions/articles/65000182717-account-balances-and-locks)。它还跟踪用户的其他代币和 NFT，这些代币和 NFT 被算作 CESS 中的资产。还有处理交易费用的模块 。这些模块包括 Indices, Balances, TransactionPayment, Assets, 及 AssetTxPayment 模块。

3. **共识模块**：这些模块协同工作，记录区块链中的 sessions 以及每个 session 中相应的区块验证节点和区块生成节点，也负责记录这些验证节点的投票以最终确定区块。这些模块协同工作以确保区块数据有效，其中包括 Authorship、Babe、Grandpa、Stake、Session、Historical、Offences、ImOnline、AuthorityDiscovery、VoterList 和 ElectionProviderMultiPhase 模块。

4. **治理模块**：这些模块提供与国库合作的功能，并记录提案供理事会和公众投票决定如何分配国库资金。一些财政拨款可能是要求提交解决方案的一次性的赏金，或子赏金。这些模块包括 Council、TechnicalCommittee、TechnicalMembership、Treasury、Bounties 和 ChildBounties 模块。

5. **智能合约模块**：这些模块授予运行智能合约的链功能。CESS 链支持两种智能合约—— ink！合约 (这是由 Contract Pallet 赋能的)；以及由以太坊兼容合约 (这是由 EVM、DynamicFee Pallet 赋能的)。另外在 CESS 内还提供了一些 EVM 预编译，以允许 Substrate 链识别一些 EVM 内置函数。

6. **CESS 特定模块**：这些是由 CESS 核心开发团队构建的模块，用于实现 CESS 功能。前五个类别的大多数 Pallets 都在 Substrate 框架中提供。我们进行了进一步的定制，使其适合 CESS 区块链。然而，在本类中的 Pallet 使 CESS 区块链变得独一无二，并实现了核心的去中心化存储和缓存功能。

接下来，让我们深入了解这些 CESS 的专用模块。

# CESS 模块

## FileBank 模块

本模块由管理存储空间的逻辑组成。它允许调用者（用户）购买存储空间、扩展购买的空间、续订存储租约。该 Pallet 还实现了 CRUD（创建、读取、更新、删除）用户存储桶的功能，这一概念类似于用户目录。实际文件被分段并存储在底层存储网络中，但其元数据存储在链上。

代码实现: [`c-pallets/file-bank/src/lib.rs`](https://github.com/CESSProject/cess/blob/v0.7.4/c-pallets/file-bank/src/lib.rs)

有效调用:

- `upload_declaration()`: 供用户在上传文件之前声明文件的所有权。这将检查文件是否在链上并标记相应的元数据。
- `deal_reassign_miner()`: 将存储任务（存储交易）重新分配给另一个存储矿工。
- `ownership_transfer()`: 将文档的所有权从一个用户转移给另一个用户。除了文件之外，还需要更新一些状态，包括释放前所有者已用的存储空间并扣除新所有者的可用空间。
- `transfer_report()`: 上传链上存储文件的元数据。如果文件的元数据以前存在，则会更新。
- `calculate_end()`: 结束某个指定哈希的存储任务。已使用的存储空间被释放，交易信息被清理。
- `replace_idle_space()`: 存储矿工中的特定空闲空间被用户内容替换。
- `delete_file()`: 删除所有者拥有的文件。调用者可以是所有者本人，也可以被授予所有者管理文件的权限。
- `cert_idle_space()`: 最多上传10个空闲文件，用于证明存储矿机空间。
- `create_bucket()`: 创建存储桶。
- `delete_bucket()`: 删除存储桶。
- `generate_restoral_order()`: 生成恢复某些内容的命令。它存储在队列中以供存储矿工拾取。
- `claim_restoral_order()`: 存储矿工前来领取恢复订单。
- `claim_restoral_noexist_order()`: 存储矿工来生成新的恢复订单并领取。它是以上两个功能的组合。
- `restoral_order_complete()`: 存储矿工声称恢复订单已完成。
- `root_clear_failed_count()`: 清除所有存储矿机的失败次数。只能由 `root` 用户调用。
- `miner_clear_failed_count()`: 清除呼叫失败次数。

## TeeWorker 模块

本模块提供的功能主要与共识矿工相关，包括注册节点成为共识矿工、更新内部记录、退出共识矿工等。

代码实现: [`c-pallets/tee-worker/src/lib.rs`](https://github.com/CESSProject/cess/blob/v0.7.4/c-pallets/tee-worker/src/lib.rs)

有效调用:

- `register()`: 调用者注册成为共识矿工。它可以选择提供一个用于管理其权益的存储账户、一个对等 ID、一个 PoDR2 公钥以及一份 SGX 证明报告，以证明该节点具有 SGX 功能。
- `update_whitelist()`: 更新矿工 enclaves 的白名单存储。只能由 `root` 用户调用。
- `exit()`: 调用者退出共识矿工。

## Audit 模块

本模块管理多备份可恢复存储证明 (PoDR²)。它包括生成和验证矿工服务文件和空闲文件证明的逻辑。它还为存储矿工带来随机挑战。它与其他模块配合使用，应对存储挑战并在需要时对矿工进行惩罚。

代码实现: [`c-pallets/audit/src/lib.rs`](https://github.com/CESSProject/cess/blob/v0.7.4/c-pallets/audit/src/lib.rs)

有效调用:

- `save_challenge_info()`：生成新的挑战请求并将其保存到链上。它可以是服役空间挑战，也可以是闲置空间挑战。
- `submit_idle_proof()`: 存储矿工根据之前生成的质询请求，在链上提交闲置空间证明。
- `submit_service_proof()`: 存储矿工根据之前生成的挑战请求，在链上提交服役空间证明。
- `submit_verify_idle_result()`: 对某个指定挑战请求和闲置空间证明，验证者检查空闲证明是否有效并满足挑战请求。
- `submit_verify_service_result()`: 对某个指定挑战请求和服役空间证明，验证者检查服务证明是否有效并满足挑战请求。

## Sminer 模块

本模块包含与存储矿工相关的操作，允许他们声明其提供多少空间、持续多长时间、为其声明的服务抵押其代币，以及完全撤回服务提供。

代码实现: [`c-pallets/sminer/src/lib.rs`](https://github.com/cessProject/cess/blob/v0.7.4/c-pallets/sminer/src/lib.rs)

有效调用:

- `regnstk()`: 负责注册新的存储矿工并为其服务添加权益。
- `increase_collateral()`: 增加存储矿工的股份（又称抵押品）。
- `update_beneficiary()`: 更新存储矿机的受益人账户。
- `update_peer_id()`: 更新存储矿机的 Peer ID。
- `receive_reward()`: 存储矿工调用来获取奖励。
- `miner_exit_prep()`: 存储矿机请求将来退出其服务。
- `miner_exit()`: 存储矿机退出服务。
- `mine_withdraw()`: 存储矿工请求提现。发出恢复命令，以便将矿机的存储内容迁移到其他矿机。
- `faucet_top_up()`: 调用者将其持有的部分代币返还至水龙头。
- `faucet()`: 调用者向水龙头索要一些代币。每天只能调用此函数一次。

## StorageHandler 模块

本模块管理并记录底层存储矿工提供的总存储空间。它允许用户租赁、扩展空间和更新定价。

代码实现: [`c-pallets/storage-handler/src/lib.rs`](https://github.com/cessProject/cess/blob/v0.7.4/c-pallets/storage-handler/src/lib.rs)

有效调用:

- `buy_space()`: 调用者请求购买空间。
- `expansion_space()`: 调用者请求购买额外空间。他之前得调用过 `buy_space()`。请求的空间必须大于他现有已购买的空间。
- `renewal_space()`: 将当前存储空间的租期延长给定的天数。
- `update_price()`: 按指定价格更新存储空间的基本价格。只能由 `root` 用户调用。
- `update_user_life()`: 按给定时间段更新用户存储寿命。只能由 `root` 用户调用。

## SchedulerCredit 模块

本模块管理共识矿工（又名调度人）的声誉以维护链状态。这包括处理文件的记录和未能实现调度目标的处罚记录。最后根据设定的算法给共识矿工打分。该分数将作为 R2S 共识的一部分决定下一轮的验证者。

代码实现: [`c-pallets/scheduler-credit/src/lib.rs`](https://github.com/cessProject/cess/blob/v0.7.4/c-pallets/scheduler-credit/src/lib.rs)

无有效调用，都是只供内部使用的支援函数。

## Oss 模块

本模块记录了 CESS 对象存储服务的信息。它允许注册、更新、和销毁对象存储服务。

代码实现: [`c-pallets/oss/src/lib.rs`](https://github.com/cessProject/cess/blob/v0.7.4/c-pallets/oss/src/lib.rs)

有效调用：

- `authorize()`: 为调用者设置指定的委托人。这可以理解为调用者将自己的部分权限委托给了委托人。
- `cancel_authorize()`: 取消对委托人的授权，取消上面指定的委托人代表调用者行事的权利。
- `register()`: 使用特定的 Peer ID 将调用者节点注册为新的 OSS 服务提供者端点。
- `update()`: 使用新的 Peer ID 更新调用者节点。
- `destroy()`: 取消调用者节点提供OSS服务。

## Cacher 模块

本模块为缓存矿工提供功能。缓存矿工可以检索交易记录中的账单并提供文件下载服务。

代码实现: [`c-pallets/cacher/src/lib.rs`](https://github.com/cessProject/cess/blob/v0.7.4/c-pallets/cacher/src/lib.rs)

有效调用:

- `register()`: 为调用者节点注册为缓存提供者。
- `update()`: 更新调用者的缓存信息。调用者必须事先注册为缓存提供者。
- `logout()`: 调用者正在退出缓存提供者身份。
- `pay()`: 向所有缓存供应商付款。
