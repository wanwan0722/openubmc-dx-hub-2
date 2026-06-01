# openUBMC 开发者体验全景

体验管理知识库，部署在 GitHub Pages 上，通过固定链接向领导传播。

## 目录结构

```
openubmc-dx-hub/
├── index.html          # 全景网页（唯一对外入口，不需要改动）
├── data/               # 所有数据文件（日常维护这里）
│   ├── config.json     # 全局配置：当前期次、焦点、一句话结论
│   ├── personas.json   # 开发者画像
│   ├── journey.json    # 旅程阶段定义 + 问题数统计
│   ├── voc.csv         # VoC 大表（持续追加行）
│   ├── insights.json   # 访谈洞察（每期追加一条，最新的放第一位）
│   ├── time_cost.csv   # 耗时时间序列（每期追加一行）
│   └── score.csv       # 主观打分（每期追加一行，辅助参考）
└── assets/             # 图片资源（旅程图、画像图等）
```

---

## 每月更新 SOP

### 必改（配合汇报节奏）

**1. config.json** — 更新期次、焦点和结论
```json
{
  "current_period": "2026-Q1",
  "last_updated": "2026-03-31",
  "spotlight": {
    "stage": "安装部署",
    "reason": "本期耗时上升明显"
  },
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
  "quotes": [
    { "content": "开发者原话", "stage": "阶段名", "developer_id": "dev_xxx" }
  ],
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

**voc.csv** — 追加新 VoC 行
- status 字段值：`已解决` / `进行中` / `待处理`
- journey_stage 必须与 journey.json 中的 label 完全一致

**journey.json** — 更新问题数量统计
- 每期核对 `open_issues` 和 `resolved_issues` 是否准确

---

### 季度检查

- 评估 `personas.json` 画像是否需要更新
- 检查 `journey.json` 各阶段问题状态

---

## 部署到 GitHub Pages

1. 将整个文件夹推送到 GitHub 仓库
2. 进入仓库 Settings → Pages
3. Source 选择 `main` 分支，目录选 `/（root）`
4. 保存后获得固定访问链接

每次更新数据文件后，推送到 GitHub，页面自动刷新。

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
