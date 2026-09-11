---
name: interview-card
description: 将面试评价文字或面试逐字稿转换为统一风格的 HTML 卡片页面，支持一键导出截图与复制文字版评价。当用户提供面试评价文本、面试录音/妙记逐字稿、面试会议链接、候选人简历、岗位 JD，或说"帮我生成面试卡片"、"出面试评价卡片"、"按面试模板整理"、"从逐字稿提炼评价"等场景时使用。支持从妙记链接自动拉取逐字稿（lark-cli）、简历一致性核查、多岗位评分模板路由。生成完毕后自动保存为 HTML 文件并告知路径。
---

# 面试评价卡片生成

## 第一步：输入收集（缺什么问什么）

标准输入是**三样材料 + 一个信息**。开场时一次性列给用户，让用户尽量一次给全；缺哪项再逐个追问，不要凭空编造：

| 输入 | 必需性 | 接受的形态 | 缺失时怎么办 |
| ---- | ------ | ---------- | ------------ |
| ① 面试会议记录 | 必需 | 妙记链接（自动拉逐字稿，见下方管道）/ 逐字稿文本粘贴 | 必须追问，拿不到就无法生成 |
| ② 候选人简历 | 强烈建议 | PDF 文件直接拖进对话（可直接读取）/ 文本粘贴 | 追问一次；用户明确没有则跳过简历相关分析 |
| ③ 岗位 JD | 建议给 | 文本粘贴 / 飞书文档链接（用 lark-doc 能力读取） | 追问一次；用户明确没有则岗位信息改问用户，不做 JD 权重标注 |
| ④ 面试轮次 | 必需 | 一面 / 二面 / 三面 | 必须追问（决定用哪套评分维度） |

追问话术要点：每项都提示"接受的形态"，例如"简历可以直接把 PDF 拖进对话"。候选人姓名、应聘岗位优先从简历和 JD 自动提取，提取不到再问用户。

> 兼容旧用法：用户直接粘贴一段"结构化评价文本"（含"面试结论""建议级别"等字段）时，跳过解析流程直接提取字段生成卡片（见"输入模式判断"）。

---

## 逐字稿获取管道（妙记链接 → 逐字稿）

用户给的是妙记链接时，**先静默尝试自动拉取，失败才打扰用户**：

1. **提取 token**：妙记 URL 格式为 `http(s)://<host>/minutes/<minute-token>`，取路径最后一段。
2. **拉取逐字稿**：
   ```bash
   lark-cli vc +notes --minute-tokens <minute-token>
   ```
   依赖 scope：`vc:note:read`、`minutes:minutes:readonly`、`minutes:minutes.artifacts:read`、`minutes:minutes.transcript:export`。
3. **权限不足时引导授权**（首次使用常见）：
   ```bash
   lark-cli auth login --scope "vc:note:read minutes:minutes:readonly minutes:minutes.artifacts:read minutes:minutes.transcript:export" --no-wait --json
   ```
   把返回的授权链接/二维码给用户扫码确认，然后**必须**用返回的 device_code 完成授权（设备码 600 秒过期，超时需重新生成）：
   ```bash
   lark-cli auth login --device-code <device_code>
   ```
4. **降级路径**：lark-cli 未安装 / 授权失败 / 拉取报错时，不要反复重试，直接提示用户："请打开妙记页面，复制逐字稿文本粘贴给我"。
5. **隐私**：拉取到本地的逐字稿文件用完即删，严禁提交进 git 仓库。

---

## 简历解析规则

拿到简历（PDF 或文本）后做两件事：

1. **基础信息提取**：候选人姓名（中/英文名）、工作年限、教育背景、关键项目经历——用于填充卡片头部 meta 信息，替代人工输入。
2. **简历一致性核查**：评估过程中对照简历宣称与面试实际表现。发现明显不符（如简历写"精通 XX"但面试相关问题答不上、项目经历描述与逐字稿细节矛盾），必须在对应评分维度的简评中具体点出，这是简历输入的核心价值。一致性良好时也可在"候选人表现"中提一句作为加分佐证。

没有简历时：跳过一致性核查，基础信息从逐字稿自我介绍片段提取，"项目经验真实性评估"完全依赖逐字稿。

---

## JD 解析规则

拿到 JD（文本或飞书文档链接）后：

1. **识别岗位**：从 JD 标题/正文判断应聘岗位，用于岗位路由（见下节）。
2. **提炼核心要求**：提炼 3～5 条核心岗位要求，覆盖硬技能、经验要求、软素质，每条一句话。
3. **驱动评分权重**：对照核心要求，在评分维度中标注哪几个维度是"本岗位重点"（简评优先覆盖、写得更具体），权重判断只影响简评详略与综合结论侧重，不改变 0-5 打分标准本身。

JD 只作为评估的背景上下文（岗位识别 + 权重标注 + 风险点对照），**不单独产出"JD 匹配度"板块**——匹配判断融入分项评分简评和综合结论即可，避免冗余信息。没有 JD 时：岗位信息改问用户，不做权重标注。

---

## 岗位路由（用哪套评分维度）

评分维度按岗位拆分在 [rubrics/](rubrics/) 目录，**生成前必须先查注册表 [rubrics/_index.md](rubrics/_index.md)**：

1. 从 JD（优先）或用户描述识别岗位，匹配注册表的"别名关键词"。
2. 命中"已就绪"岗位 → 读取对应 rubric 文件（如技术岗 [rubrics/rd.md](rubrics/rd.md)），按其中的轮次维度打分。
3. 命中"占位"岗位或未收录岗位 → 降级模式：AI 根据逐字稿自主总结 4～6 个考察维度，照常按 0-5 分打分，并在卡片中注明"该岗位标准评分模板待 HR 提供，本次维度由 AI 自主总结"。

评分标准统一为 **0～5 分**：0 完全不达标｜1 较差｜2 一般｜3 合格｜4 良好｜5 优秀。

### 每轮打分后必须产出的字段

- **分项评分**：每个关注点一个分数（0-5）+ 一句简评（依据逐字稿具体内容，不要空泛；JD 标注的重点维度简评写得更具体）
- **建议定级**：格式如 `P5/P6` 或 `初级/中级/高级`，基于本轮分项评分综合判断
- **下一轮面试官重点考察**：一面、二面产出，供下一轮参考；**三面（终轮）不产出**此字段，直接省略对应板块

---

## 输入模式判断

收到用户输入后，先判断输入类型再走对应流程：

| 输入类型           | 识别特征                                       | 走哪个流程                           |
| ------------------ | ---------------------------------------------- | ------------------------------------ |
| **结构化评价文本** | 包含"面试结论""候选人""建议级别"等明确字段     | → 直接提取字段，跳到"快速流程步骤 4" |
| **面试逐字稿**     | 大量口语化对话内容，包含面试官提问和候选人回答 | → 先走"逐字稿解析流程"，再回到步骤 4 |
| **妙记链接**       | URL 含 `/minutes/`                             | → 先走"逐字稿获取管道"拿逐字稿       |

---

## 逐字稿解析流程

当输入为原始逐字稿时，按以下步骤提取字段后再生成卡片：

### 1. 基础信息提取

- **候选人姓名**：优先从简历提取；其次从自我介绍或面试官称呼中提取
- **应聘岗位**：优先从 JD 提取；其次从上下文或用户补充信息中获取
- **面试轮次**：以用户明确提供的为准，默认"一面"
- **面试日期**：从文件元信息或用户说明中获取，否则用当前年份

### 2. 考察内容提取

扫描面试官的所有提问，归纳为简洁的技术点标签。每个知识点一个 `topic-tag`，格式如：

- `浏览器基础：URL 到页面展示`
- `重绘重排 / 缓存机制`
- `Event Loop / 微任务 / 宏任务`

### 3. 候选人表现分析（2～3段）

- **第一段**：整体印象 + 基础知识掌握情况
- **第二段**：项目经验真实性评估（有简历时结合简历一致性核查结论）+ 有亮点的具体内容
- **第三段**（可选）：薄弱环节的具体表现（被跳过的题、答错的点、表达问题等）

### 4. 优点提取（3～6条）

从候选人回答中提炼真实亮点，需有具体依据，不要泛泛而谈。

### 5. 风险点提取（3～5条）

- 答错或答不上的题目
- 表达混乱或结构性差的回答
- 与岗位要求（有 JD 时对照 JD 核心要求）的明显 gap
- 简历一致性核查发现的不符点

### 6. 综合结论判断

根据整体表现给出：**通过 / 不通过 / 待定**，并写 1～2 句推进理由（有 JD 时结合整体匹配结论）。

### 7. 安排提醒

提取逐字稿中面试官/候选人提到的任何异常情况（网络、设备、后续安排等）。

---

## 快速流程

1. **输入收集**：按"第一步：输入收集"确认三样材料 + 轮次是否齐全，缺项追问
2. **拿逐字稿**：妙记链接走"逐字稿获取管道"；文本粘贴直接用
3. **解析简历和 JD**：按"简历解析规则""JD 解析规则"提取信息；按"岗位路由"确定评分维度
4. 走"逐字稿解析流程"提取所有字段，按 rubric 维度逐项打分（JD 重点维度简评写得更具体）
5. 调用 `template.html` 作为设计基础（见 [template.html](template.html)）
6. 将字段填入对应 HTML 组件；同步生成 `PLAIN_TEXT_CONTENT` 纯文字版内容（见下方"纯文字版评价"）
7. 保存到用户指定路径，默认为 `~/Desktop/面试评价/面试评价_{岗位简称}_{轮次}_{候选人英文名}.html`
8. 生成完成后提示用户：卡片上有"复制文字版评价"和"导出截图"两个按钮，用于手动粘贴进审批表单（回填审批流程目前无法自动化）

---

## 字段提取规则

| 字段                 | HTML 用途                    | 备注                                      |
| -------------------- | ----------------------------- | ----------------------------------------- |
| 候选人姓名           | `<h1>` 标题 + meta-row        | 格式：`{中文名} / {英文名}`，优先取自简历 |
| 岗位                 | meta-row + 文件名             | 优先取自 JD                                |
| 面试轮次             | meta-row + 文件名             | 必须明确：一面 / 二面 / 三面              |
| 面试结论             | `badge-pass` / `badge-fail`   | 通过→绿色；不通过→红色                    |
| 分项评分             | Section 01 `score-table`      | 每个关注点一行：分数(0-5)+简评            |
| 考察内容（无维度时） | Section 01 `topic-tag` 平铺   | 仅降级模式且不打分时使用，每条一个 tag    |
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

## Section 编号规则

| 序号 | 内容             | 必须存在       |
| ---- | ---------------- | -------------- |
| 01   | 分项评分 / 考察内容 | 是（有评分维度用分项评分表，降级不打分场景用 topic-tag） |
| 02   | 候选人表现       | 是             |
| 03   | 综合结论         | 是             |
| 04   | 定级与交接建议   | 是             |
| 05   | 安排提醒（编号写死在 `{{REMINDER_SECTION}}` 内容里） | 有内容时才输出 |

---

## 组件速查

### Badge 三种状态

```html
<!-- 通过 -->
<div class="badge badge-pass">
  <span class="badge-dot"></span>面试结论：通过
</div>

<!-- 不通过 -->
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

### 考察内容 Tags（降级模式 / 无评分维度时使用）

```html
<div class="topic-grid">
  <span class="topic-tag">浏览器基础：URL 到页面展示</span>
  <!-- 每个知识点一个 topic-tag -->
</div>
```

### 分项评分表（Section 01 优先用这个）

`data-tier` 决定配色：0-2 → `low`（红）｜3 → `mid`（黄）｜4-5 → `high`（绿）。JD 标注的重点维度在标题后加 `<span class="score-focus-tag">JD 重点</span>`。

```html
<div class="score-table">
  <div class="score-row">
    <div>
      <div class="score-point-title">1. 计算机基础与核心技术知识<span class="score-focus-tag">JD 重点</span></div>
      <div class="score-point-comment">对 JS 事件循环讲解清晰，能结合项目案例说明微任务队列。</div>
    </div>
    <div class="score-value" data-tier="high">4<span class="score-max">/5</span></div>
  </div>
  <!-- 每个关注点一个 score-row -->
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

### 定级与交接建议

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

【分项评分】
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

## 文件命名与保存位置

- 保存目录：`~/Desktop/面试评价/{岗位简称}/`（**按岗位分文件夹**，目录不存在则先创建）
- 文件名：`面试评价_{岗位简称}_{轮次}_{候选人英文名}.html`
- 示例：`~/Desktop/面试评价/前端/面试评价_前端_一面_Leon.html`、`~/Desktop/面试评价/财务/面试评价_财务_二面_Amy.html`

---

## 注意事项

- 所有文本直接写入 HTML，不做 Markdown 转义
- 若用户没有提供某字段（如"高级判断"），直接省略对应 Badge，不显示空块
- 若用户没有安排提醒，省略"安排提醒"section（`{{REMINDER_SECTION}}` 置空）
- 保留 html2canvas 导出按钮，CDN 地址固定：`https://cdn.jsdelivr.net/npm/html2canvas@1.4.1/dist/html2canvas.min.js`
- 完整 CSS 和 HTML 骨架从 [template.html](template.html) 读取，不要重新设计，不要改动色板
- **回填审批流程做不到自动化**：审批 API 不支持写入已有实例的字段，最终必须人工把导出的图片或"复制文字版评价"内容手动贴进审批表单，工具的职责止于生成这两种交付物
- `PLAIN_TEXT_CONTENT` 必须和卡片可视内容保持一致，不能只复制其中一部分，否则用户复制粘贴后信息会缺失
- 降级模式（岗位 rubric 未就绪 / 三面无题库）不要因此省略打分——AI 自主定义 4～6 个考察维度后照常按 0-5 分打分，并在候选人表现段落里说明维度来源
- 简历、逐字稿、生成的卡片均含候选人个人信息，**不要提交进 git 仓库或上传公开渠道**
