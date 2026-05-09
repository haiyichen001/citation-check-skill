---
name: citation-check
description: 验证学术引用是否真实 — 查论文存不存在、信息对不对、原文引用位置内容是否匹配。支持 PDF/Word/TXT/Markdown/LaTeX。使用 /citation-check 调用。
---

# Citation Check

## Phase 0 — MCP 依赖检测

首先扫描 MCP 工具。输出一行即可：

```
MCP: arxiv ✅ | scholar ✅ | paper-search ✅
```

缺任意一个则停止，输出安装命令后结束。三项齐全才继续。

---

## Phase 1 — 读取 + 提取 + 提问

### 1.1 读取文件

| 格式 | 方式 |
|------|------|
| PDF | `mcp__pdf-reader__read_pdf` |
| DOCX | `python-docx` via Bash |
| TXT/MD | `Read` |
| LaTeX | `Read` + 解析 `\cite{}` `\bibitem{}` |

### 1.2 提取引用

输出清单：
```
文件: paper.pdf (5页) | 提取到 N 条引用 | 类型: 期刊论文/会议/...

[1] 期刊 | Vaswani et al. (2017) "Attention Is All You Need"
    位置: §I, "code-centric automation..."
[2] ...
```

### 1.3 用 AskUserQuestion 提问

提取完后**立即**调用 `AskUserQuestion` 工具，设置两个问题：

**问题一 — 验证范围（单选）**
- header: "范围"
- options:
  - `all` / "全部验证 (共 N 条)"
  - `selected` / "手动指定编号 (如 1,3,5-8)"
- multiSelect: false

**问题二 — 验证深度（单选）**
- header: "深度"
- options:
  - `quick` / "Quick — 只看 Abstract/TLDR (~10-15k tokens/条)"
  - `normal` / "Normal — Abstract+Intro+Conclusion (~20-50k tokens/条) [推荐]"
  - `deep` / "Deep — 全文逐节对比 (~50-150k tokens/条) [费token]"
- multiSelect: false

等用户选择后再进入 Phase 2。

---

## Phase 2 — 并行验证 (10条/轮)

### 强制规则
- 每轮严格 10 条，多余自动下一轮
- 每条开一个独立 Explore 子 Agent，**全部并行启动**
- Agent 完成后用 TaskUpdate 更新状态
- 本轮全完成后自动进入下一轮，不等待用户

### 单条 Agent 任务

每个 Agent 收到：编号、标题、作者、年份、DOI/ArXiv ID（如有）、引用位置的上下文（±3句）。

三层检查：

**Layer A — 存在性**

| 条件 | 工具 |
|------|------|
| DOI | `mcp__scholar__get_paper` |
| ArXiv ID | `mcp__arxiv__get_abstract` |
| 标题 | `mcp__scholar__search_papers` |
| 仅作者+关键词 | 组合搜索 |
| 兜底 | WebSearch |

标题搜索失败时：去标点、引号括核心短语、只取前 5-8 词。

**Layer B — 元数据**

对照引用类型字段表（见 REFERENCES.md）逐字段比对。作者、标题、期刊、年份、卷/期/页码为必须匹配项。

**Layer C — 内容支撑（深度由 Phase 1 用户选择决定）**

| 深度 | 范围 |
|------|------|
| quick | Abstract + TLDR |
| normal | Abstract + Introduction + Conclusion |
| deep | 全文逐节 |

提取引用处的主张 (Claim)，对比论文实际内容：支撑 / 夸大 / 无关 / 矛盾。

### 进度显示

每轮启动：
```
── Round 1/3 — [1]-[10] — 深度: normal ────────────────────
  ⏳ 10 个 Agent 并行启动中...
```

Agent 完成后即时更新：
```
  ✅ [1] Vaswani 2017 — 通过
  ⚠️ [4] Yao 2022 — 年份有出入
  ⏳ [5] Daareyni 2025 — 查证中...
```

轮次完成：
```
── Round 1/3 完成 — ✅8 ⚠️1 ❌0 🔍0 📝1 — 剩余11条，进入Round 2...
```

---

## Phase 3 — 汇总报告

所有引用查完后，输出汇总报告：

```
# 引用验证报告
文件: xxx.pdf | 引用总数: N
✅ 通过: N | ⚠️ 信息有出入: N | ❌ 伪造: N | 🔍 无法验证: N | 📝 内容不匹配: N

[1] ✅ Author (Year) "Title"
    类型: 期刊 | 位置: Line 42, §2.1
    元数据: 全部匹配
    内容: 主张 "xxx" → 论文支持 ✓
    证据: DOI/arXiv链接

[2] ⚠️ Author (Year) "Title"
    元数据不匹配: 年份 2019 → 实际 2018
    ...

[3] ❌ Author (Year) "Title"
    查证结果: 论文不存在，期刊不存在
    ...
```

## 判定标准

```
存在？ ─否→ ❌ 伪造引用
  └是→ 元数据匹配？
         ├否→ ⚠️ 信息有出入
         └是→ 内容支撑主张？
                ├否→ 📝 与引用地点不匹配
                ├是→ ✅ 通过
                └无法判定→ 🔍 无法验证
```
