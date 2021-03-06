# 网络层

---

## 1. IP 地址

合法的 IP 地址：(1 ~ 255).(0 ~ 255).(0 ~ 255).(0 ~ 255)，其中各段为十进制整数，所以如 01、00 等形式都是非法地址。

**CIDR**：与被分类后固定格式的 IP 地址不同，CIDR 将 IP 地址分为网络号与主机号，在给出 IP 地址的同时也会给出网路号的位数，如 192.168.0.1 / 24 中，192.168.0 代表了 32 位 IP 地址的前 24 位，表示网络号，1 表示主机号。

**子网掩码**：

* 网络号全1，主机号全0
* 与IP地址相与得到的即为网络号
* ABC类地址默认子网掩码：255.0.0.0/255.25.0.0/255.255.255.0
* 计算：如子网掩码为255.255.255.240（后八位为 11110000）时，比C类默认多使用了4位网络位，可提供16个子网，主机位为4位，除去全0时的子网掩码与全1时的广播地址，主机数量为14

## 2. ICMP

互联网控制报文协议，对网络状况起侦察的作用。ICMP协议有多种报文类型，分别对应不同的场景与应用

**查询报文类型**：在执行ping命令时，源主机会构建ICMP请求数据包，ICMP协议将数据包交给IP层发送至目标主机。如果源主机在规定时间内未收到目标主机的应答数据包，则认为目标主机不可达；如果受到应答数据包，收到时间与发出时间的差值即为延迟时间

**差错报文类型**：traceroute命令用于追踪数据包在网络上传输时的全部路径，它会利用ICMP的规则，故意制造一些差生错误的场景，通常是将TTL设置为一个特殊值。开始时发送TTL为1的包，遇到第一个路由，TTL减1变为0，返回一个超时的ICMP包，则第二次将TTL值加1，重复上述过程。由于发送的为UDP包，并且选择了一个不可能的值端口号值，所以当数据包到达目标主机后，目标主机在IP层会返回一个端口不可达的ICMP包，源主机收到此数据包后即得知目标主机可达，并且拿到了经过的所有IP（路由器可以不回复超时类型的ICMP报文）

## 3. 网关

当源主机与目标主机不在同一个网段时（CIDR/子网掩码），源主机需要先将数据包发往同网段的Gateway，与局域网内通信相同，使用ARP协议

网关往往是一个路由器，路由器属于三层设备，也就是说它会将MAC头与IP头都取下，根据内部的内容进行下一步的转发

需要明确的是，简单的认为，路由器的每一个网口就是其所在局域网的网关，此网口与其连接的局域网的网段相同，数据包在路由器内部根据路由器的算法进行跨网段的转发，路由器会指定数据包的出口网关IP与下一跳的入口网关IP

**网关类型**：

* **转发网关**
  * 数据包中源IP与目标IP在转发过程中始终不变。在此种转发过程中，主机与路由器、路由器与路由器之间形成局域网，路由器在使用ARP协议获得下一跳入口网关的MAC地址后发送包
  * 可以看出每到一个新的局域网，由于ARP协议的原因，MAC地址都会变，但IP头中不会保存任何网关的IP地址，否则会丢失最终目标IP，而网关的IP地址被以MAC地址的方式存储

> **为什么有了MAC地址还需要IP地址**
> 一方面由于上述的转发问题，在转发过程中MAC地址在不断的变化，需要与IP地址配合使用才能保证发送到目标主机；另外一方面MAC地址不呈现聚类，不便于管理，而IP地址能够汇聚网段，便于转发

* **NAT网关**
  * 由于公网IP地址稀缺，所以使用了一种网络地址转换（Network Address Translation）的方式，多个内网IP地址公用若干个网关IP地址，访问内网主机时需要先经过NAT网关做地址转换
  * NAT网关的转发过程与普通网关转发类似，区别在于只有当数据包分别处于源主机与目标主机所在的局域网时，各自的IP才能使用内网IP，在中间的各个局域网内，源IP与目标IP均表示为主机的公网IP
