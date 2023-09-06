# 准备条件

您需要准备以下内容才能将 Solidity 智能合约成功部署到 CESS。

- **MetaMask** ：需要获取以太坊地址并连接到 CESS 链
- **Remix IDE** ：开发、编译智能合约并将其部署到链上的IDE
- **访问 CESS 节点**：确保该节点允许访问 MetaMask

对于当前指南，我们将把 Solidity 合约部署到 CESS 测试链。通过以下步骤，您将了解如何在 CESS 链上部署基于 EVM 的合约。

# 将 CESS 网络添加到 MetaMask

打开 MetaMask 设置选项卡，单击 **Networks** 选项，单击 **Add a network**，然后 **Add a network manually**。

![deploy-sc-solidity-01](../../assets/developer/tutorials/deploy-sc-solidity/01.png)

在 **Add a network manually** 页面上，输入以下详细信息：

- Network Name: **CESS Testnet**
- New RPC URL：
   - <https://testnet-rpc0.cess.cloud/ws/>
   - <https://testnet-rpc1.cess.cloud/ws/>
   - <https://testnet-rpc2.cess.cloud/ws/>
- Chain ID: **11330**
- Currency Symbol: **MTCESS**

![deploy-sc-solidity-02](../../assets/developer/tutorials/deploy-sc-solidity/02.png)

{% hint style="info" %}
如果 MetaMask 无法连接您的 RPC URL，请确保 CESS 节点中的 MetaMask 允许 CORS。
{% endhint %}

# 将账户地址转换为 CESS 支持的地址

从 MetaMask 复制帐户地址。

![deploy-sc-solidity-03](../../assets/developer/tutorials/deploy-sc-solidity/03.png)

打开以下链接 [Substrate Address Converter](https://hoonsubin.github.io/evm-substrate-address-converter)。

![deploy-sc-solidity-04](../../assets/developer/tutorials/deploy-sc-solidity/04.png)

# 为账户注资

使用 [CESS Explorer](https://testnet.cess.cloud/) **账户->转账**，将部分余额转入合约地址。

![deploy-sc-solidity-05](../../assets/developer/tutorials/deploy-sc-solidity/05.png)

# 验证资金

要验证资金是否在以太坊账户中，请打开 MetaMask 并检查该账户是否已转移资金。

![deploy-sc-solidity-06](../../assets/developer/tutorials/deploy-sc-solidity/06.png)

# 使用 Remix IDE 部署合约

打开 [Remix IDE](https://remix.ethereum.org/) 并转到 **File explorer**。

在文件资源管理器中，打开您想要编译并部署的智能合约。

![deploy-sc-solidity-07](../../assets/developer/tutorials/deploy-sc-solidity/07.png)

选择文件后，转到 **Solidity Compiler** 頁，您应该看到所选的文件，按 **Compile** 按钮编译合约。编译后，您将看到绿色的打勾标记和编译后的 (\*.sol) 文件。

![deploy-sc-solidity-08](../../assets/developer/tutorials/deploy-sc-solidity/08.png)

点击 **Deploy 并运行 Transactions** ，编译成功后，您应该看到已编译的 \*.sol 文件被选中，且已经准备好部署。在 **Environments** 下拉列表中，选择 **Injected Provider - MetaMask** 并单击 **deploy**。

![deploy-sc-solidity-09](../../assets/developer/tutorials/deploy-sc-solidity/09.png)

{% hint style="info" %}
当您点击 **Deploy** 时，您可能需要在 MetaMask 中点击确认以允许 Remix 访问账户并提交交易。
{% endhint %}

单击 **Confirm** 提交交易以部署智能合约。

![deploy-sc-solidity-10](../../assets/developer/tutorials/deploy-sc-solidity/10.png)

交易在链上部署并挖矿后，您将看到以下消息。

![deploy-sc-solidity-11](../../assets/developer/tutorials/deploy-sc-solidity/11.png)

在 Remix 的 **Deployed Contracts** 部分，您可以调用智能合约的函数。

![deploy-sc-solidity-12](../../assets/developer/tutorials/deploy-sc-solidity/12.png)

# 将代币转入 CESS 账户

使用 [Substrate 地址转换器](https://hoonsubin.github.io/evm-substrate-address-converter) 将 Substrate 地址转换为以太坊账户地址。

![deploy-sc-solidity-13](../../assets/developer/tutorials/deploy-sc-solidity/13.png)

复制以太坊等效地址并使用 MetaMask 转账。

<table>
  <tr>
    <td>
      <img src="../../assets/developer/tutorials/deploy-sc-solidity/14.png" alt="deploy-sc-solidity-14"/>
      <br/>Metamask 转帐 1
    </td>
    <td>
      <img src="../../assets/developer/tutorials/deploy-sc-solidity/15.png" alt="deploy-sc-solidity-15"/>
      <br/>Metamask 转帐 2
    </td>
  </tr>
</table>

在 [CESS 浏覧器: Developer RPC calls](https://testnet.cess.cloud/#/rpc) 内确认智能合约帐户余额。使用上一步中的以太坊地址。

![deploy-sc-solidity-16](../../assets/developer/tutorials/deploy-sc-solidity/16.png)

# 将余额提取至CESS账户

要将以太坊账户中的余额提取到 CESS 账户，请按照路线 **Developer => Extrinsics => evm => withdraw**。

![deploy-sc-solidity-17](../../assets/developer/tutorials/deploy-sc-solidity/17.png)

在 CESS 浏覧器内的 **Accounts** 页内验证帐户余额。

![deploy-sc-solidity-18](../../assets/developer/tutorials/deploy-sc-solidity/18.png)
