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


