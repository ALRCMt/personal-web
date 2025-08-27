---
layout: page
permalink: /MtENP/index.html
title: 个人服务器搭建
---

# MtENP 系统搭建指南

本指南作为我自己摸索学习的记录，借鉴了https://github.com/firemakergk/aquar-build-helper 的内容

# 整体介绍

MtENP 系统是一套整合了多种开源组件的系统集合，本质上是跑在 Proxmox VE 虚拟化平台上的若干个虚拟机以及其中应用的个人服务器，其源于两年前欲求文件存储与共享的突发奇想，归功于<del>强大的行动力</del>（大嘘）
，在使用了各家网盘、小主机 T620&Alist 均不尽人意后，搭建了这套系统

作为一个完全的小白，这套系统的搭建帮助我了解学习计算机知识及 linux 系统，配置网络环境，以及成为一位合格的垃圾佬，可以说，我完全是一下一下摸索着搭建这套系统的，因此我认为如果你有兴趣搭建这么一套系统，不论有没有技术基础，也一定可以成功，所以在这里，我会把 MtENP 系统搭建过程中的大小问题及解决方法记录下来，希望可以帮到你

感谢这些对我有帮助的人：

[@生火人 firemaker](https://space.bilibili.com/304756911?spm_id_from=333.1387.favlist.content.click)

[@Intro_iu](https://space.bilibili.com/204773750/?spm_id_from=333.788.upinfo.head.click)

[@康文昌](https://space.bilibili.com/34786453?spm_id_from=333.1387.favlist.content.click)

[@技术爬爬虾](https://space.bilibili.com/316183842?spm_id_from=333.1387.favlist.content.click)

[@科技宅小明](https://space.bilibili.com/5626102?spm_id_from=333.1387.favlist.content.click)

如果你有疑问，或者发现了我的错误，请联系我

[![QQ](https://img.shields.io/badge/QQ-ALRCMt-white.svg)](https://qm.qq.com/q/4uVkK9nRPW?personal_qrcode_source=3)
[![邮箱](https://img.shields.io/badge/邮箱-b122330417@163.com-blue.svg)](mailto:b122330417@163.com)

## 核心功能

- Proxmox VE 虚拟化多系统
- 预装 TrueNas 并配置多种共享服务（NFS、SMB、WebDAV）
- 预装多种 Docker 服务（DPanel*docker 可视化、Syncthing*文件备份...）
- 蒲公英异地组网多平台随时使用
- 一套独立的 windows10 系统冗余
- <del>装逼</del>

## 需求与使用场景 [引用自@生火人 firemaker](https://github.com/firemakergk/aquar-build-helper?tab=readme-ov-file#%E9%9C%80%E6%B1%82%E4%B8%8E%E4%BD%BF%E7%94%A8%E5%9C%BA%E6%99%AF)

从零纯手工搭建系统的过程很长，因为这是长期沉淀下来的，有很多细节蕴含在其中。然而我更想强调的是，当你走完所有这些路途以后，你和这套系统磨合才真正开始。**一套软件系统是生长在使用者的需求上的，合理且充分的需求就像土壤，是软件系统存活乃至成长的必要条件。**

所以搭建系统前你需要问自己一些问题：

- 你希望使用这套系统解决什么问题？
- 这些问题是否有其他更便利的解决方式？
- 这套系统的特性是否适合你长期使用？

如果这些问题的答案都支持你搭建这套系统，那么请阅读接下来的内容，祝你好运

# 目录

**因为教程有点多，所以学会看目录好不好？**

1.  [**硬件选择**](#%E7%A1%AC%E4%BB%B6%E9%80%89%E6%8B%A9)
2.  [**其他条件**](#%E5%85%B6%E4%BB%96%E6%9D%A1%E4%BB%B6)
3.  [**系统配置**](#%E7%B3%BB%E7%BB%9F%E9%85%8D%E7%BD%AE)
    1.  [安装 PVE (Proxmox Virtual Environment)](#%E5%AE%89%E8%A3%85-pve-proxmox-virtual-environment)
    2.  [安装 TrueNAS (TrueNAS scale)](#%E5%AE%89%E8%A3%85-truenas-truenas-scale)
    3.  [安装 ubuntu (Ubuntu Server)](#%E5%AE%89%E8%A3%85-ubuntu-ubuntu-server)
    4.  [安装 Windows (Windows10) _不需要_](#%E5%AE%89%E8%A3%85-windows-windows10-%E4%B8%8D%E9%9C%80%E8%A6%81)
4.  [**应用配置**](#%E5%BA%94%E7%94%A8%E9%85%8D%E7%BD%AE)
    1.  [PVE 配置](#pve-%E9%85%8D%E7%BD%AE)
        1. [_设置 PVE 的 APT 源_](#1%E8%AE%BE%E7%BD%AE-pve-%E7%9A%84-apt-%E6%BA%90)
        2. [_网络唤醒 WOL_](#2%E7%BD%91%E7%BB%9C%E5%94%A4%E9%86%92-wol)
        3. [_自动开关机_](#3%E8%87%AA%E5%8A%A8%E5%BC%80%E5%85%B3%E6%9C%BA)
    2.  [旁路由 R300A 配置](#%E6%97%81%E8%B7%AF%E7%94%B1-r300a-%E9%85%8D%E7%BD%AE)
        1. [_开启路由器的 WAN 口转发_](#1%E5%BC%80%E5%90%AF%E8%B7%AF%E7%94%B1%E5%99%A8%E7%9A%84-wan-%E5%8F%A3%E8%BD%AC%E5%8F%91)
        2. [_配置异地组网_](#2%E9%85%8D%E7%BD%AE%E5%BC%82%E5%9C%B0%E7%BB%84%E7%BD%91)
    3.  [TrueNAS 配置](#truenas-%E9%85%8D%E7%BD%AE)
        1. [_实现硬盘直通_](#1%E5%AE%9E%E7%8E%B0%E7%A1%AC%E7%9B%98%E7%9B%B4%E9%80%9A)
        2. [_配置存储池及用户设置_](#2%E9%85%8D%E7%BD%AE%E5%AD%98%E5%82%A8%E6%B1%A0%E5%8F%8A%E7%94%A8%E6%88%B7%E8%AE%BE%E7%BD%AE)
        3. [_SMB 共享配置_](#3smb-%E5%85%B1%E4%BA%AB%E9%85%8D%E7%BD%AE)
        4. [_NFS 共享配置_](#4nfs-%E5%85%B1%E4%BA%AB%E9%85%8D%E7%BD%AE)
    4.  [Ubuntu 配置](#ubuntu-%E9%85%8D%E7%BD%AE)
        1. [_安装 Docker_](#1%E5%AE%89%E8%A3%85-docker-%E6%9C%80%E6%8A%98%E7%A3%A8%E4%BA%BA%E7%9A%84%E4%B8%80%E9%9B%86%E5%85%B6%E5%AE%9E%E8%BF%98%E5%A5%BD)
        2. [_安装 Docker 可视化工具 DPanel_](#2%E5%AE%89%E8%A3%85-docker-%E5%8F%AF%E8%A7%86%E5%8C%96%E5%B7%A5%E5%85%B7-dpanel)
        3. [_Docker 镜像仓库加速_](#3docker-%E9%95%9C%E5%83%8F%E4%BB%93%E5%BA%93%E5%8A%A0%E9%80%9F)
        4. [_数据卷的创建、挂载、查看、删除_](#4%E6%95%B0%E6%8D%AE%E5%8D%B7%E7%9A%84%E5%88%9B%E5%BB%BA%E6%8C%82%E8%BD%BD%E6%9F%A5%E7%9C%8B%E5%88%A0%E9%99%A4)
        5. [_将 TrueNAS 存储池挂载到指定目录_](#5%E5%B0%86-truenas-%E5%AD%98%E5%82%A8%E6%B1%A0%E6%8C%82%E8%BD%BD%E5%88%B0%E6%8C%87%E5%AE%9A%E7%9B%AE%E5%BD%95)
        6. [_Docker 部署 Resilio Sync_](#6docker-%E9%83%A8%E7%BD%B2-resilio-sync)
        7. [_Docker 部署 immich_](#7docker-%E9%83%A8%E7%BD%B2-immich)
        8. [_Docker 部署 V2rayA_](#8docker-%E9%83%A8%E7%BD%B2-v2raya)
        9. [_配置代理_](#9%E9%85%8D%E7%BD%AE%E4%BB%A3%E7%90%86)
        10. [_Docker 部署 qBittorrent WebUI_](#10docker%E9%83%A8%E7%BD%B2qbittorrent-webui)
        11. [_Docker 部署 OpenList_](#11docker%E9%83%A8%E7%BD%B2openlist)
5.  [**注意事项**](#%E6%B3%A8%E6%84%8F%E4%BA%8B%E9%A1%B9)
    1.  [PVE 安装时卡死](#01pve-%E5%AE%89%E8%A3%85%E6%97%B6%E5%8D%A1%E6%AD%BB)
    2.  [_PVE 网卡莫名其妙掉线问题 不确定_](#02pve-%E7%BD%91%E5%8D%A1%E8%8E%AB%E5%90%8D%E5%85%B6%E5%A6%99%E6%8E%89%E7%BA%BF%E9%97%AE%E9%A2%98-%E4%B8%8D%E7%A1%AE%E5%AE%9A)
    3.  [ssh 功能开启问题](#03ssh-%E5%8A%9F%E8%83%BD%E5%BC%80%E5%90%AF%E9%97%AE%E9%A2%98)
    4.  [PVE8 概要面板显示 CPU 温度](#04pve8-%E6%A6%82%E8%A6%81%E9%9D%A2%E6%9D%BF%E6%98%BE%E7%A4%BA-cpu-%E6%B8%A9%E5%BA%A6)
    5.  [Ubuntu 空间仅占用一半](#05ubuntu-%E7%A9%BA%E9%97%B4%E4%BB%85%E5%8D%A0%E7%94%A8%E4%B8%80%E5%8D%8A)
    6.  [PVE 更换 apt 源后报错](#06pve-%E6%9B%B4%E6%8D%A2-apt-%E6%BA%90%E5%90%8E%E6%8A%A5%E9%94%99)
    7.  [**C6-State 导致 PVE 崩溃**](#07c6-state%E5%AF%BC%E8%87%B4pve%E5%B4%A9%E6%BA%83)
    8.  [BIOS 时区错误](#08bios%E6%97%B6%E5%8C%BA%E9%94%99%E8%AF%AF)
    9.  [PVE 开机显示 ZFS 导入错误](#09pve%E5%BC%80%E6%9C%BA%E6%98%BE%E7%A4%BAzfs%E5%AF%BC%E5%85%A5%E9%94%99%E8%AF%AF)
    10. [PVE 降低功耗](#10pve%E9%99%8D%E4%BD%8E%E5%8A%9F%E8%80%97)
