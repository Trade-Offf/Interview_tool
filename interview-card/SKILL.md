---
name: interview-card
description: 将面试评价文字或面试逐字稿转换为统一风格的 HTML 卡片页面，支持一键导出截图。当用户提供面试评价文本、粘贴面试录音/妙记逐字稿、说"帮我生成面试卡片"、"出面试评价卡片"、"按面试模板整理"、"从逐字稿提炼评价"等场景时使用。生成完毕后自动保存为 HTML 文件并告知路径。
---

# 面试评价卡片生成

## 生成前：输入准备清单

开始生成前，先确认拿到了以下信息；缺哪项就主动问用户要哪项，不要凭空编造：

| 信息 | 必需性 | 说明 |
| ---- | ------ | ---- |
| 面试会议记录 | 必需 | 妙记链接 / 逐字稿文本，二选一皆可 |
| 面试轮次 | 必需 | 一面 / 二面 / 三面，决定用哪套评分维度 |
| 候选人基本信息 | 建议给 | 姓名、应聘岗位；缺失时尝试从逐字稿自我介绍片段提取 |
| 候选人简历亮点 | 可选 | 有助于"项目经验真实性评估"更准；没有则完全依赖逐字稿 |

> 当前仅支持**技术岗（R&D）**的结构化评分维度。其他岗位（运营/风控/BD/实习生）题库尚未配置，遇到时按"结构化评价文本"模式的自由格式处理，不要套用下方的技术岗评分维度。

## 岗位 × 轮次评分维度（技术岗 R&D）

评分标准统一为 **0～5 分**：0 完全不达标｜1 较差｜2 一般｜3 合格｜4 良好｜5 优秀。

### 一面（技术面）— 6 个关注点

1. 计算机基础与核心技术知识
2. 编码与算法能力
3. 技术广度与视野
4. 项目技术实现深度
5. 学习能力与技术成长
6. 沟通与技术表达

### 二面（技术负责人面）— 6 个关注点

1. 技术深度与架构能力
2. 复杂项目主导与交付
3. 技术决策与权衡能力
4. 问题排查与攻坚能力
5. 团队技术影响力
6. 技术规划与业务理解

### 三面 — 题库未定，AI 自主判断

HR 还没有给三面题库。遇到三面时，AI 根据逐字稿内容自主总结 4～6 个考察维度（例如：综合素质、团队协作匹配度、职业规划稳定性、抗压与应变等），按同样的 0-5 分标准打分，不要因为没有预置题库就退化成不打分。

### 每轮打分后必须产出的字段

- **分项评分**：每个关注点一个分数（0-5）+ 一句简评（依据逐字稿具体内容，不要空泛）
- **建议定级**：格式如 `P5/P6` 或 `初级/中级/高级`，基于本轮分项评分综合判断
- **下一轮面试官重点考察**：一面、二面产出，供下一轮参考；**三面（终轮）不产出**此字段，直接省略对应板块

## 输入模式判断

收到用户输入后，先判断输入类型再走对应流程：

| 输入类型           | 识别特征                                       | 走哪个流程                           |
| ------------------ | ---------------------------------------------- | ------------------------------------ |
| **结构化评价文本** | 包含"面试结论""候选人""建议级别"等明确字段     | → 直接提取字段，跳到"快速流程步骤 3" |
| **面试逐字稿**     | 大量口语化对话内容，包含面试官提问和候选人回答 | → 先走"逐字稿解析流程"，再回到步骤 3 |

---

## 逐字稿解析流程

当输入为原始逐字稿时，按以下步骤提取字段后再生成卡片：

### 1. 基础信息提取

- **候选人姓名**：从自我介绍或面试官称呼中提取
- **应聘岗位**：从上下文或用户补充信息中获取
- **面试轮次**：默认"一面"，除非上下文明确说明
- **面试日期**：从文件元信息或用户说明中获取，否则用当前年份

### 2. 考察内容提取

扫描面试官的所有提问，归纳为简洁的技术点标签。每个知识点一个 `topic-tag`，格式如：

- `浏览器基础：URL 到页面展示`
- `重绘重排 / 缓存机制`
- `Event Loop / 微任务 / 宏任务`

### 3. 候选人表现分析（2～3段）

- **第一段**：整体印象 + 基础知识掌握情况
- **第二段**：项目经验真实性评估 + 有亮点的具体内容
- **第三段**（可选）：薄弱环节的具体表现（被跳过的题、答错的点、表达问题等）

### 4. 优点提取（3～6条）

从候选人回答中提炼真实亮点，需有具体依据，不要泛泛而谈。

### 5. 风险点提取（3～5条）

- 答错或答不上的题目
- 表达混乱或结构性差的回答
- 与岗位要求的明显 gap

### 6. 综合结论判断

根据整体表现给出：**通过 / 不通过 / 待定**，并写 1～2 句推进理由。

### 7. 安排提醒

提取逐字稿中面试官/候选人提到的任何异常情况（网络、设备、后续安排等）。

---

## 快速流程

1. 判断输入类型（见上方"输入模式判断"），并确认"生成前：输入准备清单"里的必需项已拿到
2. 若为逐字稿，先走"逐字稿解析流程"提取所有字段；技术岗（R&D）额外按"岗位 × 轮次评分维度"逐项打分
3. 调用 `template.html` 作为设计基础（见 [template.html](template.html)）
4. 将字段填入对应 HTML 组件；同步生成 `PLAIN_TEXT_CONTENT` 纯文字版内容（见下方"纯文字版评价"）
5. 保存到用户指定路径，默认为 `~/Desktop/面试评价/面试评价_{岗位简称}_{轮次}_{候选人英文名}.html`
6. 生成完成后提示用户：卡片上有"复制文字版评价"和"导出截图"两个按钮，用于手动粘贴进审批表单（回填审批流程目前无法自动化）

---

## 字段提取规则

| 字段                 | HTML 用途                    | 备注                                      |
| -------------------- | ----------------------------- | ----------------------------------------- |
| 候选人姓名           | `<h1>` 标题 + meta-row        | 格式：`{中文名} / {英文名}`               |
| 岗位                 | meta-row + 文件名             |                                            |
| 面试轮次             | meta-row + 文件名             | 必须明确：一面 / 二面 / 三面              |
| 面试结论             | `badge-pass` / `badge-fail`   | 通过→绿色；不通过→红色                    |
| 分项评分（技术岗）   | Section 01 `score-table`      | 每个关注点一行：分数(0-5)+简评            |
| 考察内容（非技术岗） | Section 01 `topic-tag` 平铺   | 仅无评分维度时使用，每条一个 tag          |
| 候选人表现（段落）   | `conclusion-box` 内 `prose`   | 可多段                                    |
| 主要优点（列表）     | 左栏 `panel-plus`             | 每条一个 `<li>`                           |
| 风险点（列表）       | 右栏 `panel-risk`             | 每条一个 `<li>`                           |
| 综合结论（段落）     | `conclusion-box`              | 第一句加 `<strong>`                       |
| 建议定级             | `badge-level` + `handoff-panel` | 每轮独立给出，格式如 `P6` / `中级`      |
| 下一轮面试官重点考察 | `handoff-panel`                | 一面/二面产出；三面（终轮）省略此板块     |
| 安排提醒（列表）     | `reminder-grid`               | 每条一个 `reminder-item`                  |
| 提醒注意事项         | `reminder-note`               | ⚠️ 开头                                    |
| 面试日期             | `footer-bar` 右侧             | 格式：`YYYY · {轮次}`                     |
| 纯文字版评价         | `PLAIN_TEXT_CONTENT`（JS 变量）| 见下方"纯文字版评价"生成规则              |

---

## 组件速查

### Badge 三种状态

```html
<!-- 通过 -->
<div class="badge badge-pass">
  <span class="badge-dot"></span>面试结论：通过
</div>

<!-- 不通过（将 badge-pass → badge-fail，颜色改为红系） -->
<div class="badge badge-fail">
  <span class="badge-dot"></span>面试结论：不通过
</div>

<!-- 级别 -->
<div class="badge badge-level">
  <span class="badge-dot"></span>建议级别：中级
</div>

<!-- 待定 / 警示 -->
<div class="badge badge-warn">
  <span class="badge-dot"></span>高级判断：暂不确认
</div>
```

当结论为"不通过"时，在 CSS 中追加：

```css
.badge-fail {
  background: #fdf1f0;
  color: #b83232;
  border-color: #f0c4c0;
}
.badge-fail .badge-dot {
  background: #b83232;
}
```

### 考察内容 Tags（非技术岗 / 无评分维度时使用）

```html
<div class="topic-grid">
  <span class="topic-tag">浏览器基础：URL 到页面展示</span>
  <!-- 每个知识点一个 topic-tag -->
</div>
```

### 分项评分表（技术岗，Section 01 优先用这个）

`data-tier` 决定配色：0-2 → `low`（红）｜3 → `mid`（黄）｜4-5 → `high`（绿）。

```html
<div class="score-table">
  <div class="score-row">
    <div>
      <div class="score-point-title">1. 计算机基础与核心技术知识</div>
      <div class="score-point-comment">对 JS 事件循环讲解清晰，能结合项目案例说明微任务队列。</div>
    </div>
    <div class="score-value" data-tier="high">4<span class="score-max">/5</span></div>
  </div>
  <!-- 每个关注点一个 score-row，一面/二面各 6 个 -->
  <div class="score-total-row">
    <div class="score-total-label">平均分</div>
    <div class="score-total-value">3.8</div>
  </div>
</div>
```

### 候选人表现（段落）

```html
<div class="conclusion-box" style="background:#fafaf8;">
  <p class="prose">第一段...</p>
  <p class="prose" style="margin-top:10px;">第二段...</p>
</div>
```

### 优点 / 风险点双栏

```html
<div class="two-col">
  <div class="panel panel-plus">
    <div class="panel-title">
      <div class="panel-icon"></div>
      主要优点
    </div>
    <ul class="item-list">
      <li>...</li>
    </ul>
  </div>
  <div class="panel panel-risk">
    <div class="panel-title">
      <div class="panel-icon"></div>
      风险点
    </div>
    <ul class="item-list">
      <li>...</li>
    </ul>
  </div>
</div>
```

### 安排提醒块

```html
<div class="reminder-box">
  <div class="reminder-label">注意事项</div>
  <div class="reminder-grid">
    <div class="reminder-item">确认声音是否清晰</div>
    <!-- 每条一个 reminder-item -->
  </div>
  <div class="reminder-note">⚠️ 补充说明文字</div>
</div>
```

### 定级与交接建议（Section 04）

非终轮（一面/二面）用双栏，`handoff-grid` 不加 modifier；终轮（三面）省略"下一轮考察"面板，`handoff-grid` 加 `single` 类。

```html
<!-- 非终轮：两个面板 -->
<div class="handoff-grid">
  <div class="handoff-panel">
    <div class="panel-title"><div class="panel-icon" style="background:var(--accent);"></div>建议定级</div>
    <p class="prose">P6（中级偏高）。技术基础扎实，架构设计能力待二面进一步验证。</p>
  </div>
  <div class="handoff-panel">
    <div class="panel-title"><div class="panel-icon" style="background:var(--accent);"></div>下一轮面试官重点考察</div>
    <p class="prose">重点验证高并发场景下的系统设计能力，可追问项目中的具体架构权衡。</p>
  </div>
</div>

<!-- 终轮（三面）：只有定级面板，handoff-grid 加 single -->
<div class="handoff-grid single">
  <div class="handoff-panel">
    <div class="panel-title"><div class="panel-icon" style="background:var(--accent);"></div>建议定级</div>
    <p class="prose">P6，建议发 Offer。</p>
  </div>
</div>
```

---

## 纯文字版评价（`PLAIN_TEXT_CONTENT`）

因为审批表单只能填**图片或文字**、不能放链接，卡片右上角提供"复制文字版评价"按钮，需要在生成时把纯文字内容写入 `template.html` 里的 `{{PLAIN_TEXT_JSON}}` 占位符（`JSON.stringify` 后的字符串，注意转义换行）。

文字版内容需要覆盖卡片全部关键信息，按以下顺序组织（无内容的字段直接跳过，不留空行占位）：

```text
{候选人中文名}/{英文名} · {岗位} · {轮次}
面试结论：{通过/不通过/待定}

【分项评分】（技术岗）
1. {关注点1}：{分数}/5 — {简评}
2. {关注点2}：{分数}/5 — {简评}
...
平均分：{avg}

【候选人表现】
{表现段落1}
{表现段落2}

【主要优点】
- {优点1}
- {优点2}

【风险点】
- {风险1}
- {风险2}

【综合结论】
{综合结论文字}

【建议定级】{P6/中级 等}
【下一轮面试官重点考察】{建议内容}（终轮不含此行）
```

---

## Section 编号规则

| 序号 | 内容             | 必须存在       |
| ---- | ---------------- | -------------- |
| 01   | 分项评分 / 考察内容 | 是（技术岗用分项评分表，其他情况用 topic-tag） |
| 02   | 候选人表现       | 是             |
| 03   | 综合结论         | 是             |
| 04   | 定级与交接建议   | 是             |
| 05   | 安排提醒         | 有内容时才输出 |

---

## 文件命名与保存位置

- 保存目录：`~/Desktop/面试评价/`
- 文件名：`面试评价_{岗位简称}_{轮次}_{候选人英文名}.html`
- 示例：`~/Desktop/面试评价/面试评价_技术岗_一面_Leon.html`

---

## 注意事项

- 所有文本直接写入 HTML，不做 Markdown 转义
- 若用户没有提供某字段（如"高级判断"），直接省略对应 Badge，不显示空块
- 若用户没有安排提醒，省略 Section 05
- 保留 html2canvas 导出按钮，CDN 地址固定：`https://cdn.jsdelivr.net/npm/html2canvas@1.4.1/dist/html2canvas.min.js`
- 完整 CSS 和 HTML 骨架从 [template.html](template.html) 读取，不要重新设计，不要改动色板
- **回填审批流程做不到自动化**：审批 API 不支持写入已有实例的字段，最终必须人工把导出的图片或"复制文字版评价"内容手动贴进审批表单，工具的职责止于生成这两种交付物
- `PLAIN_TEXT_CONTENT` 必须和卡片可视内容保持一致，不能只复制其中一部分，否则用户复制粘贴后信息会缺失
- 三面没有预置题库时，不要因此省略打分——AI 自主定义 4~6 个考察维度后照常按 0-5 分打分，并在候选人表现段落里说明"三面维度由 AI 根据本轮对话内容自主总结"
