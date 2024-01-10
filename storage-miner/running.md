# 服务器要求

存储服务器的推荐要求：

| 资源 | 规格 |
| --- | --- |
| 推荐操作系统 | Linux 64 位英特尔/AMD |
| CPU 核心数 | ≥4 |
| 内存 | ≥8GB |
| 带宽 | ≥5Mbps |
| 公网 IP | 必需 |
| Linux 内核版本 | 5.11 或更高版本 |

# 服务器准备

## 安装 Docker

Docker 安装请参考[官方文档](https://docs.docker.com/engine/install/)。

## 防火墙配置

{% hint style="info" %}
以下命令均以 root 权限执行。如果出现错误消息 `permission denied`，请切换到 root 权限或在这些命令的开头添加 `sudo`。
{% endhint %}

默认情况下，节点客户端 **cess-bucket**，使用端口 4001 监听传入连接，如果您的平台阻止该端口 (或在防火墙监控内) ，您需要打开对该端口的访问。

```bash
ufw allow 4001
```

## 磁盘挂载

使用以下命令检查硬盘状态`df -h`：

```bash
$ df -h
```

如果该磁盘并未挂载，则不会在存储挖矿中被使用上。使用以下命令查看硬盘是否已挂载：

```bash
$ fdisk -l

Disk /dev/vdb: 200 GiB, 214748364800 bytes, 419430400 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0x331195d1
```

由上述可知，未挂载的磁盘为 `/dev/vdb`。我们将用来 `/dev/vdb` 演示，安装操作如下。

分配 `/dev/vdb` 磁盘：


```bash
fdisk /dev/vdb
Enter and press Enter:
n
p
1
2048
the value after default
w
```

将新划分的磁盘格式化为 `ext4` 格式：

```bash
mkfs.ext4 /dev/vdb
```

如果系统要求进一步输入，请输入 `y` 继续：

```bash
Proceed anyway? (y,N) y
```

创建 `/cess` 目录来挂载磁盘。这里用 `/cess` 目录作为例子：

```bash
mkdir /cess
echo "/dev/vdb /cess ext4 defaults 0 0" >> /etc/fstab
```

将 `/dev/vdb` 替换为您自己的磁盘名称。 命令中的 `/cess` 必须与上一步中创建的相同。如果您没有 root 权限，请尝试：

```bash
echo "/dev/vdb /cess ext4 defaults 0 0" | sudo tee -a /etc/fstab
```

安装 `/cess`：

```bash
mount -a
```

查看磁盘挂载状态：

```bash
df -h
```

如果 `/cess` 出现则说明磁盘挂载成功。

## 准备 CESS 账户

矿工需要创建两个钱包账户。

- **收益账户**：用于接收挖矿获得的奖励。
- **质押账户**：用于支付存储节点的质押费用。
- **签名账户**：用于签名区块链交易。 如果没有指定质押账户，该账户也将用于支付质押费用。

请参阅 [CESS 帐户](../community/cess-account.md) 创建 CESS 帐户，然后前往 [CESS 水龙头](https://cess.cloud/faucet.html) 获取测试代币 TCESS，或 [联系我们](../introduction/contact.md) 寻求帮助。

# 安装 CESS 客户端
1. Check for the latest version at：https://github.com/CESSProject/cess-nodeadm/tag
```
⚠️ According to the latest version number, replace the following x.x.x
```
2. Download and install
```bash
wget https://github.com/CESSProject/cess-nodeadm/archive/vx.x.x.tar.gz
tar -xvf vx.x.x.tar.gz
cd cess-nodeadm-x.x.x/
./install.sh
```

如果出现该条消息—— `Install cess nodeadm success`，则表示安装成功。

如果安装失败，请查看 [故障排除步骤](./troubleshooting.md)。

## 停止并移除旧服务
停止旧的服务：
```
cess stop
```
或者
```
cess down
```
移除旧的服务：
```
cess purge
```

## 配置 CESS 客户端
### 设置网络环境
切换到CESS开发网：
```
cess profile devtnet
```
切换到CESS测试网：
```
cess profile testnet
```

### 配置配置文件
```bash
$ cess config set

Enter cess node mode from 'authority/storage/watcher' (current: watcher, press enter to skip): storage
Enter cess storage listener port (current: 15001, press enter to skip): 
Enter cess storage earnings account (current: cXiqKzVVamJ2d5cMKomh1ED4prAnKevr2v3nZgNH87HRuY4Xy, press enter to skip): 
Enter cess storage staking signature phrase (current: situate double coral cycle ritual country rebuild ridge slush smoke verb acquire, press enter to skip): 
Enter cess storage disk path (current: /mnt/storage-disk, press enter to skip): 
Enter cess storage space, by GB unit (current: 300, press enter to skip): 
Enter the number of CPU cores used for mining; Your CPU cores are 4
  (current: 3, 0 means all cores are used; press enter to skip): 

Set configurations successfully
```

启动 CESS bucket

```bash
$ cess start

[+] Running 3/0
 ✔ Container chain       Running                                                0.0s
 ✔ Container bucket      Running                                                0.0s
 ✔ Container watchtower  Running                                                0.0s
```

如果您想加快存储矿工获取收益的速度，可以选择部署Marker型TEE Worker来帮助矿工认证空间和为服役文件打标，请参阅[TEE Worker用户指南](./TEE-Worker-Guide.md)

# 常用操作

## 检查 CESS 链同步状态

```bash
docker logs chain
```

如下图所示，如果我们在 [CESS 浏览器](https://testnet.cess.cloud/) 中看到 “best” 对应的区块高度约为最新高度，则说明本地链节点同步完成。

![CESS 区块链同步完成](../assets/storage-miner/running/sync-status.png)

只有链同步完成后，才能操作其他功能，如增加质押、查看节点状态等。

## 查看存储节点日志

```bash
docker logs bucket
```

如下图，看到 `/kldr-testnet` 表示网络环境是测试网络，看到 `Connected to the bootstrap node...` 表示有和 bootstrap 节点的连接。

![存储节点日志](../assets/storage-miner/running/view-node-log.webp)

## 查看存储桶状态

```bash
cess bucket stat
```

返回结果示例如下：

![CESS Bucket 统计](../assets/storage-miner/running/bucket-stat.png)

上述名称的进一步解释，请参阅 [术语对照](../glossary.md#storage-miner)。

## 增加质押

```bash
cess bucket increase <deposit amount>
```

## 撤回质押

当您的节点**退出 CESS 网络**（见下文）后，运行以下命令：

```bash
cess bucket withdraw
```

## 查询奖励信息

```bash
cess bucket reward
```

## 领取奖励

```bash
cess bucket claim
```

## 更新所有服务镜像

```bash
cess pullimg
```

## 停止并删除所有服务

```bash
cess down
```

## 更新收入账户

```bash
cess bucket update earnings [earnings account]
```

## 退出 CESS 网络

```bash
cess bucket exit
```

# 升级 CESS 客户端

## 停止并删除所有服务

```bash
cess stop
cess down
```

## 删除所有链数据

{% hint style="warning" %}
除非 CESS 网络已重新部署且确认数据可以清除，否则请勿执行此操作。
{% endhint %}

```bash
cess purge
```

## 更新 `cess-nodeadm`

```bash
wget https://github.com/CESSProject/cess-nodeadm/archive/vx.x.x.tar.gz
tar -xvf vx.x.x.tar.gz

cd cess-nodeadm-x.x.x

./install.sh --skip-dep
```

## 拉取镜像

```bash
cess pullimg
```
