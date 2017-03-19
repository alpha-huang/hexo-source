title: 路由器设置常用命令
author: alpha
tags:
  - 路由器
  - 网络安全
  - 密码破解
  - 笔记
categories:
  - 笔记
date: 2014-9-19 20:18:34
---
 操作环境为企业级路由器或模拟器（例如：Cisco Packet Tracer）
<!--more-->

### 基础命令

```
>用户模式	en进入特权模式
#特权模式
conf t进入配置模式
show run查看当前配置
write保存
ping 192.168.1.1测试通信
(config)#配置模式
hostname R1		修改设备名称
enable password 123		设置明文密码
enable secret 123		设置密文密码
int fa0/0		进入接口模式
line console 0		进入配置线路
(config-if)#接口模式
ip add 192.168.1.1 255.255.255.0配置接口上的ip和掩码
no sh		打开接口
(config-line)#线路模式
exec-timeout 1 30		设置超时1分30秒自动退出
```

### 破解密码
1.用记事本写好命令复制粘贴

2.流程
①　选设备，连线
②　规划ip地址，并标注
③　配置ip地址，测试直连
3.一些命令
静态路由
配置模式：
```
ip route 目标网络号 掩码 下一跳（下一个路由的进接口）
ip route 192.168.1.0 255.255.255.0 192.168.2.1
```
查看静态路由show ip route（特权模式）
查错方法：traceroute 目标地址
4.破密码
①　重启一半时按ctrl+c打断正常启动，进入工程模式
②　工程模式下confreg 0x2142（2142意味启动时跳过加载配置，2102是默认值，意味加载配置文件）
③　reset重启
④　启动后无任何配置，进入特权模式
⑤　手动加载配置copy start run
⑥　进入配置模式修改密码
⑦　恢复启动参数config-register 0x2102
⑧　保存
5.配置动态路由RIP
在路由器上运行RIP，可使得路由器自动发送已有路由给相邻的路由器从而全网联通
```
router rip		运行RIP
network 12.0.0.0		通告本地的直连网段
show ip route
eigrp 增强型内部网关协议，思科私有
router eigrp 1		1代表EIGRP的AS号
netword 12.0.0.0
```
6.多线路选择
信任度
浮动静态路由

### 配置简单的网络环境
1.交换网络
①　交换机不识别ip，路由器可以识别mac
②　交换机又叫网桥（透明网桥）
③　接交换机时接成树形而不是线形
2.DHCP动态主机配置协议
CS模型
client-server
PC		服务器/GW
```
ip dhcp excluded-address 192.168.1.1 192.168.1.10		保留不分配的ip
ip dhcp pool 1		创建地址池
network 192.168.1.0 255.255.255.0		分配网段
default-router 192.168.1.1		网关ip
*DHCP服务器应该与客户端在同一个局域网
```
3.vlan虚拟局域网
创建vlan
```
vlan 10
name cisco
vlan 20
name huawei
```
划vlan
```
int fa0/1
sw mode access		设置为接入端口
sw access vlan 10			接入vlan10
int range fa0/3-4		批量配置
sw mode access
sw access vlan 20
```
4.在交换机之间配置trunk（干道）承载多个vlan信息
```
int fa0/3
sw trunk en dot1q		三层交换机需要选择打标方式
sw mode trunk		选择干道模式
```
为了保证vlan间通信，需要添加三层交换机
①　配置干道，创建trunk
②　创建与vlan相关的网关接口
```
int vlan 10
ip add 192.168.10.254 255.255.255.0
```
③　打开路由功能
```
ip routing
*不要忘记pc需要设置Gw
```
5.
干道		show run
vlan		show vlan brief
划vlan
vlan网关		sh ip int brief
DHCP
ping


### 配置常用的家庭网络
1.家庭网络
先配置交换机再配置路由器
(1)VTP
vlan干道协议，同步vlan信息
```
vtp mode server/client
vtp domain cisco
*必须先配置trunk再配置VTP
```
(2)三层交换机
```
int fa0/1
no sw		将接口改为路由
ip add 192.168.1.1 255.255.255.0
ip routing		打开三层交换机的路由功能
router eigrp 1
no auto		取消地址汇总
```
2.ACL访问控制列表
作用是限制流量
(1)a.定义ACL规则
```
access-list 1 deny host 192.168.10.1
access 1 permit any		拒绝某台主机
access-list 2 deny 192.168.10.0 0.0.0.255
access 2 permit any		拒绝某个网段
*默认情况下会拒绝所有
access-list 3 permit 192.168.10.0 0.0.0.255
```
b.在接口上调用
```
int fa0/0
ip access-group 1 in/out		注意调用的方向
show ip access-list		查看ACL
```
(2)先写范围最小的才写大的，否则执行前面的之后就不会再执行后面的
```
access-list 4 permit host 192.168.10.10
access-list 4 deny 192.168.10.0 0.0.0.255
access-list 4 permit any
```
(3)扩展ACL
```
access-list 100 deny ip 192.168.20.0 0.0.0.255(源) host 23.1.1.2(目的)
access-list 100 permit ip any any		允许any到any
```
(4)禁止除http外的其他服务
```
access-list 100 permit tcp 192.168.20.0 0.0.0.255 host 23.1.1.2 eq 80
access-list 100 deny ip 192.168.20.0 0.0.0.255 host 23.1.1.2
access-list 100 permit ip any any
```
### 真机实验
1.交换机破密码
①　拔电源，按下mode键，插上电源进入sw模式，松开mode键
②　初始化flash		Flash_init
③　修改配置文件名称
```
dir flash/？		显示所有文件
rename flash:config.text flash:config.old
```
④　重启		boot
⑤　进入特权模式，手动加载配置copy flash:config.old run
⑥　删除密码并保存
2.路由器破密码
①　工程模式下o/r 0x2142
②　重启 i

### 默认路由（缺省路由）
1.NAT
GW上配置：
```
ip route 0.0.0.0 0.0.0.0 100.1.1.2		默认路由，代替GW上所有去往公网的路由
access-list 1 permit any		ACL用来定义内网可以被转换的PC
ip nat inside source list 1 interface f0/1 overload		将内网IP地址转换为公网IP，f0/1是公网接口
ip nat inside source static 192.168.1.4 100.1.1.4		静态的地址转换，通常给内网服务器使用
int f0/0
ip nat inside		设置内网接口
int f0/1
ip nat outside		设置公网接口
```
2.http/ftp下载、p2p下载与离线下载
3.电信分配IP地址100.1.1.0/30实际上只有
- 100.1.1.0		网段
- 100.1.1.1		电信端
- 100.1.1.2		用户端
- 100.1.1.3		广播网

### 交换机备份
STP(spanning-tree protocol)生成树协议
选出某些阻塞接口不转发流量，防止形成环路
备分步骤
①　添加设备并连线
②　配置trunk
③　创建vlan
④　设置STP的根和次根
```
spanning-tree vlan 1-4094 priority 4096
```
⑤　测试

### 网关备份
HSRP热备份路由协议
①　在需要成为主网关的路由器上配置
```
interface fa0/0		进入内网接口
ip add 192.168.1.2 255.255.255.0		物理IP
standby 1 ip 192.168.1.254		虚拟IP
standby 1 prority 200		优先级
standby 1 preempt		抢占
```
②　备份网关上的配置
```
interface fa0/0		进入内网接口
ip add 192.168.1.1 255.255.255.0
standby 1 ip 192.168.1.254
```
端口安全
限制access接口下可以学习的mac地址数量，可以进行静态绑定
```
int fa0/1
sw mode access		设置access模式
sw port-security		开启端口安全
sw port-security mac-address 0010.11DA.2668		静态绑定单个mac地址
int fa0/2
sw mode access
sw port-security
sw port-security mac-address sticky		自动绑定
sw port-security maximum 2		最大绑定个数
*如果端口被安全措施关闭，则需要先shutdown，再no sh重新打开
```
### ZBF区域防火墙
```
policy-map type inspect 1		建立策略1
class type inspect class-default		针对所有流量
inspect		检测
zone security A		创建区域A
zone security B		创建区域B
zone-pair security DMZ source A destination B		创建区域策略
service-policy type inspect 1		调用策略
int fa0/0
ip add 192.168.1.1 255.255.255.0
zone-member security A		设置为A区域接口
int fa0/0
ip add 192.168.2.1 255.255.255.0
zone-member security B		设置为B区域接口
```
FRSW帧中继交换机
encapsulated

### DHCP中继

STP加速（经典生成树）
为了防止终端接入交换网络等待30s造成的DHCP失败，将连接终端的端口配置为快速端口
```
int range fa0/5-24
```
RST（快速生成树Rapid-ST）
减少生成树的收敛时间
```
spanning-tree mode rapid		开启RST，要在所有SW上配置
```
STP流量分担
PVST每vlan生成树
针对不同的vlan进行不同的STP选举，可以阻塞不同的端口，进行流量分担
sw1:
```
spa vlan 10 pri 0			vlan10主根
spa vlan 20 pri 4096		vlan20次根
```
sw2:
```
spa vlan 10 pri 4096		vlan10次根
spa vlan 20 pri 0			vlan20主根
show spa vlan 10/20
rapid-pvst
```
