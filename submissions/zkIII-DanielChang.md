# GitHub 用户名：zkIII-DanielChang

> 赛道群内微信昵称：智感2502张大川

## 选择路线

路线三（旧项目 / 闭源项目维护展示类）

## 项目简介

**《铁冠诸侯》(Lords of Iron Crowns)** 是一款基于 JavaScript 的网页大战略游戏，双端适配（桌面 / 移动），玩法参照《十字军之王》：经营领地、征兵作战、外交博弈、吞并邻国。游戏本体为单文件 JS（`index.html`，约 3900 行），是训练营开始前的既有项目。

**训练营期间的增量工作**是为这个既有游戏从零构建 **AI 执政官 `RegentAgent`**——基于 HelloAgents 框架的 LLM 智能体，让它像人类玩家一样自主操控游戏：每回合读取中文局势观测，通过函数调用执行游戏动作，独立完成一场从发展到征战的多回合战役。

这个增量解决的真实问题是：大战略游戏状态复杂、动作空间多样，传统脚本 AI 难以通盘决策。本项目打通了「LLM 智能体 ↔ 真实网页游戏」的完整桥接链路，既让旧项目获得了可展示的新能力，也构成一个回合制策略游戏的 LLM 决策基准测试环境。

## 项目 / PR

- 仓库：[zkIII-DanielChang/Lords-of-Iron-Crowns](https://github.com/zkIII-DanielChang/Lords-of-Iron-Crowns)
- 增量代码目录：[`zkIII-DanielChang-RegentAgent/`](https://github.com/zkIII-DanielChang/Lords-of-Iron-Crowns/tree/main/zkIII-DanielChang-RegentAgent)
- Release：[release](https://github.com/zkIII-DanielChang/Lords-of-Iron-Crowns/releases/tag/release) ／ [pre_release](https://github.com/zkIII-DanielChang/Lords-of-Iron-Crowns/releases/tag/pre_release)
- 在线 Demo（点开即玩，无需下载 / 注册）：
  - 游戏全屏入口：<https://zkiii-danielchang.github.io/game/>
  - 项目介绍博客（正文内嵌同一份游戏，读完即可直接试玩，阅读与试玩零跳转）：《铁冠诸侯：一个为了好玩而做的大战略游戏》<https://zkiii-danielchang.github.io/2026/09/27/%E9%93%81%E5%86%A0%E8%AF%B8%E4%BE%AF%E9%A1%B9%E7%9B%AE%E4%BB%8B%E7%BB%8D/>
- Agent 运行演示：克隆仓库后执行 `zkIII-DanielChang-RegentAgent/main.ipynb`（从头运行即可），或按该目录 `README.md` 的脚本方式启动；每回合会打印决策摘要与国力变化，示例见下文「运行输出示例」
- 运行录屏：⚠️ 待补（可选项，建议补一段 6 回合战役录屏，约 5 分钟；本项目已具备在线可玩演示与完整 commit diff，录屏用于进一步佐证 AI 自主决策过程）

## 训练营期间的主要增量

增量以单个提交 `e5d0b29`「添加Agent」为主体，共 **26 个文件 / 7404 行新增**，另有 `1e9f348`「修改README」补充文档。具体分为四层：

**1. 游戏侧：新增 `window.GameAgent` 桥接 API（约 300 行）**

在不改动任何既有游戏逻辑的前提下，为单文件 JS 游戏增加一个桥接模块，提供四类能力：局势快照导出、动作分发、回合窗口守卫、事件弹窗处理。设计上只做 `id → 对象` 解析后转调既有函数，不重复实现游戏规则，因此人类正常游玩完全不受影响。

**2. Python 侧：新建 `RegentAgent` 完整项目**

| 模块 | 作用 |
|---|---|
| `src/game_bridge.py` | Playwright 无头 Edge + Python 本地静态服务，驱动真实游戏页面；复用系统自带 Edge（`channel="msedge"`），免下载浏览器 |
| `src/game_tools.py` | 12 个强类型 HelloAgents 工具：`game_state` + 10 个动作工具 + `note_strategy` 战略笔记；每回合动作预算防刷屏 |
| `src/prompts.py` | 生成紧凑中文局势观测：国力 / 净收入 / 关系 / 军队（含合法移动列表与战斗胜率预测）/ 战争 / 事件 / 新消息差分 / 战略笔记 |
| `src/game_agent.py` | `CampaignController` 战役主循环：每回合决策 → 工具行动 → 强制推进回合 |
| `src/compat.py` | DeepSeek DSML 兼容层：把 `deepseek-chat` 的 `<｜DSML｜ invoke>` 文本工具调用归一化为原生 `tool_calls`，框架零改动 |
| `src/e2e_test.py` | 端到端测试 |
| `main.ipynb` | 可从头执行的完整演示 |

**3. 可复现性：确定性回放**

地图种子固定 + 全逻辑 rng 播种（含撤退目的地等随机分支），做到「同种子 + 同决策 = 同战局」，使 AI 决策效果可被对照评估。

**4. 过程留档：战役全程可复盘**

每回合的 AI 决策原文（`ai_reply`）、完整动作轨迹、国力指标写入 `outputs/campaign.jsonl`，并按间隔截图 `outputs/turn_*.png`。仓库中保留的是精选副本，位于 `outputs/demo/`（`.gitignore` 默认排除全部运行时产物，仅放行该目录，避免体积膨胀与密钥泄露）。

**实测性能**（2026-09-30，`deepseek-chat`，6 回合战役）：

- 单回合决策耗时约 30~60 秒（含 4~6 次工具调用）
- 6 回合战役约 5 分钟；12 回合预计 10 分钟左右
- 非法动作 100% 被游戏侧校验拒绝并返回原因
- 零 API 路径：LLM 不可用时自动切换策略基线，6 回合约 20 秒

## 过程记录

**commit**（均为训练营期间，2026-09-28 ~ 09-30）：

| commit | 时间 | 内容 | 改动量 |
|---|---|---|---|
| `66f858a` | 09-28 16:15 | 游戏本体入库 | 1 文件 / 3878 行 |
| `d4e91dc` | 09-28 16:38 | LICENSE + README | 2 文件 / 663 行 |
| `3493e09` | 09-28 17:09 | 提交登记草稿 | 1 文件 / 37 行 |
| `e5d0b29` | 09-30 19:02 | **添加 Agent（本次核心增量）** | 26 文件 / 7404 行 |
| `1e9f348` | 09-30 19:13 | 修改 README | — |

- 提交历史：<https://github.com/zkIII-DanielChang/Lords-of-Iron-Crowns/commits/main>
- 核心增量 diff：`e5d0b29`
- **release**：已发布 `release` 与 `pre_release` 两个版本标签
- **决策轨迹**：[`outputs/demo/campaign.jsonl`](https://github.com/zkIII-DanielChang/Lords-of-Iron-Crowns/blob/main/zkIII-DanielChang-RegentAgent/outputs/demo/campaign.jsonl)——逐回合记录 AI 决策原文（`ai_reply`）、完整动作轨迹与国力指标，可直接验证是 LLM 在真实决策而非脚本回放
- **运行截图**：[`outputs/demo/`](https://github.com/zkIII-DanielChang/Lords-of-Iron-Crowns/tree/main/zkIII-DanielChang-RegentAgent/outputs/demo) 下的 `turn_0003.png`、`turn_0006.png`、`turn_0007.png`、`turn_0012.png`，对应战役第 3 / 6 / 7 / 12 回合的实际战局
- **开发日志**：见项目介绍博客《开发历程》一节（下方为摘要）

**游戏本体开发历程**（摘自博客 timeline，可证明本项目为既有项目、并展示其架构演进）：

| 时间 | 阶段 | 内容 |
|---|---|---|
| 2026-09-01 | 立项与骨架 | 确定分阶段路线；主循环、时间系统、四层画布架构搭好，空白地图能跑回合 |
| 2026-09-06 | 系统爆发周 | 大地图（Voronoi 省份、国家配色、三级边界、LOD 文字）、UI 信息系统、国家经济、军事战争、人物封臣链一口气落地 |
| 2026-09-12 | 手机适配与存档 | 世界地图竖置，底部 Tab + 三档抽屉的竖屏界面重做；localStorage 存档系统上线，刷新不丢进度 |
| 2026-09-13 | 军事打磨 | 一键全军集结（自动全领土征兵）、军队合并、收复被占领土、进攻目标红绿框可视化 |
| 2026-09-27 | 定名 + 博客发布 | 告别曾用名「王国风云」，正式定名《铁冠诸侯》；项目介绍博客上线并内嵌可玩版本 |

**训练营期间增量**：`e5d0b29`（09-30 19:02）落地 AI 执政官 `RegentAgent`，`1e9f348`（09-30 19:13）补充文档。

值得注意的是，09-27 发布的博客在「三赛道参赛声明」中，把 Agent 赛道明确写为「**计划中**」的「用一句话操控国家」，并给出了设想中的自然语言下旨示例。而 `RegentAgent` 的实际落地时间是 09-30。这一时间差本身就构成「Agent 是训练营期间从计划到实现的真实增量」的可验证证据。

**运行输出示例**（每回合决策与国力变化）：

```text
[回合 1/6] 1066年 2月 领地10省 国库3284 人力476 战争0场 动作5次
1066年 2月 | 领地10省 | 国库3284 | 人力476 | 战争0场 | 动作: build, build, raise_army, move_army, end_turn
1066年 4月 | 领地10省 | 国库3927 | 人力211 | 战争1场 | 动作: raise_army, move_army, declare_war, end_turn
```

## 个人角色与项目理解

单人项目，游戏本体与 AI 执政官均由本人独立完成，无协作分工问题。本次训练营期间负责的是 Agent 层从零到可运行的全部工作，包括游戏侧桥接 API 的设计与嵌入。

### 对既有项目（游戏本体）架构的理解

游戏本体是单文件 JS（`index.html`，约 3900 行），零构建工具、零 npm 依赖、零后端，丢到任何静态服务器上即可运行，手机竖屏单手可操作。其架构分三层：

- **渲染层：四张叠放的 Canvas**——地图层（低频重绘的省份 / 边界 / 文字）、单位层（逐帧的兵牌）、特效层（选中呼吸动画）、UI 层（悬停高亮 / 移动范围）。每层独立脏标记，相机平移缩放只在需要时重绘。
- **数据层：一切皆 `GameState`**——省份是多边形 + 邻接表，国家是省份集合 + 财政，人物靠 `liegeId` 串成封臣链：

```json
{
  "character": {
    "name": "亨利·维斯特兰",
    "title": "duke",
    "age": 43.25,
    "dip": 12, "mar": 17, "ste": 9,
    "traits": ["勇猛", "精明"],
    "liegeId": 3
  }
}
```

  顺着 `liegeId` 往上爬就是「伯爵→公爵→国王→皇帝」的完整链条；人物死亡时，头衔、直辖领地和所有附庸的效忠对象自动指向继承人。

- **存档层：地图几何不存档**——由种子确定性重建，存档只序列化可变状态（归属、建筑、人物、外交矩阵、军队、战争、财政），单份约 27KB，损坏时可静默降级新开一局。

游戏系统覆盖：程序化大地图（Voronoi 切块生成省份，海陆 / 地形 / 发展度由种子确定性生成）、封君封臣四级链条、国家经济（国库 / 人力 / 稳定度逐月结算，税收 = 发展度 × 地形修正 × 建筑加成，税务所 / 兵营 / 市场 / 要塞四种建筑）、军事战争（宣战理由、征兵集结、兵牌移动、地形防守、围城破城、战争分数、强制割地、收复失地）、人物与继承、本地存档、竖屏界面。

**正是这套「一切皆 GameState + 系统已是可编程接口」的数据驱动设计，使 Agent 得以低成本接入**——桥接层只需读取 `GameState` 生成观测、按 `id → 对象` 解析后转调既有动作函数，无需为 AI 重写任何游戏规则。这是本次增量能够「零侵入」落地的架构前提。

### 新增量（RegentAgent）数据链路

自上而下：

```text
LLM (deepseek-chat)
  ↓ 工具调用（compat.py 归一化 DSML → tool_calls）
HelloAgents SimpleAgent + ToolRegistry（12 个强类型工具）
  ↓
CampaignController（战役主循环 / 兜底策略）
  ↓
GameBridge（Playwright 无头 Edge + 本地静态服务）
  ↓ 调用 window.GameAgent
游戏本体 index.html（既有逻辑，零改动）
```

关键设计取舍：

- **零侵入桥接**：游戏侧只加一个约 300 行的 `GameAgent` 模块，全部复用既有游戏函数，避免逻辑双写导致规则不一致，也保证人类游玩不受影响。
- **回合窗口守卫**：游戏侧强制「每回合只能 `end_turn` 一次、动作上限 12 次」，从协议层杜绝模型重复结束回合或刷屏。
- **永不卡死的主循环**：`end_turn` / `resolve_event` 双重兜底 + LLM 失败自动切换策略基线，任何模型失误都不中断演示；未处理的事件弹窗自动选 primary 选项。
- **规则拒绝 ≠ 工具错误**：游戏侧的正常拒绝（如国库不足）作为 `success` 反馈返回而非抛错，避免 HelloAgents 框架的熔断器把游戏规则误判为工具故障。

## 学到的东西

- 掌握了 HelloAgents 的 `SimpleAgent` + `ToolRegistry` 函数调用范式，理解了工具 schema 设计对模型行为的影响
- 解决了非原生支持工具调用的模型（DeepSeek DSML 文本协议）与框架的对接问题，做到框架零改动
- 用 Playwright 打通 Python 与真实网页的双向通信，理解了「浏览器自动化驱动既有前端」这条低成本改造路径
- 体会到 LLM 智能体的可靠性主要靠工程约束而非模型能力：动作预算、回合守卫、多重兜底才是演示不中断的真正原因

## 后续计划

- [ ] 多模态视觉输入：截图直接喂给 VLM 决策
- [ ] 长期记忆：战局复盘写入记忆，跨战役学习
- [ ] 范式对比：ReAct / Reflection / Plan-and-Solve 同局对战评测
- [ ] 玩家接管：`headless=False` 实景观看 + 中途切换人工操控

## 其他说明

**关于本项目的性质**：游戏本体《铁冠诸侯》是训练营开始前的既有项目，本次选择路线三（旧项目维护展示类）。训练营期间的实际增量为 AI 执政官 `RegentAgent` 全套代码（26 文件 / 7404 行）及游戏侧 `GameAgent` 桥接模块，对应提交 `e5d0b29`、`1e9f348`，全部为本人独立编写。

**关于提交粒度**：核心增量集中在单个提交 `e5d0b29` 中，是因为 Agent 项目需要与游戏侧桥接 API 同时到位才能运行，开发期间在本地完成联调后一次性入库。为弥补过程记录，本项目提供了三类可独立验证的增量证据：`campaign.jsonl` 的逐回合 AI 决策原文、`turn_*.png` 的运行截图、以及已发布的 `release` / `pre_release` 版本标签。前两类需按上文提示放入 `outputs/demo/` 白名单目录并推送后才可在线查看。

**关于 AI 辅助**：开发过程中使用了 AI 辅助编码，所有代码均经本人审查、理解并实际联调通过，可现场解释任一模块的实现思路与设计取舍。

**工程规范**：MIT License、`requirements.txt` 依赖清单、`.env.example` 配置模板、`.gitignore`、`src/e2e_test.py` 端到端测试。

**致谢**：感谢 Datawhale 社区和 Hello-Agents 项目。
