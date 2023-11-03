# 安装过程中可能出现的问题

<details>
  <summary>无法下载 docker 镜像</summary>

  安装过程中将使用 docker 下载 cess 镜像。如果安装 `cess-nodeadm` 时出现如下异常：

  ![Docker 常驻进程问题](../assets/storage-miner/troubleshooting/docker-daemon-issue.png)

  确保指令是在 root 权限下执行，或使用 sudo 命令执行。然后在您的系统上启动 docker：

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
  <summary>无法找到 docker 库</summary>

  如果安装时出现如下错误 `cess-nodeadm`：

  ![Docker 库问题](../assets/storage-miner/troubleshooting/docker-package-issue.webp)

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

  ![CESS 下载镜像失败](../assets/storage-miner/troubleshooting/cess-image-download-issue.png)

  确保以 root 权限或使用 `sudo` 命令运行命令。

  然后再尝试执行 `cess config set` 命令。
</details>

<details>
  <summary>无效的配置文件 (config.yaml)</summary>

  ![无效配置](../assets/storage-miner/troubleshooting/invalid-config-issue.webp)


  删除文件 `/usr/bin/yq`：

  ```bash
  sudo rm /usr/bin/yq
  ```

  再次重新安装 `cess-nodeadm`：

  ```bash
  ./install.sh
  ```
</details>
