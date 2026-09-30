# GitHub 用户名

> 赛道群内微信昵称：RegadPole

## 选择路线

路线三

## 项目简介 

PlumBot 是一款现代、高内聚、模块化的 Minecraft/Hytale 与 QQ/IM 机器人双向互通插件，**以AGPL-v3协议开源**

架构解耦：采用门面接口 + 多宿主 + 协议适配器设计，外部插件仅依赖纯净的`:api`模块，核心业务 100% 平台无关

多宿主与多协议：原生支持 Paper/Spigot、Velocity 及 Hytale 三端；无缝对接 OneBot 与 MiraiMC

可拓展性：基于项目高度模块化的设计理念，插件具有高度可拓展性，可以简单的拓展更多平台或适配器使用，并支持接入平台内其他插件api

核心功能：聊天双向互通、基于 AC 自动机的敏感词双向脱敏/阻断、QQ 绑定与白名单前置拦截、群内性能监控、带严格鉴权与控制台回显捕获的远程指令通道

## 项目

- 仓库：https://github.com/RegadPoleCN/PlumBot

## 训练营期间的主要增量

commit增量列表: https://github.com/RegadPoleCN/PlumBot/compare/a34db318ab398eb8f90afedf36bf31b486b0ae85...8d1a9f5263c5a347d4bf32c028dddb6394b2ec18

共74个commit

- 核心安全与稳定性加固：远程指令等地方防提权注入、资源防泄漏、添加异常隔离与状态机保护。
- 架构物理拆分与包结构规范化：抽取独立纯净的 :api 模块，配合自动契约校验任务守护二进制兼容性
- 能力高内聚整合，重塑构建体系及模块结构，遵守最佳实践要求
- 多平台生态全面扩展：基于各平台原生实现内部功能，kotlin原生写法简化代码复杂度，提升效率和可用性
- JVM采样与工程化模式，缓存部分由Caffeine + Aedile实现，符合生产环境标准
- 重构文档体系，满足新版本需求
- 使用GitHub Actions进行自动构建测试，dependabot检查依赖更新

## 过程记录

详见commit

- commit: https://github.com/RegadPoleCN/PlumBot/commits/v3
- release: https://github.com/RegadPoleCN/PlumBot/releases/tag/3.0.0-beta1

## 其他说明

- 项目为个人维护，此分支为重构版本，大量借助AI工具，commit语义清晰，已发布release
- 由于ai太难用了，所以人工大量介入修改，但90%增量仍由ai实现，大部分代码已人工审查
- 训练营开始前项目几乎不可维护，结构混乱