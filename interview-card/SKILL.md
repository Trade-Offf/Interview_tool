---
name: interview-card
description: 将面试评价文字转换为统一风格的 HTML 卡片页面，支持一键导出截图。当用户提供面试评价文本、说"帮我生成面试卡片"、"出面试评价卡片"、"按面试模板整理"等场景时使用。生成完毕后自动保存为 HTML 文件并告知路径。
---

# 面试评价卡片生成

## 快速流程

1. 从用户输入中提取结构化字段（见下方字段表）
2. 调用 `template.html` 作为设计基础（见 [template.html](template.html)）
3. 将字段填入对应 HTML 组件
4. 保存到用户指定路径，默认为 `~/Desktop/面试评价/面试评价_{岗位简称}_{候选人英文名}.html`

---

## 字段提取规则

| 字段 | HTML 用途 | 备注 |
|------|-----------|------|
| 候选人姓名 | `<h1>` 标题 + meta-row | 格式：`{中文名} / {英文名}` |
| 岗位 | meta-row + 文件名 | |
| 面试轮次 | meta-row | 默认"一面" |
| 面试结论 | `badge-pass` / `badge-fail` | 通过→绿色；不通过→红色 |
| 建议级别 | `badge-level` | 暖棕色 |
| 高级判断 | `badge-warn` | 橙黄色；无则省略 |
| 考察内容（列表） | `topic-tag` 平铺 | 每条一个 tag |
| 候选人表现（段落） | `conclusion-box` 内 `prose` | 可多段 |
| 主要优点（列表） | 左栏 `panel-plus` | 每条一个 `<li>` |
| 风险点（列表） | 右栏 `panel-risk` | 每条一个 `<li>` |
| 综合结论（段落） | `conclusion-box` | 第一句加 `<strong>` |
| 安排提醒（列表） | `reminder-grid` | 每条一个 `reminder-item` |
| 提醒注意事项 | `reminder-note` | ⚠️ 开头 |
| 面试日期 | `footer-bar` 右侧 | 格式：`YYYY · {轮次}` |

---

## 组件速查

### Badge 三种状态

```html
<!-- 通过 -->
<div class="badge badge-pass"><span class="badge-dot"></span>面试结论：通过</div>

<!-- 不通过（将 badge-pass → badge-fail，颜色改为红系） -->
<div class="badge badge-fail"><span class="badge-dot"></span>面试结论：不通过</div>

<!-- 级别 -->
<div class="badge badge-level"><span class="badge-dot"></span>建议级别：中级</div>

<!-- 待定 / 警示 -->
<div class="badge badge-warn"><span class="badge-dot"></span>高级判断：暂不确认</div>
```

当结论为"不通过"时，在 CSS 中追加：

```css
.badge-fail {
  background: #fdf1f0; color: #b83232; border-color: #f0c4c0;
}
.badge-fail .badge-dot { background: #b83232; }
```

### 考察内容 Tags

```html
<div class="topic-grid">
  <span class="topic-tag">浏览器基础：URL 到页面展示</span>
  <!-- 每个知识点一个 topic-tag -->
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
    <div class="panel-title"><div class="panel-icon"></div>主要优点</div>
    <ul class="item-list"><li>...</li></ul>
  </div>
  <div class="panel panel-risk">
    <div class="panel-title"><div class="panel-icon"></div>风险点</div>
    <ul class="item-list"><li>...</li></ul>
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

---

## Section 编号规则

| 序号 | 内容 | 必须存在 |
|------|------|----------|
| 01 | 本轮考察内容 | 是 |
| 02 | 候选人表现 | 是 |
| 03 | 综合结论 | 是 |
| 04 | 安排提醒 | 有内容时才输出 |

---

## 文件命名与保存位置

- 保存目录：`~/Desktop/面试评价/`
- 文件名：`面试评价_{岗位简称}_{候选人英文名}.html`
- 示例：`~/Desktop/面试评价/面试评价_风控前端_Leon.html`

---

## 注意事项

- 所有文本直接写入 HTML，不做 Markdown 转义
- 若用户没有提供某字段（如"高级判断"），直接省略对应 Badge，不显示空块
- 若用户没有安排提醒，省略 Section 04
- 保留 html2canvas 导出按钮，CDN 地址固定：`https://cdn.jsdelivr.net/npm/html2canvas@1.4.1/dist/html2canvas.min.js`
- 完整 CSS 和 HTML 骨架从 [template.html](template.html) 读取，不要重新设计，不要改动色板
