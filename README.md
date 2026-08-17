# my-pi-harness

适用于[Pi-agent](https://github.com/earendil-works/pi-coding-agent) 的自定义技能（skills）。

## 前置要求

- **pi-docparser** 插件用于解析文件：

```bash
pi install npm:pi-docparser
```

## 技能列表

围绕「学术论文 → 中文专利技术交底书」，包含三个可用技能

| 技能 | 用途 | 输入 → 输出 |
| --- | --- | --- |
| `pdf-to-markdown-with-math` | 将学术论文 PDF 提取为 Markdown，并修复 LaTeX 公式使其正常渲染 | `paper.pdf` → `paper.md` |
| `paper-translation-en-cn` | 将英文论文 Markdown 翻译为中文 | `paper.md` → `paper_zh.md` |
| `paper-to-patent` | 将论文提炼为符合中国专利申请规范的中文「技术交底书」 | `paper_zh.md` → `patent_draft.md` |

### 工作流

三个技能串联使用，可将一篇论文最终转化为专利交底书：

```
paper.pdf
   │  pdf-to-markdown-with-math
   ▼
paper.md
   │  paper-translation-en-cn
   ▼
paper_zh.md
   │  paper-to-patent
   ▼
patent_draft.md
```
