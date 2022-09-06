# Openwrt-FZU-Network
校园网主要检测主要目前已知的（可能）存在的有：

基于 IPv4 数据包包头内的 TTL 字段的检测（固定TTL）
基于 HTTP 数据包请求头内的 User-Agent 字段的检测(UA2F)
DPI (Deep Packet Inspection) 深度包检测技术）（不常用）
基于 IPv4 数据包包头内的 Identification 字段的检测（rkp-ipid 设置 IPID）
基于网络协议栈时钟偏移的检测技术（防时钟偏移检测）
Flash Cookie 检测技术（iptables 拒绝 AC 进行 Flash 检测 不常用）
