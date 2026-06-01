# 项目说明：openUBMC 开发者体验全景

本项目是 openUBMC 开源社区的体验管理知识库，部署在 GitHub Pages 上，
通过固定链接向管理层传播。维护者为 2-3 人小团队。

---

## 你的角色

你是这个项目的协作维护者。每次对话时：
1. 先理解用户要做什么（更新数据 / 修改网页 / 新增功能）
2. 操作前说明你要改哪些文件、改什么
3. 操作完告知是否需要 commit & push

---

## 项目结构

```
openubmc-dx-hub/
├── CLAUDE.md           # 本文件，项目说明
├── README.md           # 面向维护者的操作手册
├── index.html          # 全景网页，唯一对外入口
├── data/               # 所有数据文件，日常维护核心
│   ├── config.json     # 全局配置（每期必改）
│   ├── insights.json   # 访谈洞察（每期在数组头部插入）
│   ├── voc.csv         # VoC 大表（持续追加行）
│   ├── journey.json    # 旅程阶段定义（季度检查）
│   ├── time_cost.csv   # 耗时时间序列（每期追加行）
│   ├── score.csv       # 主观打分（每期追加行，辅助参考）
│   └── personas.json   # 开发者画像（季度检查）
└── assets/             # 图片资源（旅程图、画像图等PNG）
```

**黄金规则：日常只改 `data/` 目录，不动 `index.html`（除非明确要改网页功能）。**

---

## 数据文件规范

### config.json
控制网页第一屏的核心信息，每期必改。

```json
{
  "project": "openUBMC 开发者体验全景",
  "current_period": "2026-Q1",         // 格式：YYYY-QN
  "last_updated": "2026-03-31",        // 格式：YYYY-MM-DD
  "maintainers": ["体验管理团队"],
  "spotlight": {
    "stage": "安装部署",               // 必须是合法 journey_stage 值
    "reason": "本期耗时上升明显，是当前最大瓶颈"
  },
  "headline": "本期一句话结论"
}
```

---

### insights.json
数组，最新的一期放在 **index 0**（数组最前面）。

```json
{
  "id": "insight_2026Q1",              // 格式：insight_YYYYQN
  "period": "2026-Q1",
  "date": "2026-03-31",
  "title": "本期洞察标题（一句话）",
  "findings": [                        // 固定3条
    "发现1",
    "发现2",
    "发现3"
  ],
  "quotes": [                          // 1-2条开发者原话
    {
      "content": "原话内容",
      "stage": "安装部署",             // 合法 journey_stage 值
      "developer_id": "dev_xxx"
    }
  ],
  "stages_affected": ["安装部署", "维护"]  // 合法 journey_stage 值数组
}
```

---

### voc.csv
只追加，不修改历史行。每次新增反馈从末尾追加。

```
id,date,period,source,developer_id,journey_stage,content,category,priority,status
```

**字段规范：**
- `id`：格式 `voc_NNN`，自动递增
- `date`：格式 `YYYY-MM-DD`
- `period`：格式 `YYYY-QN`
- `source`：`论坛` / `访谈` / `VoC问卷` / `issue`
- `journey_stage`：必须是合法值（见下方）
- `category`：`文档问题` / `工具问题` / `流程问题` / `平台体验` / `信息不透明` / `环境问题` / `支持问题`
- `priority`：`高` / `中` / `低`
- `status`：`已解决` / `进行中` / `待处理`

---

### time_cost.csv
每期追加一行，列顺序固定。

```
period,date,full_process_days,learn,requirement,deploy,hardware,component,interface,build,test,maintain
```

- `full_process_days`：全流程完成耗时（天，可含小数）
- 其余列：各 L1 阶段耗时（天，可含小数，暂无数据填 0）

---

### score.csv
每期追加一行，辅助参考，非核心指标。满分 5 分。

```
period,date,learn,requirement,deploy,hardware,component,interface,build,test,maintain,feedback
```

---

### journey.json
每个阶段维护 `open_issues` 和 `resolved_issues` 两个数字，季度检查时更新。

---

### personas.json
开发者画像，季度检查是否需要新增或修改画像。

---

## 合法 journey_stage 值

网页、VoC、洞察中涉及阶段名称，必须从以下列表取值（完全一致）：

```
了解学习
需求分析
安装部署
硬件适配
组件开发
接口定制
构建
测试
维护
反馈交流
```

---

## 常见操作指令示例

用户可能这样说，你对应需要做的事：

| 用户说 | 你需要做 |
|---|---|
| "更新本期数据，期次是2026-Q1" | 更新 config.json 期次和日期 |
| "新增一条VoC" | 在 voc.csv 末尾追加一行 |
| "写本期洞察" | 在 insights.json 数组头部插入新条目 |
| "更新本期耗时" | 在 time_cost.csv 末尾追加一行 |
| "把安装部署的某个VoC标为已解决" | 找到对应行，修改 status 字段 |
| "本期焦点改为维护阶段" | 修改 config.json 的 spotlight |
| "网页加一个功能" | 修改 index.html，操作前先说明方案 |

---

## Git 操作规范

每次数据更新后，建议 commit message 格式：

```
update: 2026-Q1 数据更新
update: 新增3条VoC（安装部署阶段）
update: 本期洞察和耗时数据
fix: 修正voc_021状态为已解决
```

push 到 main 分支后，GitHub Pages 自动更新，无需其他操作。

---

## 注意事项

1. **不要改变 CSV 的列顺序**，网页按固定列名读取
2. **insights.json 最新期放 index 0**，网页默认展示第一条
3. **journey_stage 值必须完全一致**，包括空格，否则 VoC 无法关联到旅程阶段
4. **不要删除历史数据**，耗时趋势图依赖完整时间序列
5. 本地预览需要启动本地服务器：`python3 -m http.server 8080`，然后访问 `http://localhost:8080`
