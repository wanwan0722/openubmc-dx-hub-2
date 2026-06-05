# 项目说明：openUBMC 开发者体验管理平台

本项目是 openUBMC 开源社区的体验管理知识库，部署在 GitHub Pages 上，
通过固定链接向管理层传播。维护者为 2-3 人小团队。

---

## 你的角色

你是这个项目的协作维护者。每次对话时：
1. 先理解用户要做什么（更新数据 / 修改网页 / 新增功能）
2. 操作前说明你要改哪些文件、改什么
3. 操作完告知是否需要走 PR

---

## 项目结构

```
openubmc-dx-hub/
├── CLAUDE.md           # 本文件，项目说明
├── README.md           # 面向维护者的操作手册
├── index.html          # 单页应用（四视图），唯一对外入口
├── data/               # 所有数据文件，日常维护核心
│   ├── config.json     # 全局配置（每期必改）
│   ├── insights.json   # 访谈洞察（每期在数组头部插入）
│   ├── voc.csv         # VoC 大表（持续追加行）
│   ├── journey.json    # 旅程阶段定义（季度检查）
│   ├── time_cost.csv   # 耗时时间序列（每期追加行）
│   ├── score.csv       # 主观打分（保留加载，当前页面未直接展示）
│   └── personas.json   # 开发者画像（保留加载，当前页面未展示）
└── assets/             # 图片资源
```

**黄金规则：日常只改 `data/` 目录，不动 `index.html`（除非明确要改网页功能）。**

---

## 网页结构（index.html · 一个首页 + 三个子视图）

纯 vanilla JS 单页应用，顶部导航在四个视图间切换。无构建步骤，无框架依赖
（仅用 PapaParse 解析 CSV、Chart.js 画趋势图）。

| 视图 | 内容 | 主要数据来源 |
|------|------|--------------|
| **首页** | 北极星 HERO（全流程耗时大数 + 趋势）、问题闭环概览（待处理 Top5 / 进行中 / 已解决 / 闭环率）、内嵌旅程流程 | time_cost.csv、config.json、voc.csv、journey.json |
| **旅程全景** | 10 阶段横向流程，点击展开耗时趋势 / Top VoC / 关联洞察 | journey.json、time_cost.csv、voc.csv、insights.json |
| **领域问题板** | 问题列表 + 四维筛选（状态/旅程阶段/问题分类/优先级），可展开看进展 | voc.csv |
| **洞察归档** | 按期次倒序的洞察卡片，展开看核心发现、原话、关联阶段 | insights.json、config.json |

### 数据如何驱动页面（重要）
- **北极星大数与趋势** ← `time_cost.csv` 的 `full_process_days`（峰值/当前/累计降幅自动算）
- **本期焦点 + 一句话结论** ← `config.json` 的 `spotlight` 与 `headline`
- **问题闭环计数与闭环率** ← `voc.csv` 的 `status` 计数（已解决 / 总数）
  - ⚠️ 若 voc.csv 当前没有「已解决」记录，首页闭环率会显示 **0%** —— 这是真实数据信号，不是 bug
- **领域问题板每条问题** ← 直接来自 `voc.csv`（精简版：用 `developer_id` 当负责人、`priority` 排序与筛选；无 votes/owner/多条时间线概念）
- **各阶段问题数** ← `journey.json` 的 `open_issues`
- **阶段关联洞察** ← `insights.json` 中 `stages_affected` 命中该阶段的洞察
- `personas.json`、`score.csv` 仍被 `loadAll()` 加载，但当前四视图未展示

---

## 数据文件规范

### config.json
控制首页核心信息，每期必改。

```json
{
  "project": "openUBMC 开发者体验全景",
  "current_period": "2026-Q1",         // 格式：YYYY-QN
  "last_updated": "2026-03-31",        // 格式：YYYY-MM-DD
  "maintainers": ["体验管理团队"],
  "spotlight": {
    "stage": "安装部署",               // 必须是合法 journey_stage 值（决定首页/旅程的焦点高亮）
    "reason": "本期耗时上升明显，是当前最大瓶颈"
  },
  "headline": "本期一句话结论"          // 显示在北极星 HERO 结论区
}
```

### insights.json
数组，最新的一期放在 **index 0**（数组最前面）。当期由 `period === config.current_period` 自动判定并高亮。

```json
{
  "id": "insight_2026Q1",              // 格式：insight_YYYYQN
  "period": "2026-Q1",
  "date": "2026-03-31",
  "title": "本期洞察标题（一句话）",
  "findings": ["发现1", "发现2", "发现3"],   // 第一条会自动用作折叠态摘要
  "quotes": [                          // 1-2条开发者原话，显示在展开区
    { "content": "原话内容", "stage": "安装部署", "developer_id": "dev_xxx" }
  ],
  "stages_affected": ["安装部署", "维护"]  // 合法 journey_stage 值数组
}
```

### voc.csv
只追加，不修改历史行。**领域问题板的全部内容直接来自这张表。**

```
id,date,period,source,developer_id,journey_stage,content,category,priority,status
```

- `id`：格式 `voc_NNN`，自动递增
- `date`：格式 `YYYY-MM-DD`
- `period`：格式 `YYYY-QN`
- `source`：`论坛` / `访谈` / `VoC问卷` / `issue`
- `journey_stage`：必须是合法值（见下方），否则无法关联到旅程阶段
- `category`：`文档问题` / `工具问题` / `流程问题` / `平台体验` / `信息不透明` / `环境问题` / `支持问题`（问题板「问题分类」筛选项即来自此列的去重值）
- `priority`：`高` / `中` / `低`（决定首页待处理 Top5 排序与问题板「优先级」筛选）
- `status`：`已解决` / `进行中` / `待处理`（驱动闭环率；改这一列即可更新进展）

### time_cost.csv
每期追加一行，列顺序固定。

```
period,date,full_process_days,learn,requirement,deploy,hardware,component,interface,build,test,maintain
```

- `full_process_days`：全流程完成耗时（天）→ 北极星大数与趋势
- 其余列：各 L1 阶段耗时（天，可含小数）→ 旅程阶段详情的耗时趋势
- 注意：**无 feedback 列**，「反馈交流」阶段耗时显示为「—」

### score.csv
每期追加一行，满分 5 分。当前页面未直接展示，但保留加载。

```
period,date,learn,requirement,deploy,hardware,component,interface,build,test,maintain,feedback
```

### journey.json
每个阶段维护 `open_issues`（→ 阶段问题数）和 `resolved_issues`，季度检查时更新。

### personas.json
开发者画像，季度检查。当前四视图未展示，但保留加载。

---

## 合法 journey_stage 值

网页、VoC、洞察中涉及阶段名称，必须从以下列表取值（完全一致，包括空格）：

```
了解学习 / 需求分析 / 安装部署 / 硬件适配 / 组件开发 / 接口定制 / 构建 / 测试 / 维护 / 反馈交流
```

---

## 合法 status 值

```
已解决 / 进行中 / 待处理
```

---

## 常见操作指令示例

| 用户说 | 你需要做 |
|---|---|
| "更新本期数据，期次是2026-Q1" | 更新 config.json 期次和日期 |
| "新增一条VoC" | 在 voc.csv 末尾追加一行（会自动出现在问题板）|
| "写本期洞察" | 在 insights.json 数组头部插入新条目 |
| "更新本期耗时" | 在 time_cost.csv 末尾追加一行 |
| "把安装部署的某个VoC标为已解决" | 找到对应行，修改 status 字段（影响闭环率）|
| "本期焦点改为维护阶段" | 修改 config.json 的 spotlight |
| "网页加一个功能" | 修改 index.html，操作前先说明方案 |

---

## Git 操作规范（默认走 PR）

**不直接推 main。** 每次改动流程：

```
1. 新建分支    git checkout -b <type>/<简述>     # 如 update/2026-Q1-data、feat/board-sort
2. 提交        git commit -m "<type>: 说明"
3. 推分支      git push origin <分支名>
4. 开 PR       打开 GitHub 给出的 compare 链接创建 Pull Request，评审/预览后合并到 main
5. 合并后 GitHub Pages 自动部署
```

commit message 前缀：`update:` / `feat:` / `fix:` / `docs:`

> 例外：仅当用户**明确说「直接推 main」**时才跳过 PR。
> 远端为 HTTPS（`https://github.com/wanwan0722/openubmc-dx-hub-2.git`），凭据走 macOS 钥匙串。

---

## 注意事项

1. **不要改变 CSV 的列顺序**，网页按固定列名读取
2. **insights.json 最新期放 index 0**，并保证某条 `period` 等于 config.current_period（决定「当期」高亮）
3. **journey_stage 值必须完全一致**，包括空格，否则 VoC 无法关联到旅程阶段
4. **不要删除历史数据**，耗时趋势图依赖完整时间序列
5. 本地预览：`npx serve .`（已配在 `.claude/launch.json`）；`python3 -m http.server 8080` 在部分沙箱环境有权限限制
