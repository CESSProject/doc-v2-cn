# 服务器要求

运行共识节点的服务器的推荐要求：

| 资源 | 规格 |
| --- | --- |
| 推荐操作系统 | Ubuntu_x64 20.04 或更高版本 |
| CPU 核心数 | ≥4 |
| 启用英特尔 SGX | 必需 |
| 内存（SGX加密内存） | ≥64GB |
| 带宽 | ≥5Mbps |
| 公网 IP | 必需 |
| Linux 内核版本 | 5.11 或更高版本 |

{% hint style="info" %}
### SGX 支持

CPU 必须支持 [Intel Software Guard Extensions](https://www.intel.com/content/www/us/en/architecture-and-technology/software-guard-extensions.html) 及 Flexible Launch Control (FLC)。BIOS 必须支持 Intel SGX，并且必须启用 Intel SGX 选项。请参阅服务器制造商的 BIOS 指南以启用 SGX 功能。这里查看 [支持 SGX 的 CPU 型号](https://ark.intel.com/content/www/us/en/ark/search/featurefilter.html?productType=873&2_SoftwareGuardExtensions=Yes)，包括 _Intel ME_，_Intel SPS_，或者 _同时是 Intel ME 和 Intel SPS_。

### 固网 IP

机器必须使用固定的公网 IP。流量出口必须与 IP 在同一网段。执行以下命令确认它们在同一网段。

```bash
curl ifconfig.co
```
{% endhint %}

# 准备 CESS 账户

运行存储验证器需要两个帐户。

- **Stash 帐户**: 需要至少从节点所有者或其他用户委托质押 300,000 TCESS 才能运行共识验证器。
- **Controller 帐户**: 需要至少 100 TCESS 来支付 Gas 费。

请参阅 [创建 CESS 账户](../community/cess-account.md)，及前往 [CESS 水龙头](https://cess.cloud/faucet.html)获取 TCESS，或[联系我们](../introduction/contact.md)获取 TCESS 代币进行质押。

创建钱包帐户后，导航至[CESS 浏览器](https://testnet.cess.cloud/)。

## 为 Stash 帐户绑定资金

选择 **Network** ，点击 **Staking** > **Accounts** > **Stash**

![新增 Stash 帐户](../assets/consensus-miner/running/acct-prep-01.webp)

选择 **Stash Account** 和 **Controller Account** 。

Value bonded：建议 300,000 TCESS。在 _payment destination_ 选择第二个选项 **Stash Account as the reward receiving account (do not increase the amount at stake)**，即挖矿收入不会自动添加到质押中。

![绑定资金](../assets/consensus-miner/running/acct-prep-02.webp)

点击 **Bond** -> **Sign and Submit** 以连接 Stash Account 和 Controller Account 。

![签名提交](../assets/consensus-miner/running/acct-prep-03.png)

资金绑定成功

![资金绑定成功](../assets/consensus-miner/running/acct-prep-04.png)

# 安装 CESS 客户端

`cess-nodeadm` 是一个 CESS 节点部署和管理程序，有助于部署和管理存储节点、共识节点和全节点，以简化所有 CESS 矿工的开发运营。

```bash
wget https://github.com/CESSProject/cess-nodeadm/archive/v0.4.4.tar.gz
tar -xvf v0.4.4.tar.gz
cd cess-nodeadm-0.4.4
sudo ./install.sh
```

{% hint style="info" %}
请检查您是否使用的是[最新版本的 `cess-nodeadm`](https://github.com/CESSProject/cess-nodeadm/tags)。目前是 **v0.4.4**。
{% endhint %}

如果出现 `Install cess nodeadm success` 消息，则表示安装成功。

如果安装失败，请查看[故障排除步骤](../storage-miner/troubleshooting.md)。

# 配置CESS客户端

运行:

```bash
cess config set
```

你应该看到类似下面的输出:

```bash
Enter cess node mode from 'authority/storage/watcher' (current: authority, press enter to skip):
Enter cess node name (current: cess, press enter to skip):
Enter external ip for the machine (current: xxx.., press enter to skip):
Enter cess chain ws url (current:ws://172.18.0.9:9944, press enter to skip):
Enter cess scheduler stash account (current: xxx.., press enter to skip):
Enter cess scheduler controller phrase (current: xxx.., press enter to skip):
Set configurations successfully

Intel SGX is already enabled on this system
Start generate configurations and docker compose file
debug: Loading config file: config.yaml
info: Generating configurations done
info: Generating docker compose file done
57ea8c914461c3184ddb......
Configurations generated at: /opt/cess/nodeadm/build
try pull images, node mode: authority
download image: cesslab/cess-chain:latest
latest: Pulling from cesslab/cess-chain
3b65ec22a9e9: Already exists
6e4a9a61f489: Pull complete
Digest: sha256:5652dce2bd28796......
Status: Downloaded newer image for cesslab/cess-chain:latest
docker.io/cesslab/cess-chain:latest
download image: cesslab/kaleido:latest
latest: Pulling from cesslab/kaleido
Digest: sha256:49899acaafd11982......
Status: Image is up to date for cesslab/kaleido:latest
docker.io/cesslab/kaleido:latest
download image: cesslab/kaleido-rotator:latest
latest: Pulling from cesslab/kaleido-rotator
Digest: sha256:42535077af6bf......
Status: Image is up to date for cesslab/kaleido-rotator:latest
docker.io/cesslab/kaleido-rotator:latest
download image: cesslab/kaleido-kafka:latest
latest: Pulling from cesslab/kaleido-kafka
Digest: sha256:29ed986ea......
Status: Image is up to date for cesslab/kaleido-kafka:latest
docker.io/cesslab/kaleido-kafka:latest
pull images finished
```
如果配置过程失败，请参考[故障排除指南](../storage-miner/troubleshooting.md)。

# 管理验证节点生命周期

## 成为验证者

1. 启动共识节点<br/>

    ```bash
    cess start
    ```
2. 生成 session 密钥<br/>

    ```bash
    cess tools rotate-keys
    ```

    ![rotate-keys 输出参考](../assets/consensus-miner/running/rotate-keys.png)

    “result” 后面引号内的字段是 session 密钥 (Session Key)，后续操作时会用到。“localhost:9933” 是默认端口。<br/>

3. 设置 session 密钥<br/>

    导航到 [CESS 浏览器](https://testnet.cess.cloud), 选择 **Network** > **Staking** > **Accounts** > **Session Key**<br/>

    ![Session Key 01](../assets/consensus-miner/running/session-key-01.png)

    红框中填写 **Session Key**<br/>

    ![Session Key 02](../assets/consensus-miner/running/session-key-02.png)

    点击 **Sign and Submit**<br/>

    ![Session Key 03](../assets/consensus-miner/running/session-key-03.png)

4. 成为验证者<br/>

    导航到 [CESS 浏览器](https://testnet.cess.cloud), 选择 **Network** > **Staking** > **Accounts** > **Validate**<br/>

    ![Validator 01](../assets/consensus-miner/running/validator-01.webp)

    ![Validator 02](../assets/consensus-miner/running/validator-02.png)

    在 _reward commission percentage_ 输入 **100**, 即表示奖励不会分配给其他人。<br/>

    在 _allows new nominations_ 下拉例表中选择 **No, block all nominations** 表示不会接受任何提名。<br/>

    再次单击 **Sign and Submit**.<br/>

    ![Validator 03](../assets/consensus-miner/running/validator-03.png)

    完成上述步骤后，打开 [CESS 浏览器](https://testnet.cess.cloud/)，点击 **Network** > **Staking** > **Waiting**.<br/>

    ![Validator 04](../assets/consensus-miner/running/validator-04.webp)

    此时您将看到该节点已经出现在候选节点列表中。

## 兑换奖励

导航至 CESS 浏览器: **Network** > **Staking** > **Payouts** > **Payout**.

![兑换: 第一步](../assets/consensus-miner/running/redemption-01.png)

在 **Payouts** 页，单击 Payout 以发起付款。任何帐户都可以发起付款。

![兑换: 第二步](../assets/consensus-miner/running/redemption-02.png)

{% hint style="info" %}
请在 84 era（测试网每个 era 24 小时）内领取奖励，即 84 天。在此期间未领取奖励的，将无法领取。
{% endhint %}

## 退出共识验证

1. 停止共识节点<br/>

    在 [CESS 浏览器](https://testnet.cess.cloud), 导航至：**Network > Staking > Account Actions > Stop**.<br/>

    ![Exiting-01](../assets/consensus-miner/running/exiting-01.png)

2. 清除 session 密钥<br/>

    在 [CESS 浏览器](https://testnet.cess.cloud), 导航至：**Developer -> Submission**<br/>

    ![Exiting-02](../assets/consensus-miner/running/exiting-02.png)

    在 _using the selected account controller_ 内选定要用的控制帐户，然后在 _submit the following extrinsic_，输入 **session** 并在旁边框中选 **purgeKeys()**。<br/>

    ![Exiting-03](../assets/consensus-miner/running/exiting-03.png)

    点击 **Submit Transaction** 来清除 session 密钥<br/>

    ![Exiting-04](../assets/consensus-miner/running/exiting-04.png)

## 赎回质押

1. 解绑资金<br/>

    28 个 eras 后（测试网每个 eras 为 24 小时），执行以下操作：在 [CESS 浏览器](https://testnet.cess.cloud/), 导航至：**Network > Staking > Account Actions > Unbond Funds**.<br/>

    ![Staking 01](../assets/consensus-miner/running/staking-01.png)

2. 停止 CESS 客户端<br/>

    ```bash
    cess stop
    ```

# 常用操作

## 启动共识节点

```bash
cess start
```

## 查询矿机状态

```bash
$ cess status

-----------------------------------------
 NAMES           STATUS
kld-agent       Up 2 minutes
kld-sgx         Up 2 minutes
kaleido-kafka   Up 2 minutes
chain           Up 2 minutes
watchtower      Up 2 minutes
-----------------------------------------
```

## 检查配置信息

```bash
cess config show
```

## 停止并删除所有服务

```bash
cess down
```

# 升级 CESS 客户端

## 停止并删除所有服务

```bash
cess stop
cess down
```

## 删除所有链数据

{% hint style="warning" %}
除非 CESS 网络已重新部署且确认数据可以清除，否则请勿执行此操作。
{% endhint %}

```bash
cess purge
```

## 更新 `cess-nodeadm`

```bash
wget https://github.com/CESSProject/cess-nodeadm/archive/<new-version>.tar.gz
tar -xvf <new-version>.tar.gz
cd cess-nodeadm-<new-version>
./install.sh --skip-dep
```

目前 [最新版本](https://github.com/CESSProject/cess-nodeadm/tags) 是 **v0.4.4**.

## 拉取镜像

```bash
cess pullimg
```
