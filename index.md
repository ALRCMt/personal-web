---
layout: home
mermaid: true
title: 主页
permalink: /
---
<img src="./images/fav.png" alt="ALRCMt" width="150px"/>

<b>这里是ALRCMt的个人网站！ :)</b>

<table width="200">
<tr>
<th colspan=2>疯狂星期四，v我50</th>
</tr>
<tr>
<th><a href="./images/wxlll.jpg" target="_blank"><img width="50" height="50" src="./images/wechatpay.png"></a></th>
<th>我，Mt 打钱 懂？</th>
</tr>
</table>

<hr />

鉴证大舞台有种你就来，没种我也来，我不过只是看门人  
<img src="/images/mhyyyy.png" alt="沉痛悼念米指导" height="200px">

沉默看书，QQ、B站、知乎、TG、X通通不辩经（哎哲逼怎么这么坏）  
理科生理论水平低低低低（我看文科生也没好到哪里去）  

装机入门水平，玩玩NAS、软路由，平时折腾折腾Liunx、PVE  
<img src="/images/psc.jpg" alt="喜报" width="300px">  
不曾熟练掌握任何计算机语言，极不熟练使用JavaScript  
无论写什么都要De一周Bug  
<img src="https://raw.githubusercontent.com/ALRCMt/MtAIO-Build/cb99109050678a8dc9f7933bb70bc5681e4f1084/photo/%E5%B1%8F%E5%B9%95%E6%88%AA%E5%9B%BE%202025-08-16%20161707.png" width="280px">  

Ctrl+C太好用了你知道吗 （哎\u{200b}怎么这么坏）

> <b>重大提示：
> <i>我不是男娘 不OD 不改花刀</i></b>

<hr />
<i>装机线是不理的</i>  
<img src="./images/server_open.jpg" width="280px">

<i>代码是懒得写的（截止2025.9）</i>  
<img src="./images/stats.png" alt="stats" width="300px">

<i>塔学造诣是没有的</i>  
<img src="./images/ban.png" alt="账号已封禁">

<hr />

### 成就

- H.M.R.资料库的建立（旧）
- MtAIO个人服务器的搭建

前者没有什么技术含量（实则不然），断断续续整理了两千多件社会科学资料，目前该项目正处于重开发状态  
后者纯折腾折腾，看我教程应该都看得明白（是吧？），相信就连你也可以学会  

- HMR(显然服务没有开)
<img src="./images/hmr_data.png" width="650px">

- 服务器外观（手办是蕾娜，随便买的便宜货）  
<img src="./images/server_over.png" width="400px">

- 服务器web  
<img src="./images/MtENP_web.png" width="600px">
<hr />

<br />


<b>我家稀烂的网络结构(目前)</b>

```mermaid
flowchart TD
    subgraph A [“二楼网络”]
        direction TB
        A1[“互联网</br>（光纤接入）”]
        A2[“光猫兼主路由</br>HS8546V5</br>路由模式</br>IP: 192.168.1.1”]
        A3[“光猫自带WiFi</br>SSID: CMCC-XXX”]
        
        A1 -- 光纤 --> A2
        A2 -- 无线 --> A3
    end

    subgraph B [“三楼网络”]
        direction TB
        B1[“三楼路由器</br>R300A</br>NAT模式</br>LAN IP: 10.168.1.1”]
        B2[“三楼WiFi</br>（由R300A提供）”]
        B3[“PVE 服务器</br>IP: 10.168.1.217”]
        B4[“蒲公英组网服务</br>（基于R300A网络）”]
        
        B1 -- 无线 --> B2
        B1 -- 有线 --> B3
        B1 -- 网络连接 --> B4
    end

    A2 -- “网线</br>（光猫LAN口 → R300A WAN口）” --> B1

```
<b>计划网络结构</b>
```mermaid
flowchart TD
    subgraph A [“二楼: 光猫层”]
        direction TB
        A1[“互联网</br>（光纤接入）”]
        A2[“HS8546V5光猫</br>桥接模式</br>管理IP: 192.168.1.1”]
        A3[“光猫自带WiFi</br>通过单线复用启用”]
        A1 -- 光纤 --> A2
        A2 -- 无线 --> A3
    end

    subgraph C [“三楼: 主路由及设备层”]
        direction TB
        C1[“主路由 RAX3000M</br>PPPoE拨号</br>WAN: 获取公网IP</br>LAN: 如 192.168.10.1”]
        C2[“PVE 服务器</br>IP: 192.168.10.x</br>（需获取IPv6）”]
        C1 -- 有线连接 --> C2
    end

    B[“单根网线</br>（单线复用）”]

    A2 -- “承载:</br>- RAX3000M拨号数据</br>- 光猫WiFi回程数据” --> B
    B --> C1

    D[“IPv6数据流</br>（穿透至PVE）”]

    A1 -.->|分配IPv6前缀| A2
    A2 -.->|桥接IPv6| C1
    C1 -.->|路由/分配IPv6| C2
    
```