# blog1


---
你需要准备一个良好的网络环境，如果没有，建议去购买成品或找别人配置

硬件配置：性能好一点的刷好armbian的电视盒子/开发板等等，推荐2+8及以上配置

软件环境：1.推荐使用命令行工具：Xshell（管理文件特别方便）

2.如果网络环境不理想强烈建议下载好镜像再导入，使用DockerPull


## 🔌 登录刷成 Armbian 的电视盒子

当你入手一个已经刷入 Armbian 系统的电视盒子后，第一步就是通过 SSH 登录它进行配置。以下是推荐的登录流程：

### 🛜 第一步：连接设备并查找 IP 地址

1. 将电视盒子通过网线或 Wi-Fi 接入你的家庭路由器。
2. 登录你的路由器后台（通常是 `192.168.1.1` 或 `192.168.0.1`）。
3. 在“连接设备”或“DHCP客户端列表”中查找最新上线的设备，通常名称为 `armbian` 或显示为未知设备。
4. 记录该设备的 IP 地址，例如 `192.168.1.123`。

### 🔐 第二步：获取默认账户信息

商家通常会提供默认的 SSH 登录用户名和密码，例如：

- 用户名：`root` 或 `armbian`
- 密码：`1234` 或商家自定义密码

> ⚠️ 登录后建议立即修改密码以保障安全。

### 💻 第三步：使用 Xshell 登录设备

1. 打开 Xshell，新建一个会话。
2. 在“主机”栏填入刚才查到的 IP 地址，例如 `192.168.1.123`。
3. 端口保持默认 `22`，协议选择 `SSH`。
4. 点击“连接”，在弹出的窗口中输入用户名和密码。
5. 登录成功后，你将进入 Armbian 的命令行界面。

### ✅ 登录成功后建议操作

- 修改默认密码：
  ```bash
  passwd
  ```
- 更新系统：
  ```bash
  sudo apt update && sudo apt upgrade -y
  ```
- 确认系统架构（最好是aarch64）：
  ```bash
  uname -m
  ```

---




## 一、系统准备（Armbian）

确保你的 Armbian 系统已更新并安装 Docker：

```bash
sudo apt update
sudo apt install docker.io docker-compose git
```

插入 RTL-SDR 设备后，确认识别：

```bash
lsusb
```

你应该看到类似：

```
Realtek RTL2838 DVB-T
```

---

## 二、安装 RTL-SDR 驱动

Armbian 支持直接安装：

```bash
sudo apt install rtl-sdr
```

卸载 DVB-T 驱动防止冲突：

```bash
sudo rmmod dvb_usb_rtl28xxu
echo "blacklist dvb_usb_rtl28xxu" | sudo tee /etc/modprobe.d/rtl-blacklist.conf
```

测试设备是否可用：

```bash
rtl_test
```

---

## 🐳 三、部署 OpenWebRX+ 容器（网络环境允许的情况用这个）

创建工作目录：

```bash
mkdir -p ~/openwebrx-plus
cd ~/openwebrx-plus
```

创建 `docker-compose.yml` 文件：

```yaml
version: "2"
services:
  openwebrx:
    image: jketterl/openwebrx-plus
    container_name: openwebrx
    ports:
      - "8073:8073"
    devices:
      - "/dev/bus/usb/001:/dev/bus/usb/001"
    volumes:
      - "./settings:/var/lib/openwebrx"
    restart: always
```

启动容器：

```bash
docker-compose up -d
```

---
## 🐳 三、部署 OpenWebRX+ 容器（网络环境不好的情况用这个）

使用 docker-pull-tar 下载 OpenWebRX+ 镜像（适用于 Armbian）

如果你的 Armbian 系统无法直接使用 `docker pull` 命令，或者你希望提前下载镜像并离线导入，可以使用 [topcss/docker-pull-tar](https://github.com/topcss/docker-pull-tar) 工具。该工具支持国内镜像源加速，适合 ARM 架构设备。

### 📦 推荐镜像信息

- 镜像名称：`slechev/openwebrxplus-softmbe:latest`
- 架构：`arm64` 或 `armhf`（根据你的设备架构确认）
- 仓库地址（默认）：`docker.xuanyuan.me`

---

### 🧰 下载工具

1. 打开项目页面：[https://github.com/topcss/docker-pull-tar](https://github.com/topcss/docker-pull-tar)
2. 进入 Releases 页面，下载 `DockerPull.exe`（适用于 Windows）：


### 🚀 下载镜像并生成 tar 包（命令行方式）

在 Windows 上打开命令行，执行：

```bash
DockerPull.exe -i slechev/openwebrxplus-softmbe:latest -a arm64 -r docker.xuanyuan.me
```

执行后将生成一个镜像文件，例如：

```
slechev_openwebrxplus-softmbe_latest_arm64.tar
```

---

### 📁 导入到 Armbian 系统

将 `.tar` 文件通过 U 盘或 SCP 传输到 Armbian 系统，然后执行：

```bash
docker load -i slechev_openwebrxplus-softmbe_latest_arm64.tar
```

导入成功后可通过以下命令验证：

```bash
docker images
```

---

这样你就完成了镜像的下载与导入，无需联网即可部署 OpenWebRX+ 接收站。

## 🔐 四、添加管理员账户

进入容器：

```bash
docker exec -it openwebrx bash
```

添加用户：

```bash
python3 /opt/openwebrx/openwebrx.py admin adduser ts
```

输入两次密码即可。

---

## 🧩 五、插件配置（如 tune_precise）

进入容器后编辑插件加载文件：

```bash
nano /usr/lib/python3/dist-packages/htdocs/plugins/receiver/init.js
```

添加：

```js
Plugins.load('https://0xaf.github.io/openwebrxplus-plugins/receiver/tune_precise/tune_precise.js');
```

保存后重启容器：

```bash
docker restart openwebrx
```

---

## 🔄 六、设置自启动（已在 Compose 中配置）

在 `docker-compose.yml` 中已设置：

```yaml
restart: always
```

系统重启后容器会自动启动。

---

## ✅ 七、访问与验证

打开浏览器访问：

```
http://<你的Armbian IP>:8073
```

使用你创建的账户登录，进入设置页面，确认 RTL-SDR 设备已识别，插件已加载。

---

如果你还想添加其他插件（如 SSTV、AIS、FAX）、启用数字语音解码（DMR、D-Star），或者配置远程访问，我可以继续帮你扩展这个接收站。你想下一步搞哪块？我随时在这儿。
