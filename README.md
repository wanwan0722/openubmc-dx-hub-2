# openUBMC 开发者体验管理平台

体验管理知识库，部署在 GitHub Pages 上，通过固定链接向领导传播。

## 目录结构

```
openubmc-dx-hub/
├── index.html          # 单页应用（四视图，唯一对外入口，一般不需改）
├── data/               # 所有数据文件（日常维护这里）
│   ├── config.json     # 全局配置：当前期次、焦点、一句话结论
│   ├── personas.json   # 开发者画像（保留加载，当前页面未展示）
│   ├── journey.json    # 旅程阶段定义 + 问题数统计
│   ├── voc.csv         # VoC 大表（持续追加行；领域问题板直接读这张表）
│   ├── insights.json   # 访谈洞察（每期追加一条，最新的放第一位）
│   ├── time_cost.csv   # 耗时时间序列（每期追加一行）
│   └── score.csv       # 主观打分（保留加载，当前页面未直接展示）
└── assets/             # 图片资源
```

## 网页有什么（一个首页 + 三个子视图）

顶部导航切换四个视图，全部由 `data/` 里的文件实时驱动：

| 视图 | 看什么 | 数据来源 |
|------|--------|----------|
| **首页** | 全流程耗时大数 + 趋势、问题闭环概览（待处理 Top5 / 进行中 / 已解决 / 闭环率）、内嵌旅程流程 | time_cost.csv、config.json、voc.csv、journey.json |
| **旅程全景** | 10 个阶段的耗时与问题数，点开看趋势 / Top VoC / 关联洞察 | journey.json、time_cost.csv、voc.csv、insights.json |
| **领域问题板** | 全部 VoC 列表 + 状态/阶段/分类/优先级四维筛选 | voc.csv |
| **洞察归档** | 按期次倒序的洞察，展开看 3 条核心发现、原话、关联阶段 | insights.json、config.json |

> 提示：首页「闭环率」= voc.csv 里「已解决」条数 ÷ 总条数。
> 想让闭环率有数据，把对应 VoC 的 `status` 改成「已解决」即可。

---

## 每月更新 SOP

### 必改（配合汇报节奏）

**1. config.json** — 更新期次、焦点和结论
```json
{
  "current_period": "2026-Q1",
  "last_updated": "2026-03-31",
  "spotlight": { "stage": "安装部署", "reason": "本期耗时上升明显" },
  "headline": "本期一句话结论写在这里"
}
```

**2. insights.json** — 在数组最前面插入新一期洞察
```json
{
  "id": "insight_2026Q1",
  "period": "2026-Q1",
  "date": "2026-03-31",
  "title": "本期洞察标题",
  "findings": ["发现1", "发现2", "发现3"],
  "quotes": [ { "content": "开发者原话", "stage": "阶段名", "developer_id": "dev_xxx" } ],
  "stages_affected": ["安装部署", "维护"]
}
```

**3. time_cost.csv** — 追加一行耗时数据
```
2026-Q1,2026-03-31,31,1.0,2.5,10.0,7.0,4.5,3.0,1.0,3.0,2.5
```
列顺序：period, date, full_process_days, learn, requirement, deploy, hardware, component, interface, build, test, maintain

**4. score.csv** — 追加一行打分（可选）
```
2026-Q1,2026-03-31,3.7,3.1,2.7,3.3,3.2,2.9,4.1,3.2,2.6,2.8
```

---

### 选改（有新反馈时）

**voc.csv** — 追加新 VoC 行（会自动出现在领域问题板）
- `status` 字段值：`已解决` / `进行中` / `待处理`
- `priority` 字段值：`高` / `中` / `低`（决定首页待处理 Top5 排序）
- `journey_stage` 必须与 journey.json 中的 label 完全一致

**journey.json** — 更新问题数量统计
- 每期核对 `open_issues` 和 `resolved_issues` 是否准确

---

### 季度检查

- 评估 `personas.json` 画像是否需要更新
- 检查 `journey.json` 各阶段问题状态

---

## 本地预览

```bash
npx serve .
# 然后打开终端里给出的本地地址（默认 http://localhost:3000）
```

> `python3 -m http.server 8080` 也可用，但部分受限环境下会报权限错误，推荐用 `npx serve`。

---

## 提交与部署（默认走 PR）

**不直接推 main。** 改完数据后：

```bash
git checkout -b update/2026-Q1-data      # 1. 新建分支
git add -A && git commit -m "update: 2026-Q1 数据更新"   # 2. 提交
git push origin update/2026-Q1-data      # 3. 推分支
# 4. 打开 GitHub 给出的链接，创建 Pull Request，确认后合并到 main
```

合并到 `main` 后，GitHub Pages 自动重新部署，约 1-2 分钟后对外链接更新。

> 首次配置 GitHub Pages：仓库 Settings → Pages → Source 选 `main` 分支、目录 `/(root)`。
> 远端地址为 HTTPS（`https://github.com/wanwan0722/openubmc-dx-hub-2.git`）。

---

## 字段规范

### journey_stage 合法值（必须与此一致）
```
了解学习 / 需求分析 / 安装部署 / 硬件适配 /
组件开发 / 接口定制 / 构建 / 测试 / 维护 / 反馈交流
```

### voc.csv status 合法值
```
已解决 / 进行中 / 待处理
```

### voc.csv source 合法值（建议统一）
```
论坛 / 访谈 / VoC问卷 / issue
```
