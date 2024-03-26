# Architecture

多存储节点架构如下所示:
- watchTower: 当本地的 bucket 镜像与官方的 bucket 镜像有差异时，watchtower会自动拉取官方的新镜像，在创建一个新的 bucket 容器后删除旧的 bucket 容器
- bucket: 存储节点，多个存储节点之间会进行 P2P 通信，在示例配置文件中配置的通信端口为：15001、15002
- chain: 链节点，存储节点默认通过链节点的 9944 端口获取区块信息；链节点之间默认通过 30336 端口进行数据同步

![多存储节点架构](../assets/storage-miner/multi-buckets/multibucket.png)

# 方法一：一键安装多个存储节点

## 1. 下载并安装 multibucket 管理客户端

```bash
sudo wget https://github.com/CESSProject/cess-multibucket-admin/archive/v0.0.1.tar.gz
sudo tar -xvf v0.0.1.tar.gz
cd cess-multibucket-admin-0.0.1
sudo bash ./install.sh
```

## 2. 自定义配置文件
{% hint style="info" %}
安装完成后, 用户需要根据实际需求修改文件: `/opt/cess/multibucket-admin/config.yaml`
{% endhint %}

- UseSpace: 存储节点的存储能力，单位为 GB
- UseCpu: 存储节点使用的逻辑核心数
- port: 存储节点 P2P 通信端口，每个存储节点的端口必须不同且没有被其他服务占用
- diskPath: 存储节点工作的系统绝对路径，需要在该路径下挂载文件系统
- earningsAcc: 收入账户， [如何获取账户和助记词](https://docs.cess.cloud/core/v/zh/storage-miner/running#zhun-bei-cess-zhang-hu)
- stakingAcc: 质押 TCESS 的付款账户，每提供 1T 的存储空间需要质押 4000 个 TCESS
- mnemonic: 账户助记词，由 12 个单词组成，每个存储节点需要提供不同的助记词
- chainWsUrl: 默认通过本地的 RPC 节点进行存储节点之间的数据同步，`buckets[].chainWsUrl` 的优先级高于 `node.chainWsUrl`
- backupChainWsUrls: 备用 RPC 节点，可以使用官方提供的 RPC 节点或者你所知的其他 RPC 节点，`buckets[].backupChainWsUrls` 的优先级高于 `node.backupChainWsUrls`

{% hint style="warning" %}
你可以通过 lvm 的方式在一块硬盘上创建多个虚拟逻辑卷，将多个虚拟逻辑卷挂载在不同的 diskPath 上，但当该硬盘损坏时，所有依赖 lvm 的存储节点都出现故障 !
{% endhint %}

   ```yaml
   ## node configurations template
   node:
      ## the mode of node: multibucket
      mode: "multibucket"
      ## the profile of node: devnet/testnet/mainnet
      profile: "testnet"
      # default chain url for bucket, this config will be overwritten in buckets[] as below
      chainWsUrl: "ws://127.0.0.1:9944/"
      # default backup chain urls for bucket, this config will be overwritten in buckets[] as below
      backupChainWsUrls: ["wss://testnet-rpc0.cess.cloud/ws/", "wss://testnet-rpc1.cess.cloud/ws/", "wss://testnet-rpc2.cess.cloud/ws/", "wss://testnet-rpc3.cess.cloud/ws/"]

   ## chain configurations
   ## set option: '--skip-chain' or '-s' to skip installing chain
   ## if set option: --skip-chain, please set official chain in bucket[].chainWsUrl
   chain:
      ## the name of chain node
      name: "cess"
      ## the port of chain node
      port: 30336
      ## listen rpc service at port 9944
      rpcPort: 9944

   ## bucket configurations  (multi storage miner mode)
   buckets:
      - name: "bucket_1"
        # P2P communication port
        port: 15001
        # Maximum space used, the unit is GiB
        UseSpace: 1000
        # Number of cpu's used, 0 means use all
        UseCpu: 2
        # earnings account
        earningsAcc: "cXxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"
        # Staking account
        # If you fill in the staking account, the staking will be paid by the staking account,
        # otherwise the staking will be paid by the earningsAcc.
        stakingAcc: "cXxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"
        # Signature account mnemonic
        # each bucket's mnemonic should be different
        mnemonic: "aaaaa bbbbb ccccc ddddd eeeee fffff ggggg hhhhh iiiii jjjjj kkkkk lllll"
        # a directory mount with filesystem
        diskPath: "/mnt/cess_storage1"
        # The rpc endpoint of the chain
        # `official chain: wss://testnet-rpc0.cess.cloud/ws/ wss://testnet-rpc1.cess.cloud/ws/ wss://testnet-rpc2.cess.cloud/ws/`, "wss://testnet-rpc3.cess.cloud/ws/"
        chainWsUrl: "ws://127.0.0.1:9944/"
        backupChainWsUrls: []
        # Priority tee list address
        TeeList:
           - "127.0.0.1:8080"
           - "127.0.0.1:8081"
        # Bootstrap Nodes
        Boot: "_dnsaddr.boot-bucket-testnet.cess.cloud"
        
      - name: "bucket_2"
        # P2P communication port
        port: 15002
        # Maximum space used, the unit is GiB
        UseSpace: 1000
        # Number of cpu's used
        UseCpu: 2
        # earnings account
        earningsAcc: "cXxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"
        # Staking account
        # If you fill in the staking account, the staking will be paid by the staking account,
        # otherwise the staking will be paid by the earningsAcc.
        stakingAcc: "cXxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"
        # Signature account mnemonic
        # each bucket's mnemonic should be different
        mnemonic: "lllll kkkkk jjjjj iiiii hhhhh ggggg fffff eeeee ddddd ccccc bbbbb aaaaa"
        # a directory mount with filesystem
        diskPath: "/mnt/cess_storage2"
        # The rpc endpoint of the chain
        # `official chain: wss://testnet-rpc0.cess.cloud/ws/ wss://testnet-rpc1.cess.cloud/ws/ wss://testnet-rpc2.cess.cloud/ws/`, "wss://testnet-rpc3.cess.cloud/ws/"
        chainWsUrl: "ws://127.0.0.1:9944/"
        backupChainWsUrls: []
        # Priority tee list address
        TeeList:
           - "127.0.0.1:8080"
           - "127.0.0.1:8081"
        # Bootstrap Nodes
        Boot: "_dnsaddr.boot-bucket-testnet.cess.cloud"
   ```

## 3. 生成配置文件
执行以下命令会根据文件：`/opt/cess/multibucket-admin/config.yaml` 去生成每个存储节点的配置文件 `config.yaml` 和 `docker-compose.yaml` 配置文件

  ```bash
  sudo cess-multibucket-admin config generate
  ```

- 每一个存储节点的配置文件会生成在其对应挂载路径的 `bucket` 目录下，如存储节点 `bucket_1` 的配置文件生成在： `/mnt/cess_storage1/bucket/config.yaml`
- docker-compose.yaml 文件生成在路径： `/opt/cess/multibucket-admin/build/docker-compose.yaml`

{% hint style="warning" %}
除非自定义了配置文件的路径，否则以： `/opt/cess/multibucket-admin/config.yaml` 为准
{% endhint %}

## 4. 一键安装多个存储节点

### 启动所有服务
启动 watchTower、multi-buckets、rpc 服务
  ```bash
  sudo cess-multibucket-admin install
  ```

### 跳过安装本地 RPC 节点
如果配置文件中配置了官方的 RPC 节点或其他已知的 RPC 节点，可以通过 `--skip-chain` 跳过启动本地的 RPC 节点

  ```bash
  sudo cess-multibucket-admin install --skip-chain
  ```

## 5. 常用操作

**停止某个服务**，如 `sudo cess-multibucket-admin stop bucket_1` 停止存储节点：`bucket_1`
```bash
  sudo cess-multibucket-admin stop $1
```

**停止所有服务**
```bash
  sudo cess-multibucket-admin stop
```

**停止所有服务并删除相关镜像**
```bash
  sudo cess-multibucket-admin down
```

**重启所有服务**
```bash
  sudo cess-multibucket-admin restart
```

**重启某个服务**，如 `sudo cess-multibucket-admin reload bucket_1` 重新运行存储节点：`bucket_1`
```bash
  sudo cess-multibucket-admin restart $1
```

**查看版本相关信息**
```bash
  sudo cess-multibucket-admin version
```

**查看服务状态**
```bash
  sudo cess-multibucket-admin status
```

**重新拉取镜像**
```bash
  sudo cess-multibucket-admin pullimg
```

**查看当前磁盘使用情况**
```bash
  sudo cess-multibucket-admin tools space-info
```

**查看存储桶状态**
```bash
  sudo cess-multibucket-admin buckets stat
```

**增加质押**
```bash
  sudo cess-multibucket-admin buckets increase staking <deposit amount>
```

**撤回质押**

当您的节点**退出 CESS 网络**（见下文）后，运行以下命令撤回质押：
```bash
  sudo cess-multibucket-admin buckets withdraw
```

**查询奖励信息**
```bash
  udo cess-multibucket-admin buckets reward
```

**领取奖励**
```bash
  sudo cess-multibucket-admin buckets claim
```

**更新收入账户**
```bash
  sudo cess-multibucket-admin buckets update earnings [earnings account]
```

**退出 CESS 网络**
```bash
  sudo cess-multibucket-admin buckets exit
```

**移除现有的服务**

{% hint style="warning" %}
除非 CESS 网络已重新部署且确认数据可以清除，否则请勿执行此操作。
{% endhint %}
```bash
  sudo cess-multibucket-admin purge
```

# 方法二：依次安装多个存储节点

## 1. 准备 RPC 节点

您可以在自己的机器中运行自己的 RPC 节点，或者使用 CESS 官方提供的 RPC 节点。

如果您使用 CESS 官方提供的 RPC 节点，从下面列表中进行选择：

- `wss://testnet-rpc0.cess.cloud/ws/`
- `wss://testnet-rpc1.cess.cloud/ws/`
- `wss://testnet-rpc2.cess.cloud/ws/`

如果您想运行自己的 RPC 节点，有两种方法：一是通过 cess-nodeadm 程序启动，二是直接运行 cess-node 程序。下面分别介绍了这两种运行方法。

### 通过 cess-nodeadm 程序运行 RPC 节点

1. 检查 cess-nodeadm 最新的版本
   cesss-nodeadm最新版本号位置：<https://github.com/CESSProject/cess-nodeadm/tags>

   ⚠️本小节下文中所有的x.x.x用最新的版本号代替，例如最新的版本号是v0.5.5，则x.x.x用0.5.5代替。

2. 检查已安装的 cess-nodeadm 版本
   在控制台中输入 `cess version` 命令，检查 `nodeadm version` 版本是否是最新的版本。

   如果 `nodeadm version` 是最新的版本，则可以跳过第3步。如果不是最新的版本则需要走第3步进行安装。
   如果没有看到 `nodeadm version`，说明未安装过 `cess-nodeadm`，则需要走第 3 步进行安装。

3. 下载并安装cess-nodeadm程序

   ```bash
   wget https://github.com/CESSProject/cess-nodeadm/archive/vx.x.x.tar.gz
   tar -xvf vx.x.x.tar.gz
   cd cess-nodeadm-x.x.x/
   ./install.sh
   ```

4. 停止cess-node服务
   停止服务命令：cess stop chain

5. 选择脚本配置参数
   ```plain
   cess config set

   Enter cess node mode from 'authority/storage/watcher' (current: watcher,
   press enter to skip): watcher
   Enter cess node name (current: cess, press enter to skip): local-chain
   Enter cess chain pruning mode, 'archive' or number (current: null, press
   enter to skip): archive #number of blocks saved 
   ```

6. 启动本地链节点
   ```plain
   cess start chain
   ```

7. 查看链节点是否正常同步区块
   ```plain
   docker logs chain
   ```

### 直接运行 cess-node 程序

1. 安装rust环境
   [参考 Substrate 官方教程](https://docs.substrate.io/install/)

2. 获取 cess-node 最新的发布版本
   [检查 cess-node 最新的版本](https://github.com/CESSProject/cess/tags)

   以v0.7.7为最新版本为例，下载并解压cess-node程序：

   ```bash
   wget https://github.com/CESSProject/cess/archive/v0.7.7.tar.gz
   tar -zxvf v0.7.7.tar.gz
   ```

3. 编译cess-node程序
   进入cess-node目录：

   ```bash
   cd cess-0.7.7/
   cargo build --release
   ```

4. 启动RPC服务

   ```bash
   # 0.7.7版本以前含0.7.7版本输入：
   ./target/release/cess-node --base-path 【您自定义数据库存放路径】 --chain cess-testnet --port 30333 --ws-port 9944 --rpc-port 9933 --unsafe-rpc-external --unsafe-ws-external --name 【您自定义的名字】 --rpc-cors all --ws-max-connections 2020 --state-pruning archive

   #0.7.7版本以后输入：
   ./target/release/cess-node --base-path 【您自定义数据库存放路径】 --chain cess-testnet --port 30333 --rpc-port 9944 --unsafe-rpc-external --name 【您自定义的名字】 --rpc-cors all --rpc-max-connections 2020 --state-pruning archive
   ```

   若节点正在打印区块同步日志，则代表运行成功。

   ⚠️您需要保持cess-node程序一直运行，建议使用 [**screen**](https://linuxize.com/post/how-to-use-linux-screen/) 或 [**tmux**](https://linuxize.com/post/getting-started-with-tmux/) 命令为 cess-node 开启独立的窗口。

## 2. 安装 Docker 和 Docker Compose

以 Ubuntu（官方推荐存储节点操作系统）为例安装 docker.

这里参看[官方文档](https://docs.docker.com/engine/install/)。

1. 更新系统软件包列表

   ```bash
   sudo apt update
   ```

2. 安装必要的依赖项：

   ```bash
   sudo apt install apt-transport-https ca-certificates curl software-properties-common
   ```

3. 添加 Docker GPG 密钥：

   ```bash
   curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
   ```

4. 设置 Docker 仓库：

   ```bash
   echo "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
   ```

5. 更新软件包索引并安装Docker引擎：

   ```bash
   sudo apt update
   sudo apt install docker-ce docker-ce-cli containerd.io
   ```

6. 将当前用户添加到docker组中，这样不需要使用sudo命令来运行docker，重启后生效：

   ```bash
   sudo usermod -aG docker $USER
   ```

## 3. 配置存储节点信息

1. 为每个存储节点创建工作目录（以运行两个存储节点为例，两个存储程序的数据目录分别位于 `/mnt/disk0` 和 `/mnt/disk1`，您可以修改为自己的目录。）:

   ```bash
   cd /mnt/disk0/
   mkdir bucket storage
   cd /mnt/disk1/
   mkdir bucket storage
   ```

   其中bucket目录用于存放存储节点配置文件，storage目录作为存储节点运行的工作目录；

2. 复制以下内容，分别在每个存储节点 bucket 目录下创建 config.yaml 文件并粘贴：

   ```yaml
   # The rpc endpoint of the chain node
   Rpc:
     - "ws://127.0.0.1:9944/"
     - "wss://testnet-rpc0.cess.cloud/ws/"
     - "wss://testnet-rpc1.cess.cloud/ws/"
     - "wss://testnet-rpc2.cess.cloud/ws/"
   # Bootstrap Nodes
   Boot:
     - "_dnsaddr.boot-bucket-testnet.cess.cloud"
   # Signature account mnemonic
   Mnemonic: "xxx xxx ... xxx"
   # Staking account
   # If you fill in the staking account, the staking will be paid by the staking account,
   # otherwise the staking will be paid by the signature account.
   StakingAcc: "cXxxx...xxx"
   # earnings account
   EarningsAcc: cXxxx...xxx
   # Service workspace
   Workspace: "/opt/bucket-disk"
   # P2P communication port
   Port: 4001
   # Maximum space used, the unit is GiB
   UseSpace: 2000
   # Number of cpu's used, 0 means use all
   UseCpu: 4
   # Priority tee list address
   TeeList:
     - "127.0.0.1:8080"
     - "127.0.0.1:8081"
   ```

   需要注意，每个存储节点应设置不同的工作账户，工作路径和端口等。

   请注意，配置RPC时，第一个是本地的 RPC 节点地址，若采用第三种类型则只需要配置外部 RPC 地址；配置文件中展示的第2-第4个 RPC 地址为 CESS 官方推荐地址，Boot中展示的地址为 CESS 官方提供的存储节点 boot node 节点地址。

3. 配置好配置文件的存储节点目录应当如下图所示：

   ![配置结构](../assets/storage-miner/multi-buckets/folder.png)

## 4. 配置并启动存储节点容器

请根据以下内容创建 `docker-compose.yaml` 文件，用于批量启动存储节点容器：

```yaml
version: '3'
name: cess-storage
services:
   bucket_0: #services name
      image: 'cesslab/cess-bucket:testnet'
      network_mode: host
      restart: always
      volumes: #Mapping of host disk to container
         - '/mnt/disk0/bucket:/opt/bucket' #Node configuration directory
         - '/mnt/disk0/storage/:/opt/bucket-disk' #Node working directory
      command:
         - run
         - '-c'
         - /opt/bucket/config.yaml
      logging:
         driver: json-file
         options:
            max-size: 500m
      container_name: bucket0 #container name
   bucket_1:
      image: 'cesslab/cess-bucket:testnet'
      network_mode: host
      restart: always
      volumes:
         - '/mnt/disk1/bucket:/opt/bucket' #Node configuration directory
         - '/mnt/disk1/storage/:/opt/bucket-disk' #Node working directory,
      command:
         - run
         - '-c'
         - /opt/bucket/config.yaml
      logging:
         driver: json-file
         options:
            max-size: 500m
      container_name: bucket1
   watchtower: #only needs to be run once
      image: containrrr/watchtower
      container_name: watchtower
      network_mode: host
      restart: always
      volumes:
         - '/var/run/docker.sock:/var/run/docker.sock'
      command:
         - '--cleanup'
         - '--interval'
         - '300'
         - '--enable-lifecycle-hooks'
         - chain
         - bucket
      logging:
         driver: json-file
         options:
            max-size: 100m
            max-file: '7'
```

上述文件中，运行多少个存储节点容器就需要配置多少个存储节点服务，yaml 文件通过缩进两格来表示层级关系，如上述文件中，配置了 `bucket_0` 和 `bucket_1` 两个服务；每个服务中需要重点配置服务名，容器名以及宿主机到容器的目录映射；如在 `bucket_0` 中，目录映射配置如下：

```yaml
    volumes: #Mapping of host disk to container
      - '/mnt/disk0/bucket:/opt/bucket' #Node configuration directory
      - '/mnt/disk0/storage/:/opt/bucket-disk' #Node working directory
```

其中 `/mnt/disk0/bucket/` 是之前创建好的存放节点 0 配置文件的目录，里面有 `config.yaml`。`/mnt/disk0/storage/` 是之前创建好的存储节点 0 用于工作的工作目录；

`watchtower` 服务用于监控各存储节点容器状态，并自动为容器动态更新最新的镜像，每台服务器配置一个该服务即可；

可以将 `docker-compose.yaml` 文件放在任何可访问的地方，配置好文件后，运行 `docker compose up -d` 命令来启动存储节点容器。

您可以通过 `docker ps -a` 命令查看存储节点的运行状态。
