# 使用openwrt路由（以红米AC2100为例）防校园网多设备检测

##### 校园网主要检测主要目前已知的（可能）存在的有：

基于 IPv4 数据包包头内的 TTL 字段的检测（固定TTL）<br>基于 HTTP 数据包请求头内的 User-Agent 字段的检测(UA2F)<br>DPI (Deep Packet Inspection) 深度包检测技术）（不常用）<br>基于 IPv4 数据包包头内的 Identification 字段的检测（rkp-ipid 设置 IPID）<br>基于网络协议栈时钟偏移的检测技术（防时钟偏移检测）<br>Flash Cookie 检测技术（iptables 拒绝 AC 进行 Flash 检测 不常用）

## 注意

1. **不要用 root 用户进行编译**
2. 国内用户编译前最好准备好梯子
3. 默认登陆IP 192.168.1.1 密码 password

## 一、编译命令

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

3. 下载源代码，更新 feeds 

   ```
   git clone https://github.com/coolsnowwolf/lede
   cd lede
   ./scripts/feeds update -a
   ./scripts/feeds install -a
   ```

4. 加入UA2F模块和RKP-IPID模块

   ```
   git clone https://github.com/EOYOHOO/UA2F.git package/UA2F
   git clone https://github.com/EOYOHOO/rkp-ipid.git package/rkp-ipid
   ```

5. 配置

   ```
   make menuconfig
   ```

   以红米AC2100为例，按下图选择路由器（根据自己的路由器设置前三项）

   ![选择设备](https://github.com/wzt0921/Openwrt-FZU-Network/blob/main/images/%E9%80%89%E6%8B%A9%E8%AE%BE%E5%A4%87.PNG)

   接着，选择需要安装的模块

   ```
   # 勾选上ua2f
   # network->Routing and Redirection->ua2f
   # 勾选上ipid
   # kernel-modules->Other modules->kmod-rkp-ipid
   #在openwrt目录下
   nano .config
   #加入一句
   CONFIG_NETFILTER_NETLINK_GLUE_CT=y
   # 选上模块
   make menuconfig
   # network->firewall->iptables-mod-filter
   # network->firewall->iptables-mod-ipopt
   # network->firewall->iptables-mod-u32
   # network->firewall->iptables-mod-conntrack-extra
   # kernel modules->Netfilter Extensions->kmod-ipt-u32
   # kernel modules->Netfilter Extensions->kmod-ipt-ipopt
   #若无其他要求，即可准备编译
   ```

6. 下载 dl 库，编译固件 ，预计时间较长（-j 后面是线程数，第一次编译推荐用单线程）

   ```
   make download -j8
   make V=s -j1
   ```

   若为二次编译

   ```
   make V=s -j$(nproc)
   ```

7. 根据路由器型号自行刷入编译好的固件，这里不提供具体教程

8. 防检测配置

   （1）进入 OpenWRT 系统设置, 勾选 Enable NTP client（启用 NTP 客户端）和 Provide NTP server（作为 NTP 服务器提供服务）。NTP server candidates（候选 NTP 服务器）四个框框分别填写 ntp1.aliyun.com、time1.cloud.tencent.com、stdtime.gov.hk 、pool.ntp.org，如下图

   ![NTP设置](https://github.com/wzt0921/Openwrt-FZU-Network/blob/main/images/NTP%E8%AE%BE%E7%BD%AE.PNG)

   （2）防火墙添加以下自定义规则

   ```
   iptables -t nat -A PREROUTING -p udp --dport 53 -j REDIRECT --to-ports 53
   
   iptables -t nat -A PREROUTING -p tcp --dport 53 -j REDIRECT --to-ports 53
   
   # 通过 rkp-ipid 设置 IPID
   
   # 若没有加入rkp-ipid模块，此部分不需要加入
   
   iptables -t mangle -N IPID_MOD
   
   iptables -t mangle -A FORWARD -j IPID_MOD
   
   iptables -t mangle -A OUTPUT -j IPID_MOD
   
   iptables -t mangle -A IPID_MOD -d 0.0.0.0/8 -j RETURN
   
   iptables -t mangle -A IPID_MOD -d 127.0.0.0/8 -j RETURN
   
   #由于本校局域网是A类网，所以我将这一条注释掉了，具体要不要注释结合你所在的校园网
   
   # iptables -t mangle -A IPID_MOD -d 10.0.0.0/8 -j RETURN
   
   iptables -t mangle -A IPID_MOD -d 172.16.0.0/12 -j RETURN
   
   iptables -t mangle -A IPID_MOD -d 192.168.0.0/16 -j RETURN
   
   iptables -t mangle -A IPID_MOD -d 255.0.0.0/8 -j RETURN
   
   iptables -t mangle -A IPID_MOD -j MARK --set-xmark 0x10/0x10
   
   # 防时钟偏移检测
   
   iptables -t nat -N ntp_force_local
   
   iptables -t nat -I PREROUTING -p udp --dport 123 -j ntp_force_local
   
   iptables -t nat -A ntp_force_local -d 0.0.0.0/8 -j RETURN
   
   iptables -t nat -A ntp_force_local -d 127.0.0.0/8 -j RETURN
   
   iptables -t nat -A ntp_force_local -d 192.168.0.0/16 -j RETURN
   
   iptables -t nat -A ntp_force_local -s 192.168.0.0/16 -j DNAT --to-destination 192.168.1.1
   
   # 通过 iptables 修改 TTL 值
   
   iptables -t mangle -A POSTROUTING -j TTL --ttl-set 64
   
   # iptables 拒绝 AC 进行 Flash 检测
   
   iptables -I FORWARD -p tcp --sport 80 --tcp-flags ACK ACK -m string --algo bm --string " src=\"http://1.1.1." -j DROP
   ```

   （3）UA2F配置

   ```
   # 开机自启
   uci set ua2f.enabled.enabled=1
   # 自动配置防火墙（默认开启）（建议开启）
   uci set ua2f.firewall.handle_fw=1
   uci set ua2f.firewall.handle_tls=1
   uci set ua2f.firewall.handle_mmtls=1
   uci set ua2f.firewall.handle_intranet=1
   # 保存配置
   uci commit ua2f
   
   操作：# 开机自启
   service ua2f enable
   # 启动ua2f
   service ua2f start
   
   # 手动关闭ua2f
   service ua2f stop		
   ```

到此就可以进行检测了

[UA检测](http://ua.233996.xyz/)

如果你的真实UA是(服务器获取的UA)显示：

```
FFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFF
```

说明配置正确

## 注意：

1、翻墙可能会使UA2F失效

2、UA2F只修改HTTP流量，HTTPS是加密的，因此无需修改

3、由于微信mmtls协议的影响，会可能会导致微信图片无法发送等问题，此问题可执行

```
uci set ua2f.firewall.handle_mmtls=0
uci commit ua2f 
```

## 致谢

**openwrt源码来源于：**[Lean](https://github.com/coolsnowwolf/lede)<br>**参考来源于：**[EOYOHOO/Campus-network](https://github.com/EOYOHOO/Campus-network)<br>**ua2f来源于：** [Zxilly/UA2F](https://github.com/Zxilly/UA2F)<br>**修改 IPID 来源于：** [CHN-beta/rkp-ipid](https://github.com/CHN-beta/rkp-ipid)
