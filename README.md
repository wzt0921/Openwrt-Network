# 使用openwrt路由（例红米AC2100）防校园网多设备检测

##### 校园网主要检测主要目前已知的（可能）存在的有：

基于 IPv4 数据包包头内的 TTL 字段的检测（固定TTL）
基于 HTTP 数据包请求头内的 User-Agent 字段的检测(UA2F)

DPI (Deep Packet Inspection) 深度包检测技术）（不常用）
基于 IPv4 数据包包头内的 Identification 字段的检测（rkp-ipid 设置 IPID）
基于网络协议栈时钟偏移的检测技术（防时钟偏移检测）
Flash Cookie 检测技术（iptables 拒绝 AC 进行 Flash 检测 不常用）

## 注意

1. **不要用 root 用户进行编译**
2. 国内用户编译前最好准备好梯子
3. 默认登陆IP 192.168.1.1 密码 password
