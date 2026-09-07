---
title: 语雀lakebook转Obsidian
created: 2026-09-06
updated: 2026-09-06
source: ZCode会话（D:\obsidian_docs 库整理）
tags:
  - 经验收获
  - Obsidian
  - 语雀
---

# 语雀lakebook转Obsidian

## 问题/背景

需要把语雀导出的 `.lakebook` 知识库包批量转成 Obsidian 笔记，保留目录层级、内链和图谱结构。`.lakebook` 无法直接导入 Obsidian，官方也没有转换工具。

## 原因分析

- `.lakebook` 实际是 **tar 包**（不是 zip），解包后每个知识库一个目录，含 `$meta.json` 和每篇文档的 `<slug>.json`。
- 目录树不在单独文件里，而是藏在 `$meta.json` 的 `meta` 字段（双重 JSON）中的 `book.tocYml`，需要按行解析。
- **元数据 JSON 里每篇文档的 `body` 是空的**，正文必须再去读对应的 `<slug>.json`（`doc.body`，语雀 lake 格式 HTML）。
- Obsidian 关系图谱只认笔记间 `[[链接]]`，**不认文件夹层级**——只按目录建文件、从根 MOC 平铺直链，图谱会是一个扁平星形而不是树。

## 解决方案

1. `tar -xf xxx.lakebook -C 目标目录` 解包。
2. 解析 `$meta.json`：`json.loads(json.load(f)["meta"])`，取 `book.tocYml` 按行正则解析出 `type/level/title/url`，用栈结构按 level 还原层级。
3. 每篇 `type: DOC` 的条目按 `url`（slug）读对应 JSON 的 `doc.body`，用 BeautifulSoup 把 lake HTML 转 Markdown：`ne-p`→段落、`h1-h6`→标题、`pre[data-language]`→代码块、`ne-ul/ne-ol`→列表、`code`→行内代码。
4. 每篇笔记写 frontmatter（title/source/created/updated/tags），按层级写入 `<大类>/子目录/`。
5. **每个中间目录建同名索引页**（文件夹笔记），内含"上级：[[…]]"和下级 `[[链接]]` 列表，让图谱呈树状；根目录建大类 MOC 汇总。
6. 0 字的隐藏草稿（`visible=0` 或正文为空）直接跳过。

## 关键代码/命令

```bash
tar -xf "知识库.lakebook" -C 解包目录
```

```python
# 双重 JSON：$meta.json 的 "meta" 字段是字符串，要再 loads 一次
meta = json.loads(json.load(open("$meta.json", encoding="utf-8"))["meta"])
toc = meta["book"]["tocYml"]          # 目录树在这
docs = {d["slug"]: d for d in meta["docs"]}

# 正文不在 docs 元数据里，要按 slug 单独读文件
body = json.load(open(f"{slug}.json", encoding="utf-8"))["doc"]["body"]
```

## 经验教训

- 遇到陌生格式先用 `file` 命令识别真实类型，别被扩展名骗（lakebook 是 tar 不是 zip）。
- 语雀导出包的元数据和正文分离，目录树以 YAML 文本嵌在 JSON 里，解析前先 `json.loads` 一层看真实结构。
- Obsidian 里"文件夹结构"和"图谱结构"是两回事；要图谱好看，每个层级都得有带链接的索引页（文件夹笔记 + MOC 模式）。
- 临时脚本命名别撞 Python 标准库（`inspect.py` 会导致 `import bs4` 时报 `ImportError`）。
