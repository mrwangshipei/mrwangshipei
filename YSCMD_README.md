# YSCMD 管理系统

[![License: Proprietary](https://img.shields.io/badge/License-Proprietary-red)](https://en.wikipedia.org/wiki/Proprietary_software)

YSCMD（数据中心管理系统）是一个用于**数据中心管理、数据更新与上传**的综合性后台系统，集成了前后端功能，适用于数据密集型业务场景。

> **注意**: 本项目为商业闭源软件，仅供内部使用。

本系统基于 [偌依开源管理系统](https://github.com/yangzongzhuan/RuoYi) 修改扩展，针对特定业务流程进行了定制优化，适配企业数据上传、更新和权限管理需求。

---
## 目录
- [YSCMD 管理系统](#yscmd-管理系统)
  - [目录](#目录)
  - [🚀 功能概览](#-功能概览)
  - [🛠 技术栈](#-技术栈)
    - [后端：](#后端)
    - [前端：](#前端)
    - [客户端：](#客户端)
  - [📦 安装与运行（开发模式）](#-安装与运行开发模式)
  - [📕 项目开发指南（Windows + Docker）](#-项目开发指南windows--docker)
    - [以下是王工开发使用的环境：](#以下是王工开发使用的环境)
    - [通信开发指南](#通信开发指南)
  - [🚀 项目部署指南（Windows + Docker）](#-项目部署指南windows--docker)
    - [一、部署操作](#一部署操作)
      - [生成部署包](#生成部署包)
      - [上传并解压到目标服务器](#上传并解压到目标服务器)
    - [📁 项目结构说明](#-项目结构说明)
  - [开发须知](#开发须知)
    - [📜 LICENSE 协议说明](#-license-协议说明)
    - [👨‍💻 开发维护者](#-开发维护者)
    - [❓ 常见问题解答](#-常见问题解答)
    - [📌 TODO（后续开发计划）](#-todo后续开发计划)

<!-- TOC -->
<!-- /TOC -->
---

## 🚀 功能概览

- ✅ 数据文件上传与批量更新管理
- ✅ 数据中心模块视图与权限控制
- ✅ 多角色支持的权限系统（基于 RBAC）
- ✅ 文件版本跟踪与状态标记
- ✅ 审核流程、审批与日志记录
- ✅ 系统参数配置与扩展性良好的模块架构

---

## 🛠 技术栈

### 后端：
- 基于 Spring Boot
- MyBatis Plus + Redis + JWT
- 使用 ruoyi 架构体系（部分模块重构）
- 支持 RESTful API 规范

### 前端：
- Vue 3 + Element Plus + Vue Router + Vuex
- 使用偌依 Vue 前端架构（重构部分页面）
- Axios 通信模块优化

### 客户端：
- Winform + .net FrameWork + M2Mqtt.net


---

## 📦 安装与运行（开发模式）

> ⚠️ 默认使用 MySQL 和 Redis，需本地安装或提供服务地址。

## 📕 项目开发指南（Windows + Docker）

### 以下是王工开发使用的环境：

> OpenJDK 版本:
> java version "1.8.0_311"
> Java(TM) SE Runtime Environment (build 1.8.0_311-b11)
> Java HotSpot(TM) 64-Bit Server VM (build 25.311-b11, mixed mode)

> Docker 版本:
> docker --version
> Docker version 28.3.2, build 578ccf6

> 操作系统信息:
> systeminfo | findstr /B "OS"
> OS 名称:            Microsoft Windows 11 企业版
> OS 版本:            10.0.26100 暂缺 Build 26100
> OS 制造商:          Microsoft Corporation
> OS 配置:            独立工作站
> OS 构建类型:        Multiprocessor Free

> fnm 版本：
> fnm 1.38.1

> node 版本：
> node v16.20.2


> npm 版本：
> npm 8.19.4
> registry = "https://registry.npmjs.org/"

### 通信开发指南

后端与下位机的通信协议是mqtt，我们一般需要用ssl加自定义加密模式
首先通信的基础模板如下
```json
{
  "fn":"FUNC001",
  "dn":"DEVICE123",
  "msg":"Hello World",
  "k":"sadkpokaaopdkasopd1sa-231",
  "timestamp":"17769583565"
  }
```
要通过后端的验证需要以下两个前提
1. **timestamp**作为毫秒时间戳，必须和服务器的时间差距在30分种内
2. k作为密钥 必须使用某种加密算法加密时间戳，使用私钥加密，私钥在源码中查阅。

所有的协议都是由基础模板的扩展，基础模板已经在c#提供的dll中封装

## 🚀 项目部署指南（Windows + Docker）

### 一、部署操作

#### 生成部署包

在项目根目录双击 **package.bat**
→ 自动生成 **yscmd-deploy.zip**

#### 上传并解压到目标服务器

- 得到 yscmd-deploy 文件夹
- 根据需求压缩到服务器解压
- 打开处于文件夹根目录的 **000_Main.bat**
- 提示如下是正常的
``` cmd
=========================================
         容器维护中枢脚本
=========================================
 [1] 完整部署 (00_deploy-full.bat)
 [2] 更新部署 (01_deploy-update.bat)
 [3] 查询一次容器状态 (02_state_single.bat)
 [4] 持续监控容器状态 (02_state_watch.bat)
 [5] 停止部署 (99_deploy_shutdown.bat)
 [6] 删除 yscmd 容器 (DD_remove-yscmd-containers.bat)
 [7] 查看已停止容器的日志 (调用 bin/errlog.bat)
 [8] 启动/安装docker (调用 docker\01_install\01_Start_docker.bat)
 [9] 构建 docker jdk 镜像 ( 一般无需构建 )  (调用 02_ifneed_buildimage\01_single_build-image.bat)
 [10] 构建 docker 其他 镜像  ( 一般无需构建 )  (调用 docker\02_ifneed_buildimage\02_single_pull-image.bat)
 [11] 导出所有镜像  ( 一般无需导出 )  (调用 docker\02_ifneed_buildimage\03_single_export-image.bat)
 [12] 导入所有镜像 (调用 docker\02_ifneed_buildimage\03_single_export-image.bat)
-----------------------------------------
 [X] 退出
=========================================
请输入选项编号并回车:
```

- 在初次部署需要： 8-12-1-4，直到看到所有节点都处于绿色状态
- 在后续部署需要： 8-2-4，直到看到所有节点都处于绿色状态

### 📁 项目结构说明

参考ruoyi开源项目

## 开发须知

1. 如何在现有的数据库上修改，并保证后续版本迭代无误？

> 在项目的文件夹 **yscmd\sql** 中的文件按照名称排序，可以看到最后一个版本 05_somechange.sql,复刻命名06_somechange.sql，在**package.bat**执行后会自动打包

2. 我的数据库等文件部署后在哪里？

> 数据库会部署在**yscmd-deploy.zip**解压文件夹中的sql路径下，不要随意删除这个文件夹，并谨慎的选择一开始的部署磁盘。

---

### 📜 LICENSE 协议说明

本项目基于 偌依 ruoyi 二次开发，遵守其 MIT 协议。
请在使用或分发本系统时，保留原始协议文件及说明。

### 👨‍💻 开发维护者

王工（主程序员）

---

### ❓ 常见问题解答
[Some Help](./doc/help.md)

### 📘 API接口文档
[API Documentation](./doc/api_documentation.md)

### 📌 TODO（后续开发计划）
