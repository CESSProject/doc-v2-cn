参考下面的CESS总体代币经济学：

![CESS 代币经济](../assets/storage-miner/reward/tokenomics-v1.png)

CESS 网络将发行 **总计100 亿个代币**，其中 **30% 分配给存储矿工**，**15% 分配给共识矿工**。

在第一年，共发行 **1.875 亿个代币**，并在整年中每个 era 中均匀分发。总奖励每年以阶梯式减少，年衰减率为 0.841（0.5<sup>0.25</sup>），即每四年奖励减半。

# 奖励

对于存储节点，只有在存储矿工完成随机审计挑战后才会获得奖励。奖励金额取决于该挑战轮次的奖池和存储矿工在网络中的存储能力比例。奖励分发后，存储节点必须发起交易来领取奖励。

奖池来源于网络在每个时代生成的固定数量的币。在最初的四年里，平均每个区块发行71个CESS代币，这个发行量每四年减半一次。

存储矿工在 *第 k 轮* 的奖励根据以下因素确定：

- **TotalReward** ：*第 k 轮* CESS 网络的总奖励。
- **ServiceSpace** ：*第 k 轮* 存储矿工已使用的空间。
- **IdleSpace** ：*第 k 轮* 存储矿机的闲置空间。
- **TotalStoragePower** ：*第 k 轮* 中所有存储矿工算力的总和。

$$\boxed{StoragePower_k = IdleSpace_k * 0.3 + ServiceSpace_k * 0.7}$$

$$\boxed{RewardOrder_k = TotalReward_k * \cfrac{StoragePower_k}{TotalStoragePower_k}}$$

`RewardOrder` 是该轮存储矿工的奖励。一旦奖励确定，立即分配奖励订单的 20%，其余的在随后的 180 轮中分配，每次分配剩余金额的 1/180。

*第 k 轮* 存储矿工的可用奖励计算如下：

$$\boxed{AvailableReward_k = (RewardOrder_k * 20\%) + \sum_{t=k-180}^{k-1} \cfrac{RewardOrder_t}{180}}$$

奖励订单将在全部分配完毕后删除。因此，存储矿工最多可以从 181 个订单中获得奖励。

一旦计算出矿工的奖励，它就会存储在聚合池中。矿工需要发送交易才能获取奖励。他们可以选择等待并在之后一次性领取奖励，以节省交易费用。

# 惩罚机制

存储矿工的惩罚会惩罚其当前的质押金额，惩罚的金额与惩罚种类以及当前存储矿工的闲置服役总空间有关，惩罚有两种，一种是正常的证明惩罚 (proving slashes)，一种是较为严重的清算惩罚 (clearance slashes)。

## 证明惩罚 (Proving Slashes)

存储矿工将收到两次挑战。如果矿工无法通过 TEE Worker 的验证，它将受到 证明惩罚。

惩罚算法：

$$\boxed{StorageSpace = IdleSpace + ServiceSpace}$$

$$\boxed{SlashLimit = 4000\text{ \small{CESS/TB}} * StorageSpace\text{ \small{(向上舍入最接近的 TB 数)}}}$$

这意味着即使存储空间小于 1TB，也将以 1TB 计算。如果闲置空间连续两次未能通过验证，则惩罚金额为 **惩罚上限 * 25%**。

## 申报惩罚 (Clearance Slashes)

如果矿工无法完成存储挑战，它将被惩罚。存储挑战失败一次，扣除 **惩罚上限 * 30%**；连续两次失败，**惩罚上限 * 50%**；连续第三次失败，扣除 **惩罚上限 * 100%**，返还剩余质押金额，并将该节点从存储节点集中删除。
