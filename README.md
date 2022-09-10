# 使用openwrt路由（例红米AC2100）防校园网多设备检测

##### 校园网主要检测主要目前已知的（可能）存在的有：

基于 IPv4 数据包包头内的 TTL 字段的检测（固定TTL）<br>基于 HTTP 数据包请求头内的 User-Agent 字段的检测(UA2F)<br>DPI (Deep Packet Inspection) 深度包检测技术）（不常用）<br>基于 IPv4 数据包包头内的 Identification 字段的检测（rkp-ipid 设置 IPID）<br>基于网络协议栈时钟偏移的检测技术（防时钟偏移检测）<br>Flash Cookie 检测技术（iptables 拒绝 AC 进行 Flash 检测 不常用）

## 注意

1. **不要用 root 用户进行编译**
2. 国内用户编译前最好准备好梯子
3. 默认登陆IP 192.168.1.1 密码 password

## 一、准备编译环境

1. 首先装好 Linux 系统，推荐 Debian 11 或 Ubuntu LTS
2. 安装编译依赖

```
sudo apt update -y
sudo apt full-upgrade -y
sudo apt install -y ack antlr3 asciidoc autoconf automake autopoint binutils bison build-essential \
bzip2 ccache cmake cpio curl device-tree-compiler fastjar flex gawk gettext gcc-multilib g++-multilib \
git gperf haveged help2man intltool libc6-dev-i386 libelf-dev libglib2.0-dev libgmp3-dev libltdl-dev \
libmpc-dev libmpfr-dev libncurses5-dev libncursesw5-dev libreadline-dev libssl-dev libtool lrzsz \
mkisofs msmtp nano ninja-build p7zip p7zip-full patch pkgconf python2.7 python3 python3-pip libpython3-dev qemu-utils \
rsync scons squashfs-tools subversion swig texinfo uglifyjs upx-ucl unzip vim wget xmlto xxd zlib1g-dev
```

