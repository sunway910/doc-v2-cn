## 第一步、准备RPC 节点

您可以在自己的机器中运行自己的RPC节点，或者使用CESS官方提供的RPC节点。

如果您使用CESS官方提供的RPC节点，从下面列表中进行选择：

>wss://testnet-rpc0.cess.cloud/ws/
>wss://testnet-rpc1.cess.cloud/ws/
>wss://testnet-rpc2.cess.cloud/ws/
如果您想运行自己的RPC节点，有两种方法：一是通过cess-nodeadm程序启动，二是直接运行cess-node程序。下面分别介绍了这两种运行方法。

### 1）通过cess-nodeadm 程序运行RPC节点

1. 检查cess-nodeadm最新的版本
cesss-nodeadm最新版本号位置：[https://github.com/CESSProject/cess-nodeadm/tags](https://github.com/CESSProject/cess-nodeadm/tags)

>⚠️本小节下文中所有的x.x.x用最新的版本号代替，例如最新的版本号是v0.5.3，则x.x.x用0.5.3代替。
2. 检查已安装的cess-nodeadm版本
在控制台中输入cess version命令，检查nodeadm version版本是否是最新的版本。

>如果nodeadm version是最新的版本，则可以跳过第3步。
>如果不是最新的版本则需要走第3步进行安装。
>如果没有看到nodeadm version，说明未安装过cess-nodeadm，则需要走第3步进行安装。
3. 下载并安装cess-nodeadm程序
```plain
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

### 2）直接运行cess-node程序

1. 安装rust环境
参考substrate官方教程: [https://docs.substrate.io/install/](https://docs.substrate.io/install/)

2. 获取cess-node 最新的发布版本
检查cess-node最新的版本：[https://github.com/CESSProject/cess/tags](https://github.com/CESSProject/cess/tags)

以v0.7.5为最新版本为例，下载并解压cess-node程序：

```plain
wget https://github.com/CESSProject/cess/archive/v0.7.5.tar.gz
tar -zxvf v0.7.5.tar.gz
```

3. 编译cess-node程序
进入cess-node目录：

```plain
cd cess-0.7.5/
```
编译cess-node程序：
```plain
cargo build --release
```

4. 启动RPC服务
```bash
#0.7.5版本以前含0.7.5版本输入：  
./target/release/cess-node --base-path 【您自定义数据库存放路径】 --chain cess-testnet --port 30333 --ws-port 9944 --rpc-port 9933 --unsafe-rpc-external --unsafe-ws-external --name 【您自定义的名字】 --rpc-cors all --ws-max-connections 2020 --state-pruning archive
#0.7.5版本以后输入： 
./target/release/cess-node --base-path 【您自定义数据库存放路径】 --chain cess-testnet --port 30333 --rpc-port 9944 --unsafe-rpc-external --name 【您自定义的名字】 --rpc-cors all --rpc-max-connections 2020 --state-pruning archive
```
若节点正在打印区块同步日志，则代表运行成功。
>⚠️您需要保持cess-node程序一直运行，建议使用[screen](https://linuxize.com/post/how-to-use-linux-screen/)或[tmux](https://linuxize.com/post/getting-started-with-tmux/)命令为cess-node开启独立的窗口。
## 第二步、安装docker和docker compose

以ubuntu（官方推荐存储节点操作系统）为例安装docker：

1.更新系统软件包列表

```plain
sudo apt update
```
2.安装必要的依赖项：
```bash
sudo apt install apt-transport-https ca-certificates curl software-properties-common
```
3.添加Docker GPG密钥：
```bash
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
```
4.设置Docker仓库：
```bash
echo "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```
5.更新软件包索引并安装Docker引擎：
```bash
sudo apt update
sudo apt install docker-ce docker-ce-cli containerd.io
```
6.将当前用户添加到docker组中，这样不需要使用sudo命令来运行docker，重启后生效：
```bash
sudo usermod -aG docker $USER
```

## 
## 第三步、配置存储节点信息

1.为每个存储节点创建工作目录（以运行两个存储节点为例，两个存储程序的数据目录分别位于/mnt/disk0和/mnt/disk1，您可以修改为自己的目录。）：

```bash
cd /mnt/disk0/
mkdir bucket storage
cd /mnt/disk1/
mkdir bucket storage
```
其中bucket目录用于存放存储节点配置文件，storage目录作为存储节点运行的工作目录；
2.复制以下内容，分别在每个存储节点bucket目录下创建config.yaml文件并粘贴：

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
需要注意，每个存储节点应设置不同的工作账户，工作路径和端口等
>请注意，配置RPC时，第一个是本地的RPC节点地址，若采用第三种类型则只需要配置外部RPC地址；配置文件中展示的第2-第4个RPC地址为CESS官方推荐地址，Boot中展示的地址为CESS官方提供的存储节点boot node节点地址。
3.配置好配置文件的存储节点目录应当如下图所示：

![图片](data:image/png;base64,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)


## 第四步、配置并启动存储节点容器

请根据以下内容创建docker-compose.yaml文件，用于批量启动存储节点容器：

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
上述文件中，运行多少个存储节点容器就需要配置多少个存储节点服务，yaml文件通过缩进两格来表示层级关系，如上述文件中，配置了bucket_0和bucket_1两个服务；每个服务中需要重点配置服务名，容器名以及宿主机到容器的目录映射；如在bucket_0中，目录映射配置如下：
```yaml
    volumes: #Mapping of host disk to container
      - '/mnt/disk0/bucket:/opt/bucket' #Node configuration directory
      - '/mnt/disk0/storage/:/opt/bucket-disk' #Node working directory
```
其中"/mnt/disk0/bucket/"是之前创建好的存放节点0配置文件的目录，里面有config.yaml， "/mnt/disk0/storage/"是之前创建好的存储节点0用于工作的工作目录；
watchtower服务用于监控各存储节点容器状态，并自动为容器动态更新最新的镜像，每台服务器配置一个该服务即可；

可以将docker-compose.yaml文件放在任何可访问的地方，配置好文件后，运行 `docker compose up -d` 命令来启动存储节点容器。

您可以通过`docker ps -a`命令查看存储节点的运行状态。

