# 概述

在本教程中，我们将通过 [**cess-js-sdk**](https://www.npmjs.com/package/cess-js-sdk) 与 CESS 测试网进行交互。本教程适用于想要将 CESS 合并到自己产品/系统中的 Node.js 或前端开发人员。

# 准备条件

- [Node.js](https://nodejs.org/en) v18 或更高版本

# 开发代码

教程代码存在于 [**cess-course** Github repo](https://github.com/CESSProject/cess-course/tree/main/examples/js) 中，我们将在本教程中一起浏览这些代码。

教程代码将执行以下操作：

1. 检查我租用的空间数量及其到期时间（涉链上读取操作）
2. 在 CESS 测试网上租用空间（涉链上覆写操作）
3. 创建一个存储桶（涉链上覆写操作）
4. 查询我在 CESS 云中的现有文件（涉链上读取操作）
5. 将文件上传到存储桶（涉链上覆写操作）
6. 将文件下载回来。

{% hint style="info" %}

**CESS 测试网 RPC 端点**: <wss://testnet-rpc0.cess.cloud/ws/><br/>
**CESS 测试网文件网关**: <https://deoss-pub-gateway.cess.cloud/>

{% endhint %}

首先，通过运行以下命令安装 `cess-js-sdk`。

```bash
pnpm add cess-js-sdk
# or
yarn add cess-js-sdk
# or
npm add cess-js-sdk
```

这些命令将 **cess-js-sdk** 依赖项添加到您的项目中。

然后我们通过调用 `InitAPI()` 初始化 API：

```ts
import {
  InitAPI,
  Space,
  Bucket,
  File,
  testnetConfig
} from "cess-js-sdk";

async function main() {
  const { api, keyring } = await InitAPI(testnetConfig);
  const space = new Space(api, keyring);
  const bucket = new Bucket(api, keyring);
  const file = new File(api, keyring, testnetConfig.gatewayURL);
}
```

该 `InitAPI()` 函数接收了一个 `CESSConfig` 对象，如下：

```ts
interface CESSConfig {
  nodeURL: string;    // This is the RPC endpoint connecting to
  gatewayURL: string; // This is the DeOSS gateway URL. Please use the above provided one for testnet
  keyringOption: {
    type: string;     // By default it is SR25519 (ref: https://wiki.polkadot.network/docs/learn-cryptography#keypairs-and-signing)
    ss58Format: number; // Prefix for CESS. For CESS Testnet, it is `42`
  };
}
```

`testnetConfig` 是一个 **CESSConfig** 实例，其配置连接到 CESS 测试网。然后我们可以使用 `api` 和 `keyring` 返回值来创建 **Space** 、**Bucket** 和 **File** 服务。这些服务中的每一个都需要 `api` 和 `keyring` 作为参数传入。

接下来，让我们更深入地了解一下 **Space** 服务的使用。

## 使用 Space 服务

我们首先来看一下 [`checkNRentSpace()` 函数](https://github.com/CESSProject/cess-course/blob/308ec7fe053e92c08e4c2d634579f84b359072ac/examples/js/src/index.ts#L57)。

```ts
async function checkNRentSpace(space: Space, common: Common) {
  const initSpace = await space.userOwnedSpace(acctId);
  console.log("query userOwnedSpace:", initSpace.data);

  const blockHeight = await common.queryBlockHeight();
  console.log("current block height:", blockHeight);

  let spaceData = common.formatSpaceInfo(initSpace.data, blockHeight);
  console.log("initial user space:", spaceData);

  if (!spaceData.totalSpace) {
    const rsResult = await space.buySpace(mnemonic, RENT_SPACE);
    console.log(rsResult);
  }
}
```

您可以通过 `space.userOwnedSpace()` 查询您的已使用空间，它将返回一个类似于以下内容的对象：

```ts
{
  totalSpace: 40802189312,
  usedSpace: 150994944,
  lockedSpace: 75497472,
  remainingSpace: 40600862720,
  start: 403800,
  deadline: 1397400,
  state: 'normal'
}
```

然后通过有一个工具库 **Common** 可以格式化上述数据，使其更易于阅读。

```ts
common.formatSpaceInfo(initSpace.data, blockHeight)`
```

将返回以下对象：

```ts
{
  totalSpace: 40802189312,
  usedSpace: 150994944,
  lockedSpace: 75497472,
  remainingSpace: 40600862720,
  start: 403800,
  deadline: 1397400,
  state: 'normal',
  totalSpaceGib: 38,
  totalSpaceStr: '38.00 GB',
  usedSpaceGib: 0.140625,
  usedSpaceStr: '144.00 MB',
  lockedSpaceGib: 0.0703125,
  lockedSpaceStr: '72.00 MB',
  remainingSpaceGib: 37.8125,
  remainingSpaceStr: '37.81 GB',
  deadlineTime: '2023-12-17 13:42:07',
  remainingDays: 52
}
```

您能发现 `blockHeight` 作为参数传入 `formatSpaceInfo()` 以计算 `deadlineTime`。

如果这是您第一次从 CESS 租用空间，请使用以下 API：

```
buySpace(mnemonic: string, gibCount: number): Promise<any>
```

或者，您可以用 `expandSpace()` 来扩展容量，或用 `renewalSpace()` 延长租用空间的长度。

- `expansionSpace(mnemonicOrAccountId32: string, gibCount: number): Promise<any>`
- `renewalSpace(mnemonic: string, days: number): Promise<any>`

## 使用 Bucket 服务

我们通过 **Bucket** 服务来查询用户创建的 Bucket 信息（类似于我们本地硬盘中的目录的概念）、创建 Bucket、删除 Bucket。

我们看一下 [`checkNCreateBucket()` 函数](https://github.com/CESSProject/cess-course/blob/308ec7fe053e92c08e4c2d634579f84b359072ac/examples/js/src/index.ts#L73)。

```ts
async function checkNCreateBucket(bucket: Bucket) {
  let res = await bucket.queryBucketList(acctId);
  console.log("queryBucketList", res.data);

  res = await bucket.queryBucketInfo(acctId, BUCKET_NAME);
  console.log("queryBucketInfo", res);

  if (res.data) {
    console.log("deleting bucket...");
    // The bucket exists already, so let's delete it first
    await bucket.deleteBucket(mnemonic, acctId, BUCKET_NAME);
  }

  res = await bucket.createBucket(mnemonic, acctId, BUCKET_NAME);
  console.log("createBucket", res);

  res = await bucket.queryBucketList(acctId);
  console.log("queryBucketList", res.data);
}
```

`queryBucketList()` 将返回类似于以下内容的对象数组：

```ts
[
  {
    objectList: [
      '7b7eba922840cc2df17a5ee5ff9504ca56d857dc5938a10dd9e0ddaa1b7a0e3f',
      // hash values
    ],
    key: 'xhftBucket'
  },
  { objectList: [], key: 'test' },
  { objectList: [], key: 'test2' }
]
```

每一个项目都是一个以`key`作为 Bucket 名字的 Bucket。所以该用户有三个存储 Bucket，分别名为`xhftBucket`、`test`、 和`test2`。`xhftBucket` 内有一个对象 （文件），而 `test` 及 `test2` 则是空的 bucket。

我们可以使用 `queryBucketInfo()` 查询有关特定存储 Bucket的更多详细信息，将返回以下内容：

```ts
{
  objectList: [],
  authority: [ 'cXgaee2N8E77JJv9gdsGAckv1Qsf3hqWYf7NL4q6ZuQzuAUtB' ]
}
```

这包含了有关存储 Bucket 内的文件以及对这些文件具有写入权限的用户的信息。默认情况下，只有存储 Bucket 的创建者可以对其进行写入。但用户可以使用 **授权服务** 将此访问权限授予其他用户。

您可以使用 `createBucket()` 创建存储 Bucket 并使用 `deleteBucket()` 删除存储 Bucket。这两个功能都要求您传递助记词，因为您将向 CESS 区块链发送签名交易（需要您的私钥）。

## 使用 File 服务

让我们看看如何查询我们上传的文件。现在查看代码 [`uploadNDownloadFile()函数`](https://github.com/CESSProject/cess-course/blob/308ec7fe053e92c08e4c2d634579f84b359072ac/examples/js/src/index.ts#L93):

```ts
async function uploadNDownloadFile(fileService: File) {
  let res = await fileService.queryFileListFull(acctId);
  console.log("queryFileListFull", res.data);
  const { fileHash } = res.data[0];

  const uploadFile = `${process.cwd()}/package.json`;
  console.log("uploadFile path:", uploadFile);
  res = await fileService.uploadFile(mnemonic, acctId, uploadFile, BUCKET_NAME);
  console.log("uploadFile", res);

  const downloadPath = `${process.cwd()}/download`;
  res = await fileService.downloadFile(fileHash, downloadPath);
  console.log("downloadFile", res);
}
```

我们用 `queryFileListFull()` 查询用户帐户可以访问的文件，将返回一个类似于以下内容的对象：

```ts
[
  {
    fileHash: '7b7eba922840cc2df17a5ee5ff9504ca56d857dc5938a10dd9e0ddaa1b7a0e3f',
    fileSize: 25165824,
    fileSizeStr: '29.00 B',
    fileName: 'hello.world',
    bucketName: 'xhftBucket',
    stat: 'Active'
  },
  {
    fileHash: 'b50b924f6292da622e6bb68407738becba1d19d6bffe18b7d5d7b37f819c312d',
    fileSize: 25165824,
    fileSizeStr: '24.00 B',
    fileName: 'xhft_test20231009.txt',
    bucketName: 'xhftBucket',
    stat: 'Active'
  },
  // ...
]
```

这包含了文件名、文件大小、所属存储 Bucket 和文件哈希。

如果要将文件上传到 CESS 去中心存储，请使用 `uploadFile()` 。除了账户助记词之外，还需要上传文件路径和存储 Bucket 的名称作为参数。

文件上传后，您可以在云端查询该文件以获取其哈希值，并使用该哈希值来下载文件，以 `downloadFile()` 返回。

# API

**Space 服务**

该服务用于管理用户空间。您可以查询、更新和扩展CESS中已使用的空间。

- `userOwnedSpace(accountId32: string): Promise<APIReturnedData>`
- `buySpace(mnemonic: string, gibCount: number): Promise<any>`
- `expansionSpace(mnemonicOrAccountId32: string, gibCount: number): Promise<any>`
- `renewalSpace(mnemonic: string, days: number): Promise<any>`

**Bucket 服务**

用于管理用户存储 Bucket，类似于本地硬盘中的目录概念。

- `queryBucketNames(accountId32: string): Promise<APIReturnedData>`
- `queryBucketList(accountId32: string): Promise<APIReturnedData>`
- `queryBucketInfo(accountId32: string, name: string): Promise<APIReturnedData>`
- `createBucket(mnemonic: string, accountId32: string, name: string): Promise<any>`
- `deleteBucket(mnemonic: string, accountId32: string, name: string): Promise<any>`

**File 服务**

该服务用于管理 CESS cloud 中的用户文件。

- `queryFileListFull(accountId32: string): Promise<APIReturnedData>`
- `queryFileList(accountId32: string): Promise<APIReturnedData>`
- `queryFileMetadata(fileHash: string): Promise<APIReturnedData>`
- `uploadFile(mnemonic: string, accountId32: string, filePath: string, bucketName: string): Promise<any>`
- `downloadFile(fileHash: string, savePath: string): Promise<any>`
- `deleteFile(mnemonic: string, accountId32: string, fileHashArray: string[]): Promise<any>`


**Authorize 服务**

该服务用于管理向其他用户授予的权限。

- `authorityList(accountId32: string): Promise<APIReturnedData>`
- `authorize(mnemonic: string, operator: string): Promise<any>`
- `cancelAuthorize(mnemonic: string, operator: string): Promise<any>`

# 结语

使用 **cess-js-sdk** API，您可以将 CESS cloud 整合到您的产品或系统中。如您对本教程有任何反馈，请随时与[我们联系](../../introduction/contact.md)。
