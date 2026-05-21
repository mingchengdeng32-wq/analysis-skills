# analysis-skills

A股研究与金融分析 Claude Code 技能包。三个技能构成一条完整的分析流水线：
先验证数据来源可靠性，再检查分析过程的统计严谨性，最后生成标准化的研究报告。

---

## 技能列表

| 技能 | 定位 | 核心能力 |
|------|------|---------|
| [`research-triangulation`](./skills/research-triangulation/SKILL.md) | 第1层：数据准备 | A股来源可靠性分级、三源验证、矛盾信息裁决 |
| [`analytical-rigor`](./skills/analytical-rigor/SKILL.md) | 第2层：分析执行 | 幸存者偏误扫描、敏感性分析、置信度三档标注 |
| [`stock-report`](./skills/stock-report/SKILL.md) | 第3层：输出成型 | 8页交互式HTML幻灯片，自动推送 GitHub Pages |

### 流水线关系

```
用户输入股票代码
      ↓
[research-triangulation]  验证数据来源层级，完成三角验证
      ↓
[analytical-rigor]        扫描统计陷阱，标注置信度，识别关键假设
      ↓
[stock-report]            生成报告，推送 GitHub Pages，返回链接
      ↓
（可选）[council]         对结论存疑时，触发4视角辩论作为最后防线
```

> `council` 是 ECC 内置技能，需单独安装：[affaan-m/everything-claude-code](https://github.com/affaan-m/everything-claude-code)

---

## 安装

### 前置要求

- [Claude Code](https://claude.ai/download)（Pro 订阅）
- Git

### 安装步骤

```bash
# 1. clone 本仓库
git clone https://github.com/mingchengdeng32-wq/analysis-skills.git
cd analysis-skills

# 2. 复制技能到 Claude Code 技能目录
# Mac / Linux
mkdir -p ~/.claude/skills/research-triangulation
mkdir -p ~/.claude/skills/analytical-rigor
mkdir -p ~/.claude/skills/stock-report
cp skills/research-triangulation/SKILL.md ~/.claude/skills/research-triangulation/SKILL.md
cp skills/analytical-rigor/SKILL.md        ~/.claude/skills/analytical-rigor/SKILL.md
cp skills/stock-report/SKILL.md            ~/.claude/skills/stock-report/SKILL.md

# Windows（PowerShell）
New-Item -ItemType Directory -Force "$env:USERPROFILE\.claude\skills\research-triangulation"
New-Item -ItemType Directory -Force "$env:USERPROFILE\.claude\skills\analytical-rigor"
New-Item -ItemType Directory -Force "$env:USERPROFILE\.claude\skills\stock-report"
Copy-Item "skills\research-triangulation\SKILL.md" "$env:USERPROFILE\.claude\skills\research-triangulation\SKILL.md"
Copy-Item "skills\analytical-rigor\SKILL.md"        "$env:USERPROFILE\.claude\skills\analytical-rigor\SKILL.md"
Copy-Item "skills\stock-report\SKILL.md"            "$env:USERPROFILE\.claude\skills\stock-report\SKILL.md"
```

安装完成后重启 Claude Code，在技能列表里看到这三个名字即表示成功。

---

## 使用方式

### 分析一只股票（三层流水线自动触发）

```
/stock-report 600519 贵州茅台
```

Claude 会依次调用三个技能，完成数据验证 → 分析校验 → 报告生成，最后返回 GitHub Pages 链接。

### 单独使用验证层

```
帮我验证这些数据来源是否可靠
核实一下这几个数字的出处
```

### 单独使用分析严谨性检查

```
检查一下这个分析结论有没有幸存者偏误
帮我做一下敏感性分析，看哪个假设最关键
```

---

## 技能说明

### research-triangulation（研究三角验证）

**解决的核心问题**：A股卖方研究存在结构性偏多倾向，券商依赖上市公司投行业务，负面评级影响商业关系。技能内置了本土化的来源可靠性分级（L1 法律强制披露 → L6 社交媒体），强制要求关键数据至少有两个独立来源确认。

**关键设计**：矛盾信息不是错误，而是信号。两个来源说法不同时，追问"为什么会有这个差异"往往比数据本身更有价值。

### analytical-rigor（分析严谨性）

**解决的核心问题**：A股分析中最危险的不是方法错误，而是隐性的认知偏误。

五项强制检查：
1. **幸存者偏误扫描** — 主动搜索退市/ST案例，把失败案例纳入参照系
2. **近期外推限制** — 少于3个数据点的趋势必须降级标注
3. **敏感性分析** — 只找那一个"如果错了结论就翻转"的关键假设
4. **置信度三档** — 数据证实 / 合理推断 / 主观判断，禁止混用语气
5. **反向证据搜索** — 结论形成后主动找能推翻它的数据

**与 council 的协作**：当 analytical-rigor 检测到结论极度依赖单一假设或存在未解决的重大矛盾时，升级至 council 的 Skeptic 角色处理。Skeptic 是一个独立 subagent，看不到你的分析过程，直接挑战前提假设。

### stock-report（A股研究报告）

生成8页标准化交互式 HTML 报告，推送 GitHub Pages。

报告结构：封面 → 公司全景 → 多头5条 → 空头5条 → PK评分 → 估值模型 → 投资结论 → 回报测算

详见 [mingchengdeng32-wq/kin-g-signal](https://github.com/mingchengdeng32-wq/kin-g-signal) 仓库，内含完整设计系统和示例报告。

---

## 设计原则

这三个技能的共同原则：

1. **流水线而非工具箱** — 三个技能有顺序依赖，不是随机调用的独立工具
2. **防御性优先** — 每个技能优先处理已知的失败模式，而非扩展功能
3. **主动找反证** — 好的分析不是找更多支持自己的证据，而是找能推翻自己的证据
4. **置信度显式化** — 任何结论都必须标注是数据证实、合理推断还是主观判断

---

## License

MIT — 自由使用和修改。

内容仅供研究参考，不构成投资建议。股市有风险，入市需谨慎。
