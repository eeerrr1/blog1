# blog1


---
你需要准备一个良好的网络环境，如果没有，建议去购买成品或找别人配置
硬件配置：性能好一点的刷好armbian的电视盒子/开发板等等，推荐2+8及以上配置
软件环境：1.推荐使用命令行工具：Xshell（管理文件特别方便）
2.如果网络环境不理想强烈建议下载好镜像再导入，使用DockerPull

请确认你的armbian系统架构

## 系统架构确认（建议在部署前执行）

在终端中执行以下命令，确认你的系统架构和是否为 64 位：

```bash
uname -m
```

常见输出解释如下：

| 输出结果     | 架构类型 | 位数说明     |
|--------------|----------|--------------|
| `x86_64`     | AMD/Intel | 64 位系统     |
| `aarch64`    | ARM       | 64 位系统     |
| `armv7l`     | ARM       | 32 位系统     |
| `armhf`      | ARM       | 32 位系统     |

如果你看到的是 `aarch64`，说明你的设备是 ARM 架构的 64 位系统，适合运行大多数插件和容器。如果是 `armv7l` 或 `armhf`，则为 32 位系统，某些功能可能受限。


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

## 🐳 三、部署 OpenWebRX+ 容器

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
