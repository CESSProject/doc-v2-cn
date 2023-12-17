在本教程中，您将学习如何使用 ink! 在 CESS 上构建一个 NFT 交易平台的基础流程。您将编写一个ink! 智能合约，为上传到 CESS 网络的文件铸造 NFT 代币。铸造 NFT 后，用户可以选择将其 NFT 代币上架或下架。因此在开始之前，让我们先了解一下 NTF 和 CESS。如果您已经了解 NFT 和 CESS，可以直接跳到 NFT 交易平台架构的部分进行阅读。

{% hint style="warning" %}

💡 温馨提示：本教程篇幅较长。但如果您确实想了解如何创建自己的 NFT 交易平台，那么花时间阅读本教程将会为您带来一定收获。

{% endhint %}

# 什么是 NFT？

在我们了解什么是 NFT（非同质化代币）之前，我们需要先了解同质化代币。同质化代币是可以互换的，就像美元或比特币一样，它们的价值在任何地方都保持不变。例如，对于实物货币钞票，您可以将一张 10 美元钞票兑换成另一张 10 美元钞票，其价值保持不变。也就是说，同质化代币可以互换因其两个值相同。

另一方面，非同质化代币不可互换。例如，土地财产是不可替代的，因为两块土地不太可能具有相同的价值。它们可能拥有使土地具有不同的价值的因素如自然资源。

在区块链中，同质化代币被称为 ERC-20 代币，是由以太坊基金会创建的标准格式。ERC-20 代币的一些示例包括比特币 (BTC)、Tether (USDT) 等。其中非同质化代币是使用 [ERC-721](https://docs.openzeppelin.com/contracts/4.x/erc721) 代币标准创建的，该标准承认代币所有权。与此类似的是用 [ink!](https://use.ink/) 写的 [PSP34 标准](https://github.com/w3f/PSPs/blob/master/PSPs/psp-34.md)。该标准用于存储区块链中代币/收藏品的所有权，而这些收藏品可以是所有权契约、土地、艺术品、视频、图像等任何东西。

# 什么是 CESS？

CESS（Cumulus Encrypted Storage System）是基于区块链的去中心化云存储网络和 CDN 网络，支持数据在线存储和实时共享，为 Web3 高频动态数据的存储和检索提供全栈解决方案。CESS 数据价值网络是以 DePIN 理念建设的 Layer 1 基础设施，具有去中心化，高效，安全隐私和可扩展等特性。

CESS 提供了 SDK 和 RESTful API，即可以通过 CESS 网络的网关上传和下载文件，而无需担心是否对 CESS 文件分发的底层机制有深度了解。您可以点击[此处](https://docs.cess.cloud/cess-wiki/introduction/overview)了解有关 CESS 的更多信息。

# NFT 交易平台架构

![CESS NFT 交易平台](../../assets/developer/tutorials/nft-marketplace/cess-marketplace.png)

上图描述了 NFT 交易平台所涉及的所有组件的整体流程。用户端应用程序由钱包和市场/平台组成，用户可以在其中以 NFT 形式买卖资产。您还需要一个基于 PSP34 标准的智能合约来实现代币所有权确立、NFT 的购买和销售。为了本教程的简洁和重点介绍，我们讲主要关注如何在 CESS 上创建 NFT 市场的基本理念，不会涉及管理用户积分奖励和后台管理的相关内容。如需了解一个完整的 NFT 应用程序的整体流程，请继续阅读以下内容。

前端应用程序分为两部分，用户端应用程序和管理后台。 管理后台将有权从智能合约中存入或提取代币，这些代币将在用户兑换奖励积分时消耗，我们称其为国库部份。 管理后台还有能力管理用户积分奖励；另一部分，用户将从用户端应用程序获取 CESS 代币开始使用 NFT 交易平台。收到一些代币后，用户可以继续购买空间或索取通过推荐计划获得的空间。一旦用户的帐户有一些可用空间，他们就可以在 CESS 网络上上传资产（图像、视频、音频等）。如需创建 NFT，用户首先需要将其 NFT 资产上传到 CESS，并将包含 NFT 所有权元数据的交易发送到 CESS 区块链。交易确认后，他们现在可以将其 NFT 挂牌出售/租赁或赠送给任何其他用户。

用户可以通过用户端应用程序访问 NFT 市场，并查看所有列出的可供购买或租赁的 NFT。要购买 NFT，他们的钱包账户中需要有足够的代币。对于每次购买，根据应用程序的配置，用户可以获得一定数量的奖励积分。奖励积分可以用于在下次购买时进行兑换。每当用户兑换奖励积分时，剩余金额将从金库中返还给卖家。一旦所有权转移给另一个用户，关联的文件也会移动到新所有者的存储空间。

{% hint style="info" %}
温馨提示：由于本教程主要关注 NFT 市场的基本功能。篇幅有限，我们不会实现上面提到的所有功能。
{% endhint %}

# 如何创建 NFT？

创建 NFT 需要选择我们想要进行 NFT 化的资产。例如，一件图像艺术品可以被铸造为 NFT。我们将使用 CESS 网络来存储我们的资产。资产可以是视频、图像、音频等任何内容，所生成的资产的 JSON 元数据将记录在在 CESS 区块链上。

请注意，我们不会将资产数据本身存储在区块链中，而是存储其元数据。这是因为将文件本身存储在区块链上会使区块链变得臃肿，使所有节点都下载我们的资产，这不仅会增加我们的成本，而且会不必要地在所有节点上复制我们的资产。

NFT 被分配了一个唯一的 ID，我们将在铸造新的 NFT 时递增该 ID。我们将使用 `last_token_id`来跟踪最后铸造的代币 ID。

```rust
#[openbrush::storage_item]
pub struct NftData {
    pub last_token_id: u64,
    pub collection_id: u32,
    pub max_supply: u64,
    pub price_per_mint: Balance,
    pub fid_list: Mapping<Id, String>,
    pub sale_list: Mapping<Id, Balance>,
}
```

当我们将文件上传到 CESS 网络时，我们会收到一个文件 ID (fid)。由于该 fid 对于 CESS 网络来说是唯一的，因此我们可以将其用作合约的 TokenID。但是，如果我们使用 fid 作为 TokenID，openbrush 库会提示 `Decoding` 错误。因此，我们使用映射 `fid_list` 将 TokenID 映射到 fid。同样，我们还有一个映射其中包含所有在 `sale_list` 上出售的 NFT。`collection_id` 是我们 NFT 集合的一个 ID，`max_supply` 是我们可以在智能合约中铸造的 NFT 的最大数量。最后，`price_per_mint` 是每个 NFT 代币的铸造价格。金额从 `price_per_mint` 中获得并收集在智能合约中，可由合约所有者提取。

# 将钱包添加到您的应用程序

我们的应用程序需要一个钱包来帮助我们签署与存储和 NFT 相关的交易并在 CESS 网络上广播。由于 CESS 构建在 Substrate 框架之上，因此 CESS 支持 [polkadot.js](https://polkadot.js.org/) 钱包。根据您选择的前端堆栈，Polkadot 提供对多个平台的支持，例如：

- 对于 Web 应用程序，请遵循 [Polkadot.js Extension](https://polkadot.js.org/docs/extension/)
- 对于 Android 应用程序，请遵循 [Nova Substrate SDK for Android](https://github.com/novasamatech/substrate-sdk-android)
- 对于 iOS 应用程序，请遵循 [Nova Substrate SDK for iOS](https://github.com/novasamatech/substrate-sdk-ios)

钱包与应用程序集成后，请确保将其配置到 CESS 测试网络。

# 将资产上传至 CESS

CESS 为我们提供了各种 SDK 和 RESTful API 来上传我们的文件。我们将使用 CESS 测试网来创建我们的 NFT 交易市场。和在主网上创建的步骤类似。我们将把 CryptoPunk 图像作为我们的资产上传到 CESS。但在将文件上传到 CESS 之前，我们需要满足一些准备条件如为帐户注资和购买空间。

![Crypto Punk](../../assets/developer/tutorials/nft-marketplace/crypto-punk.png)

## 为您的账户注资

为您的帐户注资相当容易。访问 [CESS Testnet Faucet](https://testnet-faucet.cess.cloud/) 并输入您的帐户地址，然后按获取 TCESS。您的账户将获得 10,000 个 CESS 代币。

![CESS 测试网水龙头](../../assets/developer/tutorials/nft-marketplace/testnet-faucet.png)

## 购买空间

购买空间有两种选择。

1. 使用 SDK
2. [CESS Explorer](https://testnet.cess.cloud/)

### A. 使用 SDK 购买空间

根据您为应用程序选择的 SDK，以下步骤会有相似之处。我们将在本指南中使用 [Javascript SDK](https://github.com/CESSProject/cess-js-sdk)。

1. 安装 JavaScript SDK

    ```bash
    # npm
    npm i cess-js-sdk --save
    # yarn
    yarn add cess-js-sdk -S
    # pnpm
    pnpm add cess-js-sdk
    ```

2. 导入 SDK

    ```js
    import { InitAPI, Space } from "cess-js-sdk";
    ```

3. 初始化 API

    ```js
    const {api, keyring} = InitAPI();
    const space = new Space(api, keyring);
    ```

4. 出租空间

    ```js
    // Rent 1GB of storage space for 30 days.
    result = await space.buySpace(mnemonic, 1);
    ```

### B. 使用 CESS Explorer 购买空间

要购买空间，首先，导航到 [CESS Explorer](https://testnet.cess.cloud/) 并按照说明进行操作。

1. 导航到 Developer > Extrinsics

    ![发送交易](../../assets/developer/tutorials/nft-marketplace/01-extrinsics.png)

2. 选择适当的帐户。然后选择在 “submit the following extrinsic” 中选择 `storageHandler`，然后选择`buySpace(gibCount)`。输入您想要购买的存储空间量，然后单击提交交易。

    ![租赁空间](../../assets/developer/tutorials/nft-marketplace/02-purchase-space.png)

## 上传文件至 CESS

一旦我们的帐户中有代币并且为我们的帐户分配了足够的存储空间，我们就可以将文件上传到 CESS 网络。由于我们将使用 CESS DeOSS 网关上传文件，因此我们必须对 DeOSS 网关进行身份验证，以便网关可以代表我们发送一些与存储相关的交易并生成授权令牌。对于测试网 DeOSS，请了解以下信息：

{% hint style="info" %}
💡 DeOSS 网址：<http://deoss-pub-gateway.cess.cloud/>

DeOSS 网关账户地址：`cXhwBytXqrZLr1qM5NHJhCzEMckSTzNKw17ci2aHft6ETSQm9`
{% endhint %}

1. 执行 `oss` > `authorize(operator)` 以从测试网 explorer 外部验证 DeOSS，请使用上面给出的网关帐户地址。

    ![发送 `authorize` 交易](../../assets/developer/tutorials/nft-marketplace/authorize.png)

2. 生成授权令牌：如需生成授权令牌，请遵循本指南。大约需要 5 分钟。

现在我们准备将文件上传到 CESS。我们可以使用 REST API 或 SDK 来上传文件。

### A. 使用 SDK 上传

我们将再次使用 Javascript SDK 上传文件。初始化阶段与上一节相同。初始化后，我们可以添加以下代码片段。

```js
import { InitAPI, Space, File } from "cess-js-sdk";
const MNEMONIC = "YOUR MNEMONIC SEED";
const ACCOUNT_ID = "YOUR ACCOUNT ID";
const FILE_PATH = resolvePath(joinPath(__dirname, "/home/image.png"));
let BUCKET_NAME = "bucket1";

async function main() {
  const { api, keyring } = await InitAPI(testnetConfig);
  const oss = new File(api, keyring, gatewayURL, true);
  const result = await oss.uploadFile(MNEMONIC, ACCOUNT_ID, FILE_PATH, BUCKET_NAME);
  console.log(getDataIfOk(result), "\n");
}
```

### B. 使用 REST API 上传

现在，我们可以使用 REST API 将文件上传到 CESS 网络。

要上传文件，请执行以下命令

```bash
curl -X PUT http://deoss-pub-gateway.cess.cloud/ \
  -F 'file=@cryptopunk.png;type=image/png' \
  -H "Authorization: eyJh...IL1g" \
  -H "BucketName: my_nfts"
```

此处，`-F` 用于指定文件位置和文件类型，`-H` 设置您的 `Authorization` 令牌以及 `BucketName` 该文件的存储位置。

执行此函数将返回一个 FID，我们可以用它来访问我们的文件。

# NFT 交易市场智能合约

![ink!](../../assets/developer/tutorials/nft-marketplace/use-ink.png)

我们将使用 [ink!](https://use.ink/) 语言编写智能合约。正如上面提到的，我们的智能合约将用于创建基于 [PSP34](https://github.com/w3f/PSPs/blob/master/PSPs/psp-34.md) 标准的 NFT。在我们开始之前，有一些先决条件。请查看 [Deploy an ink! Smart Contract](https://docs.cess.cloud/core/developer/tutorials/deploy-sc-ink) 教程并安装 Rust 和`cargo-contract`.

## 开发

## TODO


# 结论

在本教程中，我们了解了 NFT 是什么，以及如何使用 CESS 来存储 NFT 文件。我们还学习了如何使用 openbrush 来实施 PSP34 代币标准，以及在 NFT 交易市场应用程序中构建和使用 ink! 合约的方式，以及使用户能够铸造和销售其 NFT 的最基本功能。

# 下一步是什么？

在下一课程中，我们将学习构建一个前端。该前端将与我们的智能合约交互，并使用户能够上传、出售和购买 NFT。

# 参考

- <https://use.ink/smart-contracts-polkadot/>
- <https://spin.atomicobject.com/2021/08/16/reentrancy-guard-smart-contracts/>
