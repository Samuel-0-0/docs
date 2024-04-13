## 操作视频

<iframe src="//player.bilibili.com/player.html?aid=861634977&bvid=BV1pV4y1w7Sw&cid=935534286&p=1" scrolling="no" border="0" frameborder="no" framespacing="0" allowfullscreen="true"> </iframe>

## 文档记录

2023.2.16更新，以下步骤中的部分方法经过更新，可能和视频有所出入，按文本为准。

### 步骤一

Debian镜像下载地址
https://cdimage.debian.org/cdimage/unofficial/non-free/cd-including-firmware/current/multi-arch/iso-cd/

制作好系统的安装U盘后，下载
http://mirrors.ustc.edu.cn/debian/pool/non-free/f/firmware-nonfree/firmware-realtek_20210315-3_all.deb

放到安装U盘的firmware目录下。


插上U盘和网线，引导进安装初始界面，选择第二个，install，根据提示安装系统。

最后一步不选择安装桌面，只选择SSH server和标准系统工具。


重启后在设备上执行命令

```
ip address
```
查看IP地址。


### 步骤二

使用SSH工具，用root用户远程登陆设备。

执行命令

```
nano /etc/network/interfaces
```
如果使用WiFi连接，复制下面的内容增加到最后面，使用网线的不需要操作

```
auto wlan0
iface wlan0 inet dhcp
  wpa-ssid 777
  wps-psk 1122334455
  #WiFi隐藏时取消下面这行前面的#
  #wpa-scan-ssid 1
```

按键盘 CTRL+O，回车，CTRL+X



完整复制下面的命令到SSH客户端执行，替换源：

```
cat > /etc/apt/sources.list << EOF
#清华大学镜像站
deb https://mirrors.tuna.tsinghua.edu.cn/debian/ bullseye main contrib non-free
deb-src https://mirrors.tuna.tsinghua.edu.cn/debian/ bullseye main contrib non-free
deb https://mirrors.tuna.tsinghua.edu.cn/debian/ bullseye-updates main contrib non-free
deb-src https://mirrors.tuna.tsinghua.edu.cn/debian/ bullseye-updates main contrib non-free
deb https://mirrors.tuna.tsinghua.edu.cn/debian/ bullseye-backports main contrib non-free
deb-src https://mirrors.tuna.tsinghua.edu.cn/debian/ bullseye-backports main contrib non-free
deb https://mirrors.tuna.tsinghua.edu.cn/debian-security bullseye-security main contrib non-free
deb-src https://mirrors.tuna.tsinghua.edu.cn/debian-security bullseye-security main contrib non-free
EOF
```

分别执行下面2行命令，安装依赖程序

```
apt-get update
```

```
apt-get install firmware-linux-free firmware-linux-nonfree firmware-misc-nonfree firmware-myricom firmware-netronome firmware-netxen firmware-qcom-soc firmware-qlogic firmware-realtek firmware-samsung firmware-siano firmware-sof-signed firmware-ti-connectivity firmware-zd1211 hdmi2usb-fx2-firmware intel-microcode  amd64-microcode atmel-firmware bluez-firmware dahdi-firmware-nonfree firmware-amd-graphics firmware-ath9k-htc firmware-atheros firmware-b43legacy-installer firmware-bnx2 firmware-bnx2x firmware-brcm80211 firmware-cavium firmware-intel-sound firmware-intelwimax firmware-ipw2x00 firmware-ivtv firmware-iwlwifi firmware-libertas fonts-wqy-zenhei git sudo unzip net-tools wireless-tools -y
```

为用户samuel添加sudo权限，samuel替换为你自己的新用户名

```
usermod -a -G sudo samuel
```

执行下面命令，将用户添加到dialout用户组和tty用户组

```
gpasswd --add samuel dialout && gpasswd --add samuel tty && gpasswd --add samuel input
```

重启系统

```
reboot
```

### 步骤三

在SSH客户端使用自己新建的用户远程登陆


添加国内PIP源，先执行命令

```
mkdir ~/.pip
```

然后复制下面全部执行

```
cat > ~/.pip/pip.conf << EOF
[global]
index-url = http://mirrors.aliyun.com/pypi/simple
[install]
trusted-host=mirrors.aliyun.com
EOF
```

安装klipper依赖

```
sudo apt install python3-virtualenv python3-dev libffi-dev build-essential libncurses-dev libusb-dev avrdude gcc-avr binutils-avr avr-libc stm32flash dfu-util libnewlib-arm-none-eabi gcc-arm-none-eabi binutils-arm-none-eabi libusb-1.0-0 
```

git拉取klipper

```
cd ~ && git clone https://github.com/KevinOConnor/klipper
```

安装klipper

```
cd ~ && virtualenv -p python3 ./klippy-env && ./klippy-env/bin/pip install -r ./klipper/scripts/klippy-requirements.txt
```

添加klipper服务

```
sudo /bin/sh -c  "cat > /etc/systemd/system/klipper.service" << EOF
#Systemd Klipper Service
[Unit]
Description=Starts Klipper and provides klippy Unix Domain Socket API
Documentation=https://www.klipper3d.org/
After=network.target
Before=moonraker.service
Wants=udev.target

[Install]
Alias=klippy
WantedBy=multi-user.target

[Service]
Type=simple
User=$USER
RemainAfterExit=yes
ExecStart= /home/$USER/klippy-env/bin/python /home/$USER/klipper/klippy/klippy.py /home/$USER/printer_data/config/printer.cfg -l /home/$USER/printer_data/logs/klippy.log -a /tmp/klippy_uds
Restart=always
RestartSec=10
EOF
```

启用klipper服务

```
sudo systemctl enable klipper.service
```

拉取moonraker

```
cd ~ && git clone https://github.com/Arksine/moonraker.git
```

安装moonraker

```
cd ~/moonraker/scripts && ./install-moonraker.sh
```

修复policykit

```
cd ~ && ./moonraker/scripts/set-policykit-rules.sh
```

启动klipper服务

```
sudo systemctl start klipper
```

启动moonraker服务

```
sudo systemctl start moonraker
```

### 步骤四

安装nginx

```
sudo apt install nginx
```

完整复制下面内容并复制到SSH客户端执行：

```
sudo /bin/sh -c  "cat > /etc/nginx/conf.d/upstreams.conf" << EOF
# /etc/nginx/conf.d/upstreams.conf
upstream apiserver {
    ip_hash;
    server 127.0.0.1:7125;
}
upstream mjpgstreamer1 {
    ip_hash;
    server 127.0.0.1:8080;
}
upstream mjpgstreamer2 {
    ip_hash;
    server 127.0.0.1:8081;
}
upstream mjpgstreamer3 {
    ip_hash;
    server 127.0.0.1:8082;
}
upstream mjpgstreamer4 {
    ip_hash;
    server 127.0.0.1:8083;
}
EOF
```

完整复制下面内容并复制到SSH客户端执行：

```
sudo /bin/sh -c  "cat > /etc/nginx/conf.d/common_vars.conf" << EOF
# /etc/nginx/conf.d/common_vars.conf
map \$http_upgrade \$connection_upgrade {
    default upgrade;
    '' close;
}
EOF
```

完整复制下面内容并复制到SSH客户端执行：

```
sudo /bin/sh -c  "cat > /etc/nginx/sites-available/mainsail" << EOF
# /etc/nginx/sites-available/mainsail
server {
    listen 80 default_server;
    access_log /var/log/nginx/mainsail-access.log;
    error_log /var/log/nginx/mainsail-error.log;
    # disable this section on smaller hardware like a pi zero
    gzip on;
    gzip_vary on;
    gzip_proxied any;
    gzip_proxied expired no-cache no-store private auth;
    gzip_comp_level 4;
    gzip_buffers 16 8k;
    gzip_http_version 1.1;
    gzip_types text/plain text/css text/xml text/javascript application/javascript application/x-javascript application/json application/xml;
    # web_path from mainsail static files
    root /home/samuel/mainsail;
    index index.html;
    server_name _;
    # disable max upload size checks
    client_max_body_size 0;
    # disable proxy request buffering
    proxy_request_buffering off;

    location / {
        try_files \$uri \$uri/ /index.html;
    }

    location = /index.html {
        add_header Cache-Control "no-store, no-cache, must-revalidate";
    }

    location /websocket {
        proxy_pass http://apiserver/websocket;
        proxy_http_version 1.1;
        proxy_set_header Upgrade \$http_upgrade;
        proxy_set_header Connection \$connection_upgrade;
        proxy_set_header Host \$http_host;
        proxy_set_header X-Real-IP \$remote_addr;
        proxy_set_header X-Forwarded-For \$proxy_add_x_forwarded_for;
        proxy_read_timeout 86400;
    }

    location ~ ^/(printer|api|access|machine|server)/ {
        proxy_pass http://apiserver\$request_uri;
        proxy_set_header Host \$http_host;
        proxy_set_header X-Real-IP \$remote_addr;
        proxy_set_header X-Forwarded-For \$proxy_add_x_forwarded_for;
        proxy_set_header X-Scheme \$scheme;
    }

    location /webcam/ {
        proxy_pass http://mjpgstreamer1/;
    }

    location /webcam2/ {
        proxy_pass http://mjpgstreamer2/;
    }

    location /webcam3/ {
        proxy_pass http://mjpgstreamer3/;
    }

    location /webcam4/ {
        proxy_pass http://mjpgstreamer4/;
    }
}
EOF
```

完成配置更新

```
mkdir ~/mainsail && sudo rm /etc/nginx/sites-enabled/default && sudo ln -s /etc/nginx/sites-available/mainsail /etc/nginx/sites-enabled/ && sudo systemctl restart nginx
```

下载mainsail文件

```
cd ~/mainsail && wget -q -O mainsail.zip https://github.com/mainsail-crew/mainsail/releases/latest/download/mainsail.zip && unzip mainsail.zip && rm mainsail.zip
```

### 步骤五

安装KlipperScreen

```
cd ~ && git clone https://github.com/jordanruthe/KlipperScreen.git && cd ~/KlipperScreen && ./scripts/KlipperScreen-install.sh
```

提示需要重启，确定重启

### 问题处理

重启后屏幕不显示KlipperScreen界面，执行（samuel为用户名，自行修改）

```
cat /home/samuel/.local/share/xorg/Xorg.0.log
```

如果日志里有类似错误

```
(EE) xf86OpenConsole: Cannot open virtual console 2 (Permission denied)
```

执行

```
sudo nano /etc/X11/Xwrapper.config
```

在末尾添加一行

```
needs_root_rights=yes
```

然后执行

```
sudo dpkg-reconfigure xserver-xorg-legacy && sudo reboot
```

重启后，屏幕显示转了90度，需要永久化屏幕旋转

```
sudo /bin/sh -c  "cat > /usr/share/X11/xorg.conf.d/90-monitor.conf" << EOF
Section "Monitor"
    Identifier "DSI-1"
    Option "Rotate" "left"
    # Valid rotation options are normal,inverted,left,right
    #Option "PreferredMode" "1920x1080"
    # May be necesary if you are not getting your prefered resolution.
EndSection
EOF
```

重启KlipperScreen服务

```
sudo service KlipperScreen restart
```

如果试了还是不翻转，将前面的Identifier "DSI-1"修改为Identifier "DPI-1"后，再重复上面2步操作。


此时触摸还未旋转，执行下方命令测试触摸旋转

```
DISPLAY=:0 xinput set-prop "pointer:Goodix Capacitive TouchScreen" 'Coordinate Transformation Matrix' 0 -1 1 1 0 0 0 0 1
```

测试没有问题，修改成永久化触摸旋转

```
sudo nano /usr/share/X11/xorg.conf.d/40-libinput.conf
```

在有touchscreen的里面添加一行 Option "TransformationMatrix" "0 -1 1 1 0 0 0 0 1"

修改后如下：

```
Section "InputClass"
        Identifier "libinput touchscreen catchall"
        MatchIsTouchscreen "on"
        MatchDevicePath "/dev/input/event*"
        Driver "libinput"
        Option "TransformationMatrix" "0 -1 1 1 0 0 0 0 1"
EndSection
```

