---
title: "WireGuard_vpn"
date: 2023-10-24T09:45:06+08:00
categories:
- 收件箱
- 工作记录
tags:
- 工作
- 记录
keywords:
- 案例
#thumbnailImage: //example.com/image.jpg
---

<!--more-->
wireguard初衷：旨在比 IPsec更快、更简单、更精简，可在嵌入式接口和超级计算机上运行的vpn。
wireguard特点：
1、配置简单，像交换ssh密钥一样，双方都有对方的公钥，无需管理连接、关注状态、管理守护进程或担心幕后情况。
2、加密可靠，支持 Noise protocol framework, Curve25519, ChaCha20, Poly1305, BLAKE2, SipHash24, HKDF协议
3、攻击面小，代码量少，漏洞相对易检查
4、高性能，在linux5.6版本中被添加到内核中，使用udp封装ip数据包。
核心概念是cryptokey Routing，将公钥与隧道内允许的隧道ip地址列表相关联。每个网络接口都有一个私钥和一个对等点列表。公钥简短，可以通过带外传输，类似于将ssh公钥发送给用户来访问linux。



ip变动：
运行单个对等方初始端点变动，通过检查正确验证数据的来源发现对等放端点。当对端发生变化，都会将加密数据发送给最近的ip端点。两端都有完整的ip漫游。
ipsec 分层合理，但过于臃肿
openvpn位于用户空间，性能差，需要在内核和用户空间之间多次复制数据包，需要一个长期存在的守护进程。
openvpn tls带来了状态机，以及源ip与公钥之间的关联不太清晰。



公钥只有32字节长，可以使用base64编码为44个字符，
底层基元发现漏洞，所有端点都需要更新。
可封装v4-in-v6和v6-in-v4
使用虚拟接口，而不是转换层
接口本身有一个私钥和一个侦听的udp端口。以及一个名单。没有过对等连点都由其公钥标识。每个都有一个允许的ip列表。
用wireguard数据包外部源ip作为标识对等体的远程端点
中间人可以修改未经省份验证的外部源ip，但不能解密和修改载荷，仅仅相当于拒绝服务攻击。
功能和和函数：
DH(PRIVATE KEY,PUBLIC KEY) Curve25519 公钥和私钥的点乘法，返回32字节输出。
DH-Generate()  随机生成一个Curve25519私钥并推导出对应公钥，返回32字节的值。   
AEAD(KEY,COUNTER,PLAIN TEXT,AUTH TEXT) ChaCha20Poly1305 AEAD, as specified in RFC7539，其随机数由32位零组成，后跟计数器的64位低字节序。
XAEAD(KEY,NONCE, PLAIN TEXT, AUTH TEXT) XChaCha20Poly1305 AEAD, 和一个一次性的24byte随机数，实例使用HChaCha20和ChaCha20Poly1305.
HASH(input) blake2s(input, 32), 返回32字节输出。
MAC(KEY, INPUT)KEYED-BLAKE2S(KEY,INPUT, 16),blake2s哈希的键控mac变体函数，返回16个字节输出。
HMAC(KEY, INPUT) HMAC-BLAKE2S(KEY,INPUT,32),普通blake2s散列函数用于HMAC构造，返回32字节输出。
协议概览：
大多数情况握手为第一种情况，如果一个对等点负载不足，则cookie将消息添加的握手中，以防止拒绝服务攻击。

有时会出现乱序，使用滑动窗口来跟踪接收到的消息计数器，当bitshifts时使用更大的bitmap，多发生在多核系统上。
负载不足时，cookie回复消息
从用户的角度，几乎无状态