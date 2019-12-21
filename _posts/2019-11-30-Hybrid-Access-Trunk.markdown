---
layout: post
title: "Huawei Switch" 
date:   2019-11-30 10:35:06
categories: 
---

<!-- more -->

## Hybrid Access Trunk

终端网卡只能识别标准的untagged报文,交换机根据802.1Q协议在源MAC地址以及目的MAC地址添加4bytes的vlan信息,即是tagged报文,普通的终端网卡不能识别.

Access网口只能属于1个VLAN;Trunk和Hybrid网口可以属于多个vlan,Trunk只允许缺省vlan(pvid)的报文发送时不打标签,Hybrid可允许多个.Trunk和Hybrid之间不能直接切换,需先修改为Access.

### 流入交换机,即接收报文

Access 收到一个封包,判断是否有vlan tag.如果没有,则打上pvid的tag标签,并进行交换转发;如果有,直接丢弃.

Trunk 收到一个封包,判断是否有vlan tag.如果没有,则打上pvid的tag标签,并进行交换转发;如果有,先判断Trunk网口是否允许该vlan的封包进入,允许则转发,否则丢弃.

Hybrid 收到一个封包,判断是否有vlan tag.如果没有,则打上pvid的tag标签,并进行交换转发;如果有,先判断Hybrid网口是否允许该vlan的封包进入,允许则转发,否则丢弃.

### 流出交换机,即发送报文

Access 将流入封包的vlan tag去掉,然后发送出去

Trunk 如果流入封包的vlan tag和该Trunk口的pvid相同,则将流入封包的vlan tag去掉,然后发送出去;如果不相同,直接发送.

Hybrid 判断该Hybrid口对哪些vlan是untag,哪些是tag,如果流入封包的vlan tag属于untag范围,则去掉vlan tag,然后发送出去;如果属于tag范围,直接发送.

## 802.1x认证

802.1x协议是一种基于接口(port)的网络接入控制协议。“基于接口的网络接入控制”是指，局域网中的接入层设备通过认证来控制用户对网络资源的访问。当采用基于接口的接入控制方式时，只要该接口下的第一个用户认证成功后，其它接入用户无须认证就可使用网络资源，但是当第一个用户下线后，其它用户也会被拒绝使用网络。可通过修改接口的最大认证用户数放宽限制，或扩展到基于MAC的接入控制方式，此时该接口下的所有接入用户均需要单独认证。

认证模式：

端口控制方式：
```
自动识别(auto)：表示端口初始状态为非授权状态，仅允许EAPOL报文收发，不允许用户访问网络资源；如果认证通过，则端口切换到授权状态，允许用户访问网络资源。这也是最常见的情况。
强制授权（authorized-force）：表示端口始终处于授权状态，允许用户不经认证授权即可访问网络资源。
强制非授权（unauthorized-force）：表示端口始终处于非授权状态，不允许用户进行认证。设备端不对通过该端口接入的客户端提供认证服务。
```

在huawei域配置完成后，用户进行接入认证时，以格式“user@huawei”输入用户名即可在huawei域下进行aaa认证。如果用户名中不携带域名或携带的域名不存在，用户将会在默认域进行认证。用户所属认证域是由接入设备（RADIUS客户端）而非RADIUS Server决定。在Switch上执行命令undo radius-server user-name domain-included后，Switch会把不带域名的用户名发送到RADIUS Server认证，但Switch仍会将用户置于huawei域中进行认证。

