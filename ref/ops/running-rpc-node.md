RPC 节点不像共识节点那样直接参与出块工作，而是负责验证交易并在不同节点之间、节点与客户端之间中继通信，促进交易验证和链上信息检索等工作。如果您想运行自己的RPC 节点，有两种方法:

1. 通过 [**cess-nodeadm**](https://github.com/CESSProject/cess-nodeadm) 启动。
2. 直接运行 [**cess-node**](https://github.com/CESSProject/cess)。

## 通过 cess-nodeadm 运行 RPC 节点

1. 检查 cess-nodeadm 最新版本
   cess-nodeadm 最新版本位置: <https://github.com/CESSProject/cess-nodeadm/tags><br/>
   ⚠️ 本小节下文中所有的 `x.x.x` 用最新的版本号代替，例如最新的版本号是 `v0.5.3`，则 `x.x.x` 用 `0.5.3` 代替。

2. 检查已安装的 cess-nodeadm 版本
   在控制台中输入 `cess version` 命令，检查 `nodeadm version` 版本是否是最新的版本。
   如果nodeadm 是最新版本，则可跳过第 3 步。如果不是最新的版本则需要走第3步进行安装。如果没有看到 nodeadm version ，说明未安装过 cess-nodeadm，则需要走第3步进行安装。

3. 下载并安装 cess-nodeadm
   ```shell
   wget https://github.com/CESSProject/cess-nodeadm/archive/vx.x.x.tar.gz
   tar -xvf vx.x.x.tar.gz
   cd cess-nodeadm-x.x.x/
   ./install.sh
   ```

4. 停止RPC节点服务
   输入命令：`cess stop chain`，停止已运行的 RPC 节点服务

5. 定义脚本配置参数
   ```shell
   cess config set Enter cess node mode from 'authority/storage/watcher' (current: watcher, press enter to skip): watcher
   Enter cess node name (current: cess, press enter to skip): local-chain
   Enter cess chain pruning mode, 'archive' or number (current: null, press enter to skip): archive  #number of blocks saved
   ```

6. 启动RPC节点
   ```shell
   cess start chain
   ```

7. 查看RPC节点是否正常同步区块
   ```shell
   docker logs chain
   ```

## 直接运行 RPC 节点

1. 安装 rust 环境
   参考 [substrate官方教程](https://docs.substrate.io/install/)

2. 获取 cess-node 最新的发布版本
   [检查 cess-node 最新的版本](https://github.com/CESSProject/cess/tags)

   以v0.7.5为最新版本为例，下载并解压cess-node程序：

   ```shell
   wget https://github.com/CESSProject/cess/archive/v0.7.5.tar.gztar -zxvf v0.7.5.tar.gz
   ```

3. 编译 **cess-node**

   进入cess-node目录：
   ```shell
   cd cess-0.7.5/
   cargo build --release
   ```

4. 启动RPC服务
   ```shell
   # 0.7.5版本以前含0.7.5版本输入：
   ./target/release/cess-node --base-path 【您自定义数据库存放路径】 --chain cess-testnet --port 30333 --ws-port 9944 --rpc-port 9933 --unsafe-rpc-external --unsafe-ws-external --name 【您自定义的名字】 --rpc-cors all --ws-max-connections 2020 --state-pruning archive

   #0.7.5版本以后输入：
   ./target/release/cess-node --base-path 【您自定义数据库存放路径】 --chain cess-testnet --port 30333 --rpc-port 9944 --unsafe-rpc-external --name 【您自定义的名字】 --rpc-cors all --rpc-max-connections 2020 --state-pruning archive
   ```

   若节点正在打印区块同步日志，则代表运行成功。

   ⚠️ 您需要保持 cess-node 程序一直运行，建议使用 `screen`或 `tmux` 命令在后台运行 `cess-node`。
