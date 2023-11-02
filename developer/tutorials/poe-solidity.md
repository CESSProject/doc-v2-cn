![Solidity](../../assets/developer/tutorials/poe-solidity/solidity.png)

# 目标

在本教程中，我们将逐步介绍一个用 Solidity 编写智能合约、使用 React 编写前端的 dApp 的开发全过程。同样，我们将构建一个存在证明（PoE）dApp。对于那些不熟悉区块链上的存在证明是什么或尚未完成之前的教程的人。请看一下 [Proof of Existence 的背景介绍](./poe-ink.md#objective)。

{% hint style="success" %}

本教程的完整源代码可以在我们的 [`cess-course` 代码库](https://github.com/CESSProject/cess-course/tree/main/examples) 查看.

- [智能合约](https://github.com/CESSProject/cess-course/blob/main/examples/hardhat/contracts/ProofOfExistence.sol)
- [前端](https://github.com/CESSProject/cess-course/blob/main/examples/frontend/src/ProofOfExistenceSolidity.js)

{% endhint %}

我们先看智能合约（Solidity）端，然后再看前端。

# 智能合约（Solidity）

## 准备条件

我们将使用 [**Hardhat**](https://hardhat.org/) 作为智能合约开发工具链。首先安装并初始化 hardhat 库。

```bash
mkdir hardhat
cd hardhat
pnpm dlx hardhat init
```

您需要回答一系列的问题来设置项目。

```
✔ What do you want to do? · Create a TypeScript project
✔ Hardhat project root: · (your chosen dir)
✔ Do you want to add a .gitignore? (Y/n) · y
✔ Do you want to install this sample project's dependencies with npm (hardhat @nomicfoundation/hardhat-toolbox)? (Y/n) · y
```

![Hardhat 项目的提示](../../assets/developer/tutorials/poe-solidity/hardhat-prompting.png)

Hardhat 默认使用 `npm` package 管理器，但请随意使用您喜欢的 package 管理器。

我们还将使用[**`hardhat-deploy`**](https://github.com/wighawag/hardhat-deploy)库来帮忙部署和管理已部署的智能合约。

```bash
pnpm install -D hardhat-deploy
```

初始情况下，Hardhat 放置了一智能合约 **Lock.sol** 在 `hardhat/contracts` 目录中。通过运行`pnpm hardhat test` 检查一切是否正常，并查看所有测试用例是否通过。

If you have any issues, refer back to the [`hardhat` directory](https://github.com/CESSProject/cess-course/tree/main/examples/hardhat), its [`package.json`](https://github.com/CESSProject/cess-course/blob/main/examples/hardhat/package.json), and [`hardhat.config.ts`](https://github.com/CESSProject/cess-course/blob/main/examples/hardhat/hardhat.config.ts).


如果您有任何问题，请查看 [`hardhat`目录](https://github.com/CESSProject/cess-course/tree/main/examples/hardhat) 及其配置 [`package.json`](https://github.com/CESSProject/cess-course/blob/main/examples/hardhat/package.json)，和 [`hardhat.config.ts`](https://github.com/CESSProject/cess-course/blob/main/examples/hardhat/hardhat.config.ts) 档案。

## 开发

1. 现在开始配置 `hardhat.config.ts`，以便其可以将智能合约部署到 hardhat 网络，本地运行的 hardhat 节点，和本地运行的 CESS 节点。

    请在 `hardhat.config.ts` 使用以下配置：

    ```ts
    const config: HardhatUserConfig = {
      solidity: "0.8.19",
      defaultNetwork: "hardhat",
      namedAccounts: {
        deployer: {
          default: 0,
        },
        beneficiary: {
          default: 1,
        },
      },
      networks: {
        hardhat: {
          // issue: https://github.com/sc-forks/solidity-coverage/issues/652,
          // refer to : https://github.com/sc-forks/solidity-coverage/issues/652#issuecomment-896330136
          initialBaseFeePerGas: 0
        },
        localhost: {
          url: "http://localhost:8545",
          accounts: ["0xac0974bec39a17e36ba4a6b4d238ff944bacb478cbed5efcae784d7bf4f2ff80"],
        },
        "cess-local": {
          url: "http://localhost:9944", // RPC endpoint of CESS testnet
          chainId: 11330,
          // private key of `//Alice` from Substrate
          accounts: ["0xe5be9a5092b81bca64be81d212e7f2f9eba183bb7a90954f7b76361f6edb5c0a"],
        }
      }
    };
    ```

    以上配置了一个本地 hardhat 节点，倾听着 <http://localhost:8545> 端口，并有本地的 CESS 节点，倾听着 <http://localhost:9944> 端口。有这配置并用上和 `hardhat-deploy` 组件，我们可以通过 `pnpm hardhat deploy --network cess-local` 将智能合约部署到本地运行的 CESS 节点。

2. 对于智能合约，我们预期有以下方法：

    - `claim(bytes32 hash)`: 调用者使用指定哈希声明文件所有权的方法。
    - `forfeit(bytes32 hash)`: 调用者放弃具有指定哈希的文件所有权的方法。
    - `ownedFiles()`:  检索用户拥有的所有文件哈希值。
    - `hasClaimed(hash)`: 检查文件哈希是否已被声明。

    ```sol
    // SPDX-License-Identifier: UNLICENSED
    pragma solidity ^0.8.19;

    contract ProofOfExistence {
      mapping(bytes32 => address) public files;
      mapping(address => bytes32[]) public users;

      event Claimed(address indexed owner, bytes32 indexed file);
      event Forfeited(address indexed owner, bytes32 indexed file);

      error NotFileOwner();
      error FileAlreadyClaimed();

      modifier isOwner(bytes32 hash) {
        address from = msg.sender;
        if (files[hash] != from) revert NotFileOwner();
        _;
      }

      modifier notClaimed(bytes32 hash) {
        address from = msg.sender;
        if (files[hash] != address(0)) revert FileAlreadyClaimed();
        _;
      }

      function hasClaimed(bytes32 hash) public view returns (bool) {
        address owner = files[hash];
        return (owner != address(0));
      }

      function ownedFiles() public view returns (bytes32[] memory) {
        address from = msg.sender;
        return users[from];
      }

      function claim(bytes32 hash) public notClaimed(hash) returns (bool) {
        address from = msg.sender;

        // update storage files
        files[hash] = from;

        // update storage users
        bytes32[] storage userFiles = users[from];
        userFiles.push(hash);

        emit Claimed(from, hash);
        return true;
      }

      function forfeit(bytes32 hash) public isOwner(hash) returns (bool) {
        address from = msg.sender;

        // update storage files
        delete files[hash];

        // locate the index of the file going to be deleted.
        bytes32[] storage userFiles = users[from];
        uint32 delIdx = 0;
        for (uint32 i = 0; i < userFiles.length; i++) {
          if (userFiles[i] == hash) {
            delIdx = i;
            break;
          }
        }
        // update storage users by swap-delete
        if (delIdx != userFiles.length - 1) {
          userFiles[delIdx] = userFiles[userFiles.length - 1];
        }
        // delete
        userFiles.pop();

        emit Forfeited(from, hash);
        return true;
      }
    }
    ```

3. 现在，我们可以按照 [部署 Solidity 智能合约](./deploy-sc-solidity.md) 的教程, 部署合约到本地 CESS 节点上。

    - 在本地运行起 CESS 节点。
    - 准备好四个帐户及其地址。有关详细信息，请参阅 [Substrate EVM 之间地址互换](../guides/substrate-evm.md).
        1. Substrate 签名帐户，我们将使用 `/Alice`: `cXjmuHdBk4J3Zyt2oGodwGegNFaTFPcfC48PZ9NMmcUFzF6cc`。这个账户在本地 CESS 节点初始化了一些 TCESS 代币。
        2. 上述 CESS 签名账户的 EVM 映射账户，即 `0xd43593c715fdd31c61141abd04a99fd6822c8558`
        3. 一个 EVM 签名帐户，我们将 Alice 私钥 `0xe5be9a5092b81bca64be81d212e7f2f9eba183bb7a90954f7b76361f6edb5c0a` 导入进 Metamask 钱包，这会产生一个地址 `0x8097c3C354652CB1EEed3E5B65fBa2576470678A`。
        4. 上述 EVM 签名帐户 (#3) 的 Substrate 映射地址为 `cXgEvnbJfHsaN8HfoiEWfAi4QBENYbLKitRfG1CDYZpKTRRuw`.
    - 我们首先将一些测试代币（例如 1M 代币）从账户 #1 (Substrate 签名帐户) 透过 [Polkadot.js Apps](https://polkadot.js.org/apps/#/accounts) 转移到账户 #3 (EVM 账户) 。
    - 使用 #3 钱包中的代币，我们可以在本地 CESS 节点上部署智能合约。我们运行以下 hardhat 命令。这里，请相应更改 `hardhat.config.ts` 的部署帐户。

        ```bash
        pnpm hardhat deploy --reset --network cess-local
        ```

        请记下 ProofOfExistence 智能合约的部署地址。

4. 接下来，我们可以使用 [Remix](https://remix.ethereum.org/) 连接到智能合约并与智能合约进行交互。

# 前端

## 准备条件

- 安装 [Git](https://git-scm.com/book/en/v2/Getting-Started-Installing-Git)
- 安装 [Node v18](https://nodejs.org/en/download)
- 安装 [pnpm](https://pnpm.io/installation)
- 运行 CESS 节点的本地开发链，因为前端将连接到本地 CESS链。请 [参阅此处](./deploy-sc-ink.md#deploy-a-smart-contract) 了解如何运行本地 CESS 链。

完整的前端源代码可以在 [这里看到](https://github.com/CESSProject/cess-course/tree/main/examples/frontend)。

如果运行它，您将在右下角看到 **存在性证明（Solidity）小部件**：

![前端模版 PoE-Solidity](../../assets/developer/tutorials/poe-solidity/poe-solidity.png)

## 开发

前端的开发主要位于 `frontend/src/ProofOfExistenceSolidity.js`，如下[所示](https://github.com/CESSProject/cess-course/blob/main/examples/frontend/src/ProofOfExistenceSolidity.js)。

我们不会在此逐行列出前端代码，但会指出具体实现了的功能。

- 我们正在使用 React Hooks 的 [**wagmi**](https://wagmi.sh/) 库来与以 EVM 智能合约配合使用。

- 要使用 wagmi，我们将 CESS 本地链定义[在这里](https://github.com/CESSProject/cess-course/blob/308ec7fe053e92c08e4c2d634579f84b359072ac/examples/frontend/src/ProofOfExistenceSolidity.js#L22-L38)：


    ```js
    const RPC_ENDPOINT = "http://localhost:9944";

    const cessLocal = {
      id: 11330,
      name: "CESS Local",
      network: "cess-local",
      nativeCurrency: {
        decimal: 18,
        name: "CESS Testnet Token",
        symbol: "TCESS",
      },
      rpcUrls: {
        public: { http: [RPC_ENDPOINT] },
        default: { http: [RPC_ENDPOINT] },
      },
    };
    ```

- 然后，我们建立链的公共客户端并将客户端传递到 `createConfig()`。该 `config` 对象最后将被传递到 **WagmiConfig** React 组件中。

    ```jsx
    const { publicClient, webSocketPublicClient } = configureChains(
      [cessLocal],
      [
        jsonRpcProvider({
          rpc: (chain) => ({
            http: RPC_ENDPOINT,
          }),
        }),
      ],
    );

    const config = createConfig({
      publicClient,
      webSocketPublicClient,
    });

    //...
    export default function PoESolidityWithWagmiProvider(props) {
      return (
        <WagmiConfig config={config}>
          <PoESolidity />
        </WagmiConfig>
      );
    }
    ```

- 为了连接到我们的以太坊钱包，我们使用 [`useConnect()`](https://wagmi.sh/react/hooks/useConnect) hook，其将在 [**ConnectWallet**](https://github.com/CESSProject/cess-course/blob/308ec7fe053e92c08e4c2d634579f84b359072ac/examples/frontend/src/ProofOfExistenceSolidity.js#L180) 组件中被调用。

- 我们通过[`useAccount()`](https://wagmi.sh/react/hooks/useAccount) 和 [`useBalance()`](https://wagmi.sh/react/hooks/useBalance) hooks 获取当前所选账户的信息及其余额，这两个 hook 也将用于[**PoESolidity** 组件](https://github.com/CESSProject/cess-course/blob/308ec7fe053e92c08e4c2d634579f84b359072ac/examples/frontend/src/ProofOfExistenceSolidity.js#L65)。

- 我们使用 [`useContractRead()`](https://wagmi.sh/react/hooks/useContractRead)读取合约数据。这里需要注意的一点是我们需要指定 `account` 参数，以便在智能合约端设置好 `msg.sender`值。

- 为了写入合同，我们遵循[wagmi 文档](https://wagmi.sh/examples/contract-write-dynamic)中的做法：

    - 使用 `useDebounce()` 确保智能合约不会被频繁调用。
    - 使用 `usePrepareContractWrite()` 准备合约写入操作。
    - 使用 `useContractWrite()` 获取要在 **TxButton** 组件单击处理 `write()` 程序中传递的函数。
    - 使用 `useWaitForTransaction()` 获取交易信息及其状态。

# 教程完成

**恭喜，您已顺利完成本教程**！ 让我们回顾一下本教程：

- 您已成功在 Solidity 智能合约中实现了 PoE 逻辑，并将其部署在本地 CESS 节点上。
- 通过使用 Substrate 前端模板和 **wagmi** React hook 库，您成功实现了与智能合约交互的前端。

现在，您可以构建 dApp 并将其部署在 CESS 测试网上进行测试。也可以尝试 [使用 Ink! 开发 dApp 智能合约](./poe-ink.md)。

## 参考文档

- [Hardhat](https://hardhat.org/)
- [hardhat-deploy](https://github.com/wighawag/hardhat-deploy)
- [CESS Node](https://github.com/CESSProject/cess)
- [Substrate 前端模版](https://github.com/CESSProject/substrate-frontend-template)
- [wagmi](https://wagmi.sh/)
