# analysis-skills

A股研究与金融分析 Claude Code 技能包。`stock-report` 是流水线指挥官，自动调用其余技能完成数据收集、来源验证、竞争格局、分析校验、四视角辩论，最终生成 8 页交互式 HTML 报告并推送 GitHub Pages。

---

## 技能列表

本仓库包含 3 个技能，另需 2 个 ECC 内置技能作为依赖：

| 技能 | 来源 | 定位 | 核心能力 |
|------|------|------|---------|
| [`stock-report`](./skills/stock-report/SKILL.md) | 本仓库 | 流水线指挥官 | 编排所有技能，生成报告，推送 GitHub Pages |
| [`research-triangulation`](./skills/research-triangulation/SKILL.md) | 本仓库 | 数据准备 | A股来源可靠性分级（L1-L6）、三源验证、矛盾裁决 |
| [`analytical-rigor`](./skills/analytical-rigor/SKILL.md) | 本仓库 | 分析执行 | 幸存者偏误扫描、敏感性分析、置信度三档标注 |
| `market-research` | ECC | 竞争情报 | 行业竞争格局、TAM/CAGR、护城河评估 |
| `council` | ECC | 质疑防线 | Architect/Skeptic/Pragmatist/Critic 四视角辩论 |

### 流水线（stock-report 编排顺序）

```
/stock-report 600519 贵州茅台
        ↓
Step 1    并行5组搜索（基本面 / 业绩 / 评级 / 利空 / 利好）
        ↓
Step 1.2  [market-research]  行业TOP3-5竞争对手、TAM、集中度、护城河
        ↓
Step 1.5  [research-triangulation]  数据来源L1-L6分层验证，三源交叉确认
        ↓
Step 2    多空各5条论证 + 5情景估值（触发条件替代裸概率）
        ↓
Step 2.5  [analytical-rigor]  幸存者偏误校正、外推限制、关键假设识别
        ↓
Step 3    [council]  四视角辩论，综合裁决替代单一星级评分
        ↓
Step 4    生成 8 页交互式 HTML 报告
        ↓
Step 5    推送 GitHub Pages，返回链接
```

---

## 安装

### 前置要求

- [Claude Code](https://claude.ai/download)（Pro 订阅）
- Git
- ECC（提供 `market-research` 和 `council` 两个依赖技能）

### 第一步：安装 ECC（获取 market-research 和 council）

```bash
git clone https://github.com/affaan-m/everything-claude-code.git
cd everything-claude-code

# Mac / Linux
cp -r skills/market-research ~/.claude/skills/market-research
cp -r skills/council         ~/.claude/skills/council

# Windows（PowerShell）
Copy-Item -Recurse "skills\market-research" "$env:USERPROFILE\.claude\skills\market-research"
Copy-Item -Recurse "skills\council"         "$env:USERPROFILE\.claude\skills\council"
```

> 如果已装过完整 ECC，跳过此步。

### 第二步：安装本仓库的 3 个技能

```bash
git clone https://github.com/mingchengdeng32-wq/analysis-skills.git
cd analysis-skills

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

### 第三步：重启 Claude Code

重启后在技能列表里看到 `stock-report`、`research-triangulation`、`analytical-rigor`、`market-research`、`council` 共 5 个名字即表示安装成功。

---

## 使用方式

### 分析一只股票（全流水线自动触发）

```
/stock-report 600519 贵州茅台
/stock-report 300750 宁德时代 持仓20万 持仓90天
```

或者直接说，Claude 会自动识别：

```
帮我分析一下贵州茅台
研究下宁德时代，我持仓20万，想拿一个季度
```

### 单独使用各层

```
# 来源验证
帮我验证这些数据来源是否可靠
核实一下这几个数字的出处

# 分析严谨性检查
检查一下这个分析结论有没有幸存者偏误
帮我做一下敏感性分析，看哪个假设最关键

# 市场研究
帮我研究一下医疗健康行业的竞争格局
分析一下新能源行业的主要玩家
```

---

## 技能说明

### stock-report（流水线指挥官）

一条命令完成所有环节：数据收集 → 竞争格局 → 来源验证 → 分析校验 → 四视角辩论 → 8页HTML → GitHub Pages。

报告结构：封面 → 公司全景（含竞争对手）→ 多头5条 → 空头5条 → Council四声音 → 估值触发条件 → 投资结论 → 回报测算

示例报告和设计系统：[mingchengdeng32-wq/kin-g-signal](https://github.com/mingchengdeng32-wq/kin-g-signal)

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

### market-research（ECC 内置）

**在 stock-report 中的作用**：Step 1.2 竞争格局分析，收集行业TOP3-5竞争对手、TAM/CAGR、集中度趋势、护城河评估。结果直接用于多空论证数据支撑和 council 输入上下文。

### council（ECC 内置）

**在 stock-report 中的作用**：Step 3 四视角辩论，替代单一星级评分。Skeptic 角色不看前序分析过程，直接挑战前提假设，是整条流水线的最后防线。

---

## 设计原则

1. **流水线而非工具箱** — 技能有顺序依赖，stock-report 是唯一入口
2. **防御性优先** — 每个技能优先处理已知的失败模式，而非扩展功能
3. **主动找反证** — 好的分析不是找更多支持自己的证据，而是找能推翻自己的证据
4. **置信度显式化** — 任何结论都必须标注是数据证实、合理推断还是主观判断
5. **竞争格局前置** — 护城河和市占数据在多空论证之前收集，不是事后补充

---

## License

MIT — 自由使用和修改。

内容仅供研究参考，不构成投资建议。股市有风险，入市需谨慎。
