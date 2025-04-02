```markdown
# Open Duck Mini 运行时

## Raspberry Pi zero 2W 设置

### 安装 Raspberry Pi OS

从以下链接下载 Raspberry Pi OS Lite (64 位)：https://www.raspberrypi.com/software/operating-systems/

按照以下说明将操作系统安装到 SD 卡上：https://www.raspberrypi.com/documentation/computers/getting-started.html

使用 Raspberry Pi Imager，您可以预先配置会话、Wi-Fi 和 SSH。请按照以下步骤操作：

![imager_setup](https://github.com/user-attachments/assets/7a4987b2-de83-41dd-ab7f-585259685f16)

> 提示：我配置了树莓派连接到我的手机热点，这样我可以从任何地方连接到它。

> 译者注：这个后续我们会推一个搞好了下面这些步骤的镜像，尽可能避免配置麻烦

### 设置 SSH（如果在安装过程中未设置）

> 译者注：用树莓派的镜像刷写工具，在刷镜像的时候就可以配WIFI和SSH。SSH不是默认开启的，需要手动开启。

首次启动树莓派时，您需要连接显示器和键盘。首先应该做的就是连接到 Wi-Fi 网络并启用 SSH。

为此，您可以参考以下指南：https://www.raspberrypi.com/documentation/computers/configuration.html#setting-up-wifi

然后，您可以使用 SSH 连接到树莓派，而无需插入显示器和键盘。

### 更新系统并安装必要的软件

> 这里可以看看有没有国内的树莓派源。译者的网络环境还行就没换源。

```bash
sudo apt update
sudo apt upgrade
sudo apt install git
sudo apt install python3-pip
sudo apt install python3-virtualenvwrapper
```

将以下内容添加到 `.bashrc` 的末尾：

```bash
export WORKON_HOME=$HOME/.virtualenvs
export PROJECT_HOME=$HOME/Devel
source /usr/share/virtualenvwrapper/virtualenvwrapper.sh
```

### 启用 I2C

`sudo raspi-config` -> `Interface Options` -> `I2C`


### 设置 usb serial

```bash
cd  /etc/udev/rules.d/
sudo touch 99-usb-serial.rules
sudo nano 99-usb-serial.rules
# 将以下行复制到文件中
SUBSYSTEM=="usb-serial", DRIVER=="ftdi_sio", ATTR{latency_timer}="1"
```

### 设置用于电机控制板的 udev 规则

TODO


### 通过蓝牙设置 Xbox One 控制器

> 译者注：大家应该都玩游戏的吧。。。不玩的可以借机买个手柄玩？ ：）

> 如果没有蓝牙手柄，你可能需要改xboxcontroller模块，比较复杂。不过能改这个的。。。也不需要看我的翻译了吧。。。

打开您的 Xbox One 控制器，并通过长按控制器顶部的同步按钮将其置于配对模式。

在树莓派上运行以下命令：

```bash
bluetoothctl
scan on
```

等待控制器出现在列表中，然后运行：

```bash
pair <controller_mac_address>
trust <controller_mac_address>
connect <controller_mac_address>
```

控制器上的 LED 应该停止闪烁并保持常亮。

可以通过运行以下命令测试是否正常工作：

```bash
python3 scripts/test_xbox_controller.py
```

## 扬声器接线和配置

> 译者目前并没有做这个东西，搞好了再写

请遵循以下教程：

> 目前为止，当他们要求激活 `/dev/zero` 时，请勿激活。

https://learn.adafruit.com/adafruit-max98357-i2s-class-d-mono-amp?view=all


## 安装运行时

### 创建虚拟环境并激活它

```bash
mkvirtualenv -p python3 open-duck-mini-runtime
workon open-duck-mini-runtime
```

在树莓派上克隆此存储库，进入存储库后执行以下命令：

> 译者注：这一段很吃网络。国内的网可能不太行了捏。所以如果网不行又安装失败，可以试试我们出的镜像，装好依赖了的捏

```bash
git clone https://github.com/apirrone/Open_Duck_Mini_Runtime
cd Open_Duck_Mini_Runtime
git checkout v2
pip install -e .
```


## 测试 IMU

```bash
cd scripts/
python imu_test.py
```

## 查找关节偏移量

此脚本将引导您找到机器人的关节偏移量，您可以将这些偏移量写入 `rustypot_position_hwi.py` 中的 `self.joints_offsets`。

> 未来我们将直接将偏移量刷入每个电机的 EEPROM 中，因此此过程将不再必要。

> 译者注：上面这个是作者说的。但在他实现之前还是搞一下。

```bash
cd scripts/
python find_soft_offsets.py
```
