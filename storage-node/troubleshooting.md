# 安装过程中可能出现的问题

<details>
  <summary>无法下载 docker 镜像</summary>

  安装过程中，使用 docker 下载 cess 镜像。如果安装 `cess-nodeadm` 时出现如下异常：

  <img alt="Docker 進程問題" src="../assets/storage-node/troubleshooting/docker-daemon-issue.png" width="100%" height="auto" decoding="async" style="max-width: 100%"/>

  确保指令是在 root 权限下或使用 sudo 命令執行。在您的系统上启动 docker：

  ```bash
  systemctl start docker
  ```

  重新安装 `cess-nodeadm`：

  ```bash
  ./install.sh
  ```

  ⚠️ 注意，所有 CESS 程序命令都必须具有 sudo 权限。
</details>

<details>
  <summary>无法找到 docker 庫</summary>

  如果安装时出现如下错误 `cess-nodeadm`：

  <img alt="Docker 庫問題" src="../assets/storage-node/troubleshooting/docker-package-issue.webp" width="100%" height="auto" decoding="async" style="max-width: 100%;" />

  尝试使用以下命令删除 Docker：

  ```bash
  sudo systemctl stop docker
  docker stop $(docker ps -aq)
  docker rm -v $(docker ps -aq)
  docker rmi $(docker images -aq)
  docker volume rm $(docker volume ls -q)
  brew uninstall docker
  ```

  重新安装 Docker：

  ```bash
  sudo apt-get install docker-ce
  sudo systemctl enable docker
  sudo systemctl start docker
  ```
</details>

# 配置过程中可能出现的问题

<details>
  <summary>下载 CESS 镜像失败</summary>

  如果设置 config 时出现如下错误：

  <img alt="CESS 下载镜像失败" src="../assets/storage-node/troubleshooting/cess-image-download-issue.png" width="100%" height="auto" decoding="async" style="max-width: 100%;" />

  确保以 root 权限或使用 `sudo` 命令运行命令。

  再尝试執行 `cess config set` 命令。
</details>

<details>
  <summary>无效的配置文件 (config.yaml)</summary>

  <img alt="无效配置" src="../assets/storage-node/troubleshooting/invalid-config-issue.webp" width="100%" height="auto" decoding="async" style="max-width: 100%;" />

  删除文件 `/usr/bin/yq`：

  ```bash
  sudo rm /usr/bin/yq
  ```

  再次重新安装 `cess-nodeadm`：

  ```bash
  ./install.sh
  ```
</details>
