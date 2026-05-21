---
name: stock-report
description: 生成A股个股深度研究报告（多空PK + 估值模型 + 投资回报测算），输出8页交互式HTML幻灯片并自动推送到 GitHub Pages。输入股票代码和名称即可一键完成。触发词：分析股票、/stock-report、研究报告、多空分析。
---

# A股深度研究报告 · 通用技能

## 触发方式

```
/stock-report <股票代码> <股票名称>
/stock-report <股票代码> <股票名称> 持仓<金额> 持仓<天数>天
```

示例：
```
/stock-report 600519 贵州茅台
/stock-report 300750 宁德时代 持仓20万 持仓90天
/stock-report 000858 五粮液 持仓5万
```

用户说「帮我分析一下 600519」「研究下宁德时代」也触发本技能。

---

## Step 0：首次运行 · 自动配置（只做一次）

**每次运行前，先检查配置文件是否存在：**

```
~/.claude/skills/stock-report/config.md
```

### 如果配置文件不存在 → 执行配置向导

依次完成以下3步，然后把结果写入配置文件：

**① 找到本地报告仓库**

按优先级检测：
1. 当前工作目录下是否有 `_DESIGN_SYSTEM.md`（报告仓库标志文件）
2. 当前工作目录下是否有 `index.html` + `.git`
3. 用户主目录下是否有 `kin-g-signal` 文件夹
4. 以上都没有 → 询问用户：
   > 「请告诉我你存放报告的本地目录路径，例如：
   > Windows: D:\my-stocks
   > Mac/Linux: ~/stock-reports」

**② 检测 GitHub Pages URL**

进入仓库目录，运行：
```bash
git -C {repo_path} remote get-url origin
```

从 remote URL 推导 GitHub Pages 地址：
- `https://github.com/username/repo-name` → `https://username.github.io/repo-name`
- `git@github.com:username/repo-name.git` → `https://username.github.io/repo-name`

如果无法检测，询问用户：
> 「你的 GitHub Pages 地址是什么？（例如：https://yourname.github.io/stock-reports）」

**③ 写入配置文件**

创建 `~/.claude/skills/stock-report/config.md`：

```markdown
# stock-report 配置
repo_path: {检测到的本地路径}
pages_url: {GitHub Pages URL}
configured_at: {当前日期}
```

配置完成后告知用户：
> ✅ 配置完成，已保存到 ~/.claude/skills/stock-report/config.md
> 后续运行无需重复设置。

### 如果配置文件存在 → 直接读取，跳过向导

---

## Step 1：并行收集数据（5组同时搜索）

以下搜索**全部并行发出**，不要顺序执行：

1. `{股票名称} {代码} 2025 2026 券商研究报告 基本面分析`
2. `{股票名称} {代码} 2026 营收 净利润 业绩 一季报`
3. `{股票名称} {代码} 机构评级 目标价 买入 2026`
4. `{股票名称} {代码} 风险 利空 竞争 行业压力 2026`
5. `{股票名称} {代码} 利好 催化剂 政策 2026 下半年`

必要时追加：
- `{股票名称} 股价 资金流向 2026年5月`
- `{行业} 竞争格局 市占率 2026`

**必须收集（找不到则标注「暂无公开数据」）：**

| 数据项 | 备注 |
|--------|------|
| 当前股价 | 最新收盘价 |
| 机构目标均价 | 近90天共识 |
| 2025全年营收及增速 | YoY |
| 2025净利润及增速 | 归母净利 |
| 最新季报营收 | Q1 2026或最新 |
| 核心业务/新业务收入 | AI/新能源/出口等 |
| 经营现金流 | 2025年 |
| 机构评级分布 | 买入/增持/中性数量 |
| 近期主力资金流向 | 最近3-5个交易日 |
| 主要财务风险项 | 应收/债务/减值等 |

---

## Step 2：分析框架

### 默认持仓参数（用户未指定时使用）
- 持仓金额：20万人民币
- 持仓周期：90天
- 止损比例：-15%

用户指定的参数覆盖默认值。

### 多头论证（必涨5条）
每条包含：
- 标题（≤15字，有冲击力）
- 数据依据（2行，具体数字）
- 强度标签（如：AI +71%、政策加码、CR1 >20%）

### 空头论证（必跌5条）
同上格式，使用警示性标签。

### PK评分（6维度）
推荐维度（可根据行业适当调整）：

| 维度 | 多头侧重 | 空头侧重 |
|------|---------|---------|
| 成长性 | 新业务增速、订单 | 主业下滑、增速放缓 |
| 盈利质量 | 净利含金量、现金流 | 应收膨胀、补贴依赖 |
| 估值合理性 | PEG、行业对比 | 绝对PE偏高 |
| 行业格局 | 市占率、护城河 | 竞争加剧 |
| 资金动向 | 机构持仓增加 | 主力持续出逃 |
| 财务安全 | 资产负债率低 | 债务/减值风险 |

每维度多头/空头各1-5星，统计胜/负/平，输出总比分。

### 估值模型（5情景）
基于行业平均PE和公司预测利润：

| 情景 | 净利润假设 | PE倍数 | 颜色 |
|------|-----------|--------|------|
| 梦想 | 2028E高点 | 行业高端 | bull绿 |
| 乐观 | 2027E | 偏高 | gold金 |
| 基准 | 机构共识2026E | 当前 | ice蓝 |
| 保守 | 2026E下修 | 中等 | amber橙 |
| 悲观 | 继续恶化 | 收缩 | bear红 |

当前价在条形图上用竖线标注，max = 最高目标价的110%取整。

### 回报测算（4情景）
根据 Step 2 开头的持仓参数计算：
- 持股数量 = 持仓金额 / 当前股价
- 4个情景：强烈看涨 / 基准看涨 / 震荡横盘 / 下行止损
- 输出：盈亏金额、概率估计、期望收益（EV）、风险回报比

---

## Step 3：生成 HTML 文件

### 文件命名
- 优先使用股票名称拼音缩写（贵州茅台→maotai，宁德时代→catl，五粮液→wuliangye）
- 如无法确定，使用股票代码（600519.html）

### 输出路径
```
{config.repo_path}/{文件名}.html
```

### 设计规范（必须严格遵守）

**设计参考文件读取顺序：**
1. `{config.repo_path}/_DESIGN_SYSTEM.md`（主参考，含完整组件规范）
2. `{config.repo_path}/meinian.html`（实现参考，直接复用CSS）

**核心CSS变量（必须原样复用，不得修改）：**
```css
--bg: #07080F;  --gold: #C9A347;  --bull: #15B77D;
--bear: #E04455; --ice: #4A9EDB;  --amber: #F5A623;
--text: #E2D9CC; --serif: 'Noto Serif SC', serif;
--mono: 'JetBrains Mono', monospace;
```

**字体引入（必须包含）：**
```html
<link href="https://fonts.googleapis.com/css2?family=Noto+Serif+SC:wght@400;600;700&family=Noto+Sans+SC:wght@300;400;500;700&family=JetBrains+Mono:wght@400;600&display=swap" rel="stylesheet">
```

### 8页幻灯片结构

| 编号 | 内容 | 关键组件 |
|------|------|---------|
| 0 | 封面 | cover-stats（5项：评级/价格/目标/买点/核心矛盾） |
| 1 | 公司全景 | grid-4指标卡 + SVG柱状图 + 业务矩阵进度条 |
| 2 | 多头 | reason-row.bull-row × 5 + slideIn动画 |
| 3 | 空头 | reason-row.bear-row × 5 |
| 4 | PK | pk-table + star-bar星级 + pk-winner badge + 得分卡 |
| 5 | 估值 | val-row × 5 + current-line当前价竖线 + 情景表 + 买点卡 |
| 6 | 结论 | rating-badge + 操作策略 + kpi-row × 4 |
| 7 | 回报 | grid-4持仓参数 + roi-table × 4行 + EV/风险/风报比汇总 |

**首页返回链接：**
```html
<a class="home-link" href="index.html">← 首页</a>
```

**幻灯片控制器（必须使用 PresentationController 类）：**
- 支持键盘左右/上下箭头
- 支持鼠标滚轮
- 支持触摸左右滑动
- 底部进度条 + 右下角页码

---

## Step 4：更新首页

在 `{config.repo_path}/index.html` 找到最后一个 `.cards` 区块，在其后添加：

```html
<div class="sec-lbl">{股票名称} {代码} · {行业赛道}</div>
<div class="cards">
  <a href="{文件名}.html" class="rcard" style="border-color:rgba(xx,xx,xx,.25)">
    <div>
      <div class="rc-badge" style="...">报告 {编号}</div>
      <div class="rc-title" style="margin-top:.5rem;">{股票名称}深度研究<br>多空对决</div>
    </div>
    <div class="rc-sub">8页交互式幻灯片 · 公司全景、五大涨因、五大跌因、PK评分、估值模型、投资结论</div>
    <div class="rc-meta">
      <div class="rc-meta-item">评级：{评级}</div>
      <div class="rc-meta-item">目标价：{目标价}</div>
      <div class="rc-meta-item">8 页</div>
    </div>
    <div class="rc-arrow">→</div>
  </a>
</div>
```

---

## Step 5：Git 推送

```bash
git -C "{config.repo_path}" add {文件名}.html index.html
git -C "{config.repo_path}" commit -m "feat: add {股票名称} {代码} deep research report"
git -C "{config.repo_path}" push origin main
```

推送成功后输出：

```
✅ 报告已生成并发布

📊 {股票名称} {代码} 研究报告
🔗 报告：{config.pages_url}/{文件名}.html
🏠 首页：{config.pages_url}/

评级：{评级} | 当前价：{价格} | 目标价：{目标} | 建议买点：{买点}
```

---

## 质量检查清单（推送前验证）

- [ ] 8页幻灯片全部存在且内容完整
- [ ] 封面 cover-stats 有5个数据项
- [ ] 多头/空头各5条理由，每条有 badge
- [ ] PK表6行，每行有星级和裁判结果
- [ ] 估值图5条 val-row，当前价竖线位置正确（百分比 = 当前价/max）
- [ ] 投资结论有 rating-badge 和至少4条 kpi-row
- [ ] 回报表4行 + 底部3格汇总（EV/最大风险/风报比）
- [ ] PresentationController 脚本存在，幻灯片总数=8
- [ ] home-link 指向 index.html
- [ ] 所有数据标注来源，底部有免责声明

---

## 关键约束

1. **风格零偏移**：严格复用 `_DESIGN_SYSTEM.md` 里的 CSS 变量体系，禁止引入新颜色或字体
2. **数据时效**：搜索时加上"2026年5月"或"2026"确保数据是最新的
3. **有数据才写数字**：找不到的数据标注「暂无公开数据」，禁止捏造
4. **免责声明**：每页 `.note` 包含"不构成投资建议"
5. **推送确认**：`git push` 返回成功后才报告完成

---

## 重置配置

用户说「重新配置 stock-report」或「stock-report 重置路径」时：
删除 `~/.claude/skills/stock-report/config.md`，重新运行配置向导。
