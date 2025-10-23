# VPN 安装与配置指南
**提示：如果看完后还是觉得难以理解，可以把这篇文章地址复制粘贴到 AI 聊天工具如 ChatGPT,Gemini,Claude 等应用中，让 AI 根据文档中的内容指导你一步步操作。**
## 为什么要创建这个文档？

长期以来，非专业用户在安装和配置 VPN 服务时常常面临较大困难，因此许多人会选择购买现成的 VPN 服务，或依赖 Web 面板与第三方 Shell 脚本来实现快速安装。
然而，工具链依赖越多，潜在的安全漏洞也越多。直接通过官方渠道安装软件，相对来说更安全可靠。

本文档旨在**教会用户如何从官方渠道安装和配置 VPN 服务**，避免使用第三方脚本和 Web 面板，从而提升安全性。

即便你没有相关经验，只会基本的电脑操作，只要认真阅读本教程，也能够成功完成 VPN 的搭建与配置。

VPN 服务由 **服务端** 与 **客户端** 构成：

* **服务端**：在服务器上运行的软件，也是教程中主要讲解的部分，你可以理解成一个流量转发工具，你在访问 YouTube 时，你会先和服务器上的流量转发工具建立加密连接。
* **客户端**：在个人设备（电脑、手机等）上运行，通过网络连接到服务端，实现加密数据传输与隐私保护。

---

## 一、服务器的选择

服务器厂商提供多种类型的产品（如 Docker、函数计算环境等），但为方便配置 VPN 和付款方便，推荐选择普通厂商提供的 **VPS（虚拟专用服务器）**。

### 为什么选择 VPS

* 拥有独立操作系统和完整权限，灵活度高。
* 云计算厂商（AWS、Google Cloud、Azure 等）的网络线路从中国访问并非最佳，且对于中国用户来说付款不方便。
* 对于个人用户，普通 VPS 服务商即可满足需求，并且大部分都支持无需信用卡购买。
* 部分商家还会针对 VPN 部署的进行 VPS 配置优化，比如提供更小的硬盘空间，更大的流量来满足特定需求。

### VPS 购买经验

1. **口碑与成立时间**：通过 Google 搜索了解。
2. **配置与流量**：常见配置如 1 核 CPU、1GB 内存、500GB 流量每月、100M 带宽，足以支撑单人长期使用。价格约 30–50 美元/年。
3. **网络线路**：
   * 因中国独特的网络环境，普通线路访问速度可能较慢。
   * 推荐选择支持 **CN2、CMI、CNC2** 等下一代互联网骨干网络的 VPS，延迟低、丢包率低、带宽利用率高。
4. **测速**：
   * 向商家索要测速链接或从官方网站查找，一般都会提供（如 `http://1.1.1.1/100MB.test`）。
   * 通过浏览器下载测试速度，例如 100M 带宽应当达到 10MB/s 左右。
   * 若远低于标称速率，建议更换 VPS。

> 建议首次购买时选择 **月付**，确认稳定后再续费长期套餐。

---

## 二、操作系统的选择

推荐使用 **Ubuntu Server LTS 最新版本**（如 24.04）。

* Ubuntu 提供稳定的长期支持且几乎所有的VPS厂家都支持该系统。
* 开箱即用，无需折腾。
* 文档中软件安装均基于 `apt` 包管理器，Debian 及其衍生版同样适用。
* 购买 VPS 时通常可直接选择操作系统，如未选择，可在控制台重新安装。

购买成功后，厂商会提供以下信息：

* **IP 地址**
* **用户名**
* **密码**
* **端口号**（如果商家没有提供，默认使用22，只有极少数商家会指定一个随机的端口号，如 Bandwagonhost）

---

## 三、服务器登录

登录 VPS 需使用 **SSH 客户端**。以下分别介绍 macOS 与 Windows 的方法。

### macOS

1. 打开 **终端**（Spotlight 输入 `终端`）。
2. 输入命令：

   ```bash
   ssh root@1.1.1.1 #1.1.1.1 替换成你的服务器IP
   ```
3. 输入密码（输入时无显示，可粘贴）。
4. 若 VPS 提供了自定义端口（如 2222），需使用：

   ```bash
   ssh root@1.1.1.1 -p 2222
   ```

### Windows

* **Windows 10/11**：内置 SSH 客户端，可直接使用命令行（CMD/PowerShell）登录，端口号的指定方法和 macOS 一致：

  ```bash
  ssh root@1.1.1.1
  ```
* **旧版本 Windows**：需安装 PuTTY（[下载地址](https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html)），使用方法参考官方文档。

---

## 四、Ubuntu 系统基础设置（可选）

不同厂商的 Ubuntu 配置可能存在差异，建议统一进行以下设置，确保环境一致。

在终端中逐行输入以下命令（注释部分仅作说明，无需输入）：

```bash
# 1. 更新系统软件
sudo apt update && sudo apt upgrade -y

# 2. 设置时区与时间同步
sudo timedatectl set-timezone Asia/Shanghai
sudo timedatectl set-ntp true
timedatectl   # 查看时间

# 3. 配置防火墙
sudo apt install ufw   # 若未安装 ufw
sudo ufw enable        # 启用防火墙
sudo ufw allow 22      # 放行 SSH 端口
sudo ufw allow 443     # 放行 HTTPS 端口
sudo ufw allow 0-65535 # 按需放行其他端口，例如 2222
sudo ufw status        # 查看防火墙状态
```

> 若安装软件速度较慢，可更换软件源。某些 VPS 可能会使用远距离镜像（如美国服务器使用德国源），会导致速度缓慢。

---

## 五、服务端软件选择

传统 VPN 协议（PPTP、IPSec 等）已不再推荐。
现代抗审查 VPN 技术不断演进，应选择 **开源、社区活跃**的软件。
不用担心配置过于复杂，就是修改一个配置文件的事。你把那些面板和SHELL脚本的功能理解成在用可视化的方式在创建修改配置文件，但是我如果直接给你配置文件的模版，你只需要修改几个参数，然后保存即可，这样不是更简单吗。

以下为主流选项：

### 1. Xray

* **优点**：稳定，支持多种传输协议
* **缺点**：配置相对复杂
* [安装配置指南](#xray)

### 2. Hysteria2

* **优点**：配置简单，高性能，带宽利用率高
* **缺点**：部分地区可能限制 UDP 端口，导致不稳定
* [安装配置指南](#hysteria2)

---

## 六、客户端选择

客户端是运行在终端设备上的软件，用于连接 VPN 服务器。

### iOS / macOS

* **Shadowrocket**（收费）
  [下载地址](https://apps.apple.com/us/app/shadowrocket/id932747118)
* **Hiddify**
  [下载地址](https://apps.apple.com/us/app/hiddify-proxy-vpn/id6596777532)
* **Streisand**
  [下载地址](https://apps.apple.com/us/app/streisand/id6450534064)

> 注意：
>
> * macOS Intel 芯片可使用 v2rayN（macOS 版本）。
> * macOS M 系列芯片可直接安装 iOS 应用。

### Windows

* **v2rayN**（主流开源客户端，支持 Windows，macOS，Linux）
  [下载地址](https://github.com/2dust/v2rayN)

### Android

* **NekoBox**（开源免费）
  [下载地址](https://github.com/MatsuriDayo/NekoBoxForAndroid)

> 注意：仅从 GitHub 官方仓库下载，作者已经声明应用商店中非官方版本。
---
## 服务端安装与配置

如果你已经学会如何使用SSH，并且已经准备好了服务器，可以跳到这里。选择其中一个安装即可。


### Xray

[Xray 官方文档地址](https://github.com/XTLS/Xray-install/blob/main/README_zh-Hans.md)

---

#### 安装

复制下面的命令到服务器终端执行即可：

```bash
bash -c "$(curl -L https://github.com/XTLS/Xray-install/raw/main/install-release.sh)" @ install
````

---

#### 配置文件

安装完成后，编辑 `/usr/local/etc/xray/config.json`
该文件为 Xray 的配置文件。

* 如果不会在终端中修改，可以搜索 `vi` 或 `nano` 的使用方法。
* 更简单的方法是使用 **SFTP 客户端**（如 Windows 上的 WinSCP、macOS 上的 Cyberduck 等）。
* 使用 SFTP 登录服务器后，进入 `/usr/local/etc/xray/`，找到 `config.json`，用文本编辑器打开修改即可。

---

#### 必要修改项

1. 获取 UUID

   ```bash
   xray uuid
   ```

   示例输出：

   ```
   b99f9345-6ffd-436c-a57c-8b9a02ce5c9f
   ```

2. 生成公钥和私钥

   ```bash
   xray x25519
   ```

   示例输出：

   ```
   publicKey:  7f3da9c4e26b42a1a91d8f84b2c0e7d1e98
   privateKey: 1c9e8a6f34bd71d2294cf0a9f0aef62a24e
   ```

3. 修改配置文件中对应字段：

   * **第 30 行 `id`** → 替换为上面获取的 `UUID`
   * **第 45 行 `privateKey`** → 替换为上面获取的 `privateKey`（注意不是 `publicKey`）

除以上两处外，其余配置保持默认，避免出错。

配置文件示例来自官方仓库：
👉 [Xray-examples](https://github.com/XTLS/Xray-examples)

```json
{
    "log": {
        "loglevel": "debug"
    },
    "inbounds": [
        {
            "tag": "dokodemo-in",
            "port": 443,
            "protocol": "dokodemo-door",
            "settings": {
                "address": "127.0.0.1",
                "port": 4431,  // 指向内网中的 reality 端口，示例是这个端口，如果要自己修改了记得这里和下面的 reality 入站都要修改
                "network": "tcp"
            },
            "sniffing": { // 这里的 sniffing 不是多余的，别乱动
                "enabled": true,
                "destOverride": [
                    "tls"
                ],
                "routeOnly": true
            }
        },
        {
            "listen": "127.0.0.1",
            "port": 4431, // 见上 如果和其他服务冲突了可以换
            "protocol": "vless",
            "settings": {
                "clients": [
                    {
                        "id": "" // 运行 xray uuid 生成UUID
                    }
                ],
                "decryption": "none"
            },
            "streamSettings": {
                "network": "tcp",
                "security": "reality",
                "realitySettings": {
                    // 下方要求和普通 reality 一致，这里演示 dest 设置为 cloudflare 不被偷跑流量所以设置为 speed.cloudflare.com 了
                    // 你可以设置为其他 CF 网站，如果你的 dest 不是这种网站你也不用点了进来不是吗
                    "dest": "speed.cloudflare.com:443",
                    "serverNames": [
                        "speed.cloudflare.com"
                    ],
                    "privateKey": "", // 运行 `xray x25519` 生成
                    "shortIds": [
                        "",
                        "0123456789abcdef"
                    ]
                }
            },
            "sniffing": {
                "enabled": true,
                "destOverride": [
                    "http",
                    "tls",
                    "quic"
                ],
                "routeOnly": true
            }
        }
    ],
    "outbounds": [
        {
            "protocol": "freedom",
            "tag": "direct"
        },
        {
            "protocol": "blackhole",
            "tag": "block"
        }
    ],
    "routing": {
        "rules": [
            {
                "inboundTag": [
                    "dokodemo-in"
                ],
                // 重要，这个域名列表需要和 realitySettings 的 serverNames 保持一致
                "domain": [
                    "speed.cloudflare.com"
                ],
                "outboundTag": "direct"
            },
            {
                "inboundTag": [
                    "dokodemo-in"
                ],
                "outboundTag": "block"
            }
        ]
    }
}
```

---

#### 启动与管理

使用 `systemctl` 控制 Xray：

```bash
sudo systemctl start xray      # 启动
sudo systemctl enable xray     # 加入开机启动
sudo systemctl stop xray       # 停止
sudo systemctl restart xray    # 重启
sudo systemctl status xray     # 查看状态
```
---
#### 客户端连接说明

在客户端配置连接时，请准备以下信息：

* **协议**：VLESS
* **服务器 IP 地址**：你的服务器 IP
* **端口**：443
* **UUID**：与配置文件中保持一致
* **SNI**：`speed.cloudflare.com`
* **Public Key**：之前生成的公钥
* **Short ID**：与配置文件中保持一致
* **传输方式**：TCP
* **安全协议**：

  * 如果客户端支持 **Reality**，请勾选
  * 如果没有 Reality 选项，请选择 **TLS**

##### 注意事项

* 不同客户端的操作界面可能有所差异，无法提供统一的配置样板。
* 只需将上述信息对应填写到客户端的配置界面中即可。
* 如果某些字段在客户端中找不到，请保持为空。



### Hysteria2


**官方文档**: [Hysteria2](https://v2.hysteria.network/zh/docs/getting-started/Installation/)

---

#### 1. 安装

在服务器终端执行：

```bash
bash <(curl -fsSL https://get.hy2.sh/)
```

---

#### 2. 生成自签名证书

```bash
openssl req -x509 -nodes -newkey ec:<(openssl ecparam -name prime256v1) -keyout /etc/hysteria/server.key -out /etc/hysteria/server.crt -subj "/CN=bing.com" -days 36500 && sudo chown hysteria /etc/hysteria/server.key && sudo chown hysteria /etc/hysteria/server.crt
```

---

#### 3. 配置文件

配置文件路径：`/etc/hysteria/config.yaml`

> 修改方法参考 Xray 配置文件的修改步骤，可根据需要修改，通常只需要修改密码即可。

```yaml
listen: :443

tls:
  cert: /etc/hysteria/server.crt
  key: /etc/hysteria/server.key

auth:
  type: password
  password: Se7RAuFZ8Lzg  # 设置密码

masquerade:
  type: proxy
  proxy:
    url: https://bing.com
    rewriteHost: true
```

---

#### 4. 启动与管理

```bash
# 设置开机自启并立即启动
systemctl enable --now hysteria-server.service

# 重启服务（修改配置后执行）
systemctl restart hysteria-server.service

# 查询服务状态
systemctl status hysteria-server.service
```

---

#### 5. 客户端连接说明

需要准备以下信息：

* **协议**: hysteria2
* **IP**: 你的服务器 IP
* **端口**: 443
* **密码**: 与配置文件中一致
* **TLS**: 打开“允许不安全”，SNI 填 `bing.com`

> 补充说明：
> Hysteria2 配置相对简单。对于中国用户，有些运营商可能阻断 UDP 连接。官方提供 **端口跳跃** 方案，可参考：[Port Hopping](https://v2.hysteria.network/zh/docs/advanced/Port-Hopping/)
>

---

个人水平有限，欢迎批评指正。有好的开源协议，后续我会继续更新和优化。
