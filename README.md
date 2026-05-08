# Interview Tool — 面试评价卡片生成器

一套 Cursor Agent Skill，把结构化的面试评价文字一键转换为美观的 HTML 卡片，支持导出高清截图。

---

## 效果预览

- 暖米色纸质风格，衬线字体，干净无噪音
- 结论 Badge（通过 / 不通过 / 建议级别 / 待确认）
- 考察内容 Tag 平铺、候选人表现段落、优点 / 风险双栏对比
- 右上角一键导出 PNG 截图

---

## 安装方式

将 `interview-card/` 目录复制到你本地的 Cursor skills 目录：

```bash
# macOS / Linux
cp -r interview-card ~/.cursor/skills/

# Windows（PowerShell）
Copy-Item -Recurse interview-card $env:USERPROFILE\.cursor\skills\
```

重启 Cursor 后，Skill 自动生效。

---

## 使用方式

在 Cursor 对话框里，把面试评价文字发给 Agent，开头加：

```
帮我生成面试评价卡片：

候选人：xxx
岗位：xxx
面试结论：通过 / 不通过
...（其余内容粘贴即可）
```

Agent 会自动读取 Skill，生成 HTML 文件保存到 `~/Desktop/面试评价/`。

---

## 支持的字段

| 字段 | 是否必填 |
|------|----------|
| 候选人姓名（中/英） | 必填 |
| 岗位 | 必填 |
| 面试轮次 | 选填，默认"一面" |
| 面试结论（通过/不通过） | 必填 |
| 建议级别 | 选填 |
| 高级判断 | 选填，无则省略 Badge |
| 考察内容（列表） | 选填 |
| 候选人表现（段落） | 选填 |
| 主要优点（列表） | 选填 |
| 风险点（列表） | 选填 |
| 综合结论 | 必填 |
| 二面安排提醒 | 选填，无则省略 Section |

---

## 文件结构

```
interview-card/
├── SKILL.md        ← Cursor Agent 读取的指令文件
└── template.html   ← HTML / CSS 设计模板（含所有占位符）
```

---

## 注意事项

- 候选人卡片（生成的 HTML 文件）含个人信息，**请勿提交到公开仓库**
- 导出截图功能依赖 html2canvas CDN，需要联网
- 设计色板和排版不要修改 template.html 的 `:root` 变量，否则整体风格会失调
