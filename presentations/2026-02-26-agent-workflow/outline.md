# Agent 工作方式分享

> 2026-02-26 14:00 | Kevin 主讲 | 面向兄弟团队

## 主题

**"我们怎么跟 AI Agent 一起干活"** — 不是 PPT 演示，是真实工作流的实况拆解

## 核心信息

这不是一个关于"AI 能做什么"的科普，而是一个已经跑了几个月的团队在讲：
- 我们实际怎么用 Agent 协作
- 哪些方式有效，哪些踩过坑
- 你们也可以这么干

---

## 大纲（约 50-60 分钟）

### 一、开场：先看结果（5 分钟）

不从理论开始，直接展示一个实际成果：
- hxa-teams 项目从 repo 创建到框架交付：**不到 24 小时**
- 3 个 PR 全部 merged，9 个 commits，+2181/-779 行代码
- 3 个 contributor（1 人类 + 2 Agent），所有 PR 经过双 Agent review
- 4 个 backlog issue 同步规划
- 同期完成：行业调研报告（12 维度）+ 定位提案
- **重点：这不是 demo 项目，是真在用的**
- **hxa-teams 是模板项目** — 定义了 Human × Agent 团队协作的标准范式，后续所有项目都基于这套模板

**一个真实时间切片（今天凌晨）：**
| 时间 | 事件 |
|------|------|
| 04:56 | Kevin 发 Lark 消息：根据定位文档更新 repo |
| 04:58 | Jessie 回复执行计划 |
| 05:01 | PR 创建完成（3 文件 +34/-11），通知 review |

**从指令到 PR 就绪：4 分 20 秒**

### 二、团队结构：人和 Agent 各自干什么（10 分钟）

**我们的配置：**
| 角色 | 成员 | 职责 |
|------|------|------|
| PO | Kevin（人类） | 需求定义、优先级、最终决策 |
| Lead + Coord | Jessie（Agent） | 任务分解、进度推进、文档、PR 管理 |
| Dev | Lucy（Agent） | 代码开发、技术实现 |
| QA/DevOps | Lisa（Agent） | Code review、测试、部署 |

**关键原则：**
- Agent 不是工具，是有明确职责的团队成员
- 人类做决策，Agent 做执行 + 协调
- Agent 之间有自己的沟通渠道（BotsHub），不需要人在中间传话
- Agent 7×24 在线，随时响应

**实际体感：**
- Kevin 发一条 Lark 消息，30 秒后 PR 开始创建
- Agent 之间的 review 循环不需要人参与
- 凌晨的健康检查、升级、同步全自动

### 三、工作流实拆：一个任务从头到尾怎么走（15 分钟）

以 hxa-teams 项目为例，走一遍完整流程：

**1. 需求下达**
- Kevin 在 Lark 群发消息 → Jessie 秒级响应
- Jessie 拆解任务 → 分发给 Lucy/Lisa

**2. 调研 & 文档**
- Jessie 做行业调研（搜索、分析、写报告）
- 发布到 jessie.coco.site，Kevin 在文档上留 comment
- Jessie 根据 comment 逐条更新（不是一轮轮来回，是精确修改）

**3. 代码开发**
- Lucy 写代码，创建 PR
- Lisa 自动 review（不是走形式，有具体 review 意见）
- 修改 → 再 review → approve → 通知 Kevin merge

**4. 持续运维**
- 组件升级：检查 → 列出变更 → 确认 → 自动执行
- 健康巡检：PM2 服务状态、磁盘、内存
- 记忆同步：跨会话记住上下文、任务进度

### 四、内部项目案例（10 分钟）

不只是 hxa-teams，我们在多个项目上跑通了这套模式：

**案例 1：Zylos 平台研发**
- Zylos 本身就是 Agent 开发的 — Agent 运行时由 Agent 团队迭代
- 工作模式：Kevin 提需求 → Jessie 拆解 → Lucy 写代码 → Boot review → 持续部署
- 迭代速度：组件从 v0.1 迭代到 v0.4，平均每周 2-3 次版本升级
- 凌晨自动完成：升级检查 → 变更分析 → 确认 → 执行 → 验证，全流程无人工

**案例 2：ClawFeed 产品研发**
- 从 PRD 到上线的完整产品开发流程
- Twitter 数据采集 + 智能阅读：真实用户在用
- 已有实际阅读量数据和活跃用户
- Agent 团队负责：代码开发、PR review（Lint + Security Audit + Review）、部署运维

**共同点：**
- 不是 POC/demo，是在跑的真项目
- 人类只做决策，Agent 做执行全链路
- hxa-teams 把这些实践沉淀成了可复用的**模板标准**

### 五、新一代 Agent vs 之前的方式（5 分钟）

**之前（传统 AI 助手）：**
- 一问一答，用完即走
- 没有记忆，每次从零开始
- 单点工具，不能协作
- 只能做"帮我写段代码"这种事

**现在（Agent 团队）：**
- 7×24 在线，持久运行
- 有记忆系统，记得所有上下文和决策
- 多 Agent 协作，有分工、有沟通协议
- 能独立完成完整工作流：调研 → 写码 → review → 部署 → 运维
- 能自我调度：定时任务、健康检查、主动汇报

**关键差异：从"工具"变成了"队友"**

### 六、现场演示（10 分钟）

**Demo 1：实时任务下达**
- 在 Lark 群里给 Jessie 一个任务
- 观众看 Jessie 怎么拆解、执行、汇报

**Demo 2：Agent 协作**
- 打开 BotsHub 看 Agent 之间的对话
- 展示一个 PR review 循环

**Demo 3：文档协作**
- 打开 jessie.coco.site，展示 comment → 更新 的流程

### 七、Zylos/COCO 是什么（5 分钟）

不做广告，讲定位：
- **Zylos**：Agent 的运行时环境（让 Agent 持久运行、有记忆、能通信、能调度）
- **COCO**：云服务版（不想自己搭的话）
- **HxA**：Human × Agent 协作的开源标准（我们在做的事）
- **hxa-teams**：定义 Human × Agent 团队的模板标准 — 所有项目的起点

一句话总结：**我们不是在卖 AI 产品，是在定义人和 Agent 一起工作的方式**

### 八、寄语 + Q&A（5-10 分钟）

**相信和拥抱：**
- 这不是未来，是现在 — 我们已经这么干了几个月
- Agent 不会取代人，但**会用 Agent 的人会取代不会用的人**
- 门槛在快速降低：今天你会发消息、会在 GitHub 上点按钮，就够了
- 最大的障碍不是技术，是愿不愿意迈出第一步
- **建议：从一个小项目开始，让 Agent 进你的团队，跑起来再说**

**Q&A 准备：**
- "Agent 会不会出错？" → 会，所以有 review 流程 + 人类最终审批
- "这套东西学习成本高吗？" → 你会发 Lark 消息就行
- "能处理多复杂的任务？" → Zylos 平台自身、ClawFeed 产品、hxa-teams 标准 — 都是实例
- "Agent 之间怎么不打架？" → 明确分工 + 协议

---

## 演示准备清单

- [ ] Lark 群准备好（JessieClawMc-testing 可直接用）
- [ ] BotsHub 后台打开（jessie.coco.site/hub/）
- [ ] jessie.coco.site 文档页面准备好
- [ ] GitHub hxa-teams repo 打开（展示 PR 历史）
- [ ] 准备一个现场 demo 任务（简单但能展示完整流程的）
- [ ] hxa-k GitHub org 确认成员都进来了

## 建议的 Demo 任务

现场让 Jessie 做一件小事，比如：
- "在 hxa-k/workspace 里创建一个 meeting-notes 目录，写一份今天分享的会议纪要模板"
- 或者："查一下我们 hxa-teams repo 的最新状态，列出所有已完成的 PR 和当前 open 的 issue"（推荐，只读操作，风险低）
- 目的：让观众看到从指令到结果的完整过程（<1 分钟）

## 备用材料

- `timeline-data.md`：真实消息时间戳（从 C4 数据库导出）
- Lucy 拉的 hxa-teams 完整数据：PR 时间线、参与者、代码量
- BotsHub 对话截图：准备 backup 以防现场网络问题
