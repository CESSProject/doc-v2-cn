# 背景

CESS 测试网已上线。用户上传文件需要先购买空间与授权网关。本篇指南是针对想通过区块链浏览器直接与链进行交互进行购买空间、扩容空间等操作的的开发者的指南。

# 指南

指南中会讲解，如何购买空间、扩容空间，续租空间，以及其中的一些细节。

## 购买空间
以英文版区块链浏览器为例：

1. 在连接了任意rpc节点后，浏览器加载出区块链信息后。我们选择 Developer - Extrinsics 如下图所示。

![进入交易页面](../../assets/developer/guides/space-operation/pic1.png)

2. 选择 StorageHandler 模块， 选择 buySpace 交易。

![找到指定交易](../../assets/developer/guides/space-operation/pic2.png)

3. 在 gibCount 一栏内，填写想要购买的存储空间大小，单位以 gib 计算。确认参数无误后，点击 Submit Transaction

        此步骤请确保你的账户余额充足。当前CESS测试网的空间单价为 30 TCESS / GIB / DAY

![填写参数](../../assets/developer/guides/space-operation/pic3.png)

4. 点击 Sign and Submit 进行签名确认。 

![提交交易](../../assets/developer/guides/space-operation/pic4.png)

5. 随后等待浏览器唤起钱包进行签名，输入您自定义的密码后，点击 Sign the transaction 签名此交易。

![签名交易](../../assets/developer/guides/space-operation/pic5.png)

6. 等待交易被打包，然后广播确认。 如看到下图二所示的内容后，则说明您提交的交易已成功执行。

        需要注意，您的账户如果已购买过空间的情况，再此调用该交易会失败，如果想扩容或续租请调用相应的交易。

![广播](../../assets/developer/guides/space-operation/pic6.png)

![成功](../../assets/developer/guides/space-operation/pic7.png)

## 续租空间

在拥有空间后可以续租空间， 以英文版区块链浏览器为例：  

1. 选择 StorageHandler 模块。 然后选择 renewalSpace 交易。填写参数 days 无误后，发送交易。  

        days 代表续租的天数。

![续租交易](../../assets/developer/guides/space-operation/pic8.png)

2. 后续操作与购买空间相同。

## 扩容空间

在拥有空间后可以扩容空间， 以英文版区块链浏览器为例：  

1. 选择 StorageHandler 模块。 然后选择 expansionSpace 交易。填写参数 gibCount 无误后，发送交易。  

![扩容交易](../../assets/developer/guides/space-operation/pic9.png)

## 查询空间

通过区块链浏览器，可以直接查询某个钱包地址持有存储空间的状态。

1. 选择 Developer - Chain state

![状态查询](../../assets/developer/guides/space-operation/pic10.png)

2. 选择 StorageHandler 模块。继续选择 userOwnedSpace 接口。 填入您想查询的钱包地址。

![选择查询](../../assets/developer/guides/space-operation/pic11.png)

3. 点击 “+” 进行查询。查看返回结果。

![进行查询](../../assets/developer/guides/space-operation/pic12.png)

`totalSpace`: 表示用户当前持有的总空间，以字节为单位。  

`usedSpace`: 表示用户当前上传文件所占用的空间，以字节为单位。  

`lockedSpace`: 表示用户当前正在上传中的文件所使用的空间，以字节为单位。

`remainingSpace`: 表示用户剩余可使用的空间，以字节为单位。  

`start`: 初次购买空间时的块高。

`deadline`: 过期时的块高。

`state`: 当前用户所持有空间的状态。