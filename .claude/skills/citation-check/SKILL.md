---
name: citation-check
description: 验证文本中的学术引用是否真实 — 查论文存不存在、信息对不对、原文引用位置内容是否匹配。支持 PDF/Word/TXT/Markdown/LaTeX。使用 /citation-check 调用。
---

# Citation Check — 学术引用真实性验证

## 支持的输入格式

| 格式 | 读取方式 | 引用提取方式 |
|------|---------|-------------|
| PDF | `mcp__pdf-reader__read_pdf` 提取全文 | 正则 + 参考文献区域识别 |
| DOCX | `python-docx` 通过 Bash 读取 | 正则 + 尾注/脚注提取 |
| TXT/MD | `Read` 工具 | 正则匹配 |
| LaTeX (.tex) | `Read` 工具 | 解析 `\cite{}`、`\bibitem{}`、thebibliography 环境、biblatex `\printbibliography` |

## 引用类型字段检查表

查证时按类型逐字段比对。**加粗** 为必须匹配项，其余为辅助检查项。

### 期刊论文 Journal Article

| 字段 | 英文标识 | 中文标识 |
|------|---------|---------|
| **作者** | Author(s) | 作者 |
| **标题** | Title | 题名 |
| **期刊名** | Journal | 期刊名 |
| **年份** | Year | 年份 |
| **卷号** | Volume | 卷 |
| **期号** | Issue/Number | 期 |
| **页码** | Pages | 起止页码 |
| DOI | DOI | DOI |
| ISSN | ISSN | ISSN |

### 会议论文 Conference Paper

| 字段 | 英文标识 | 中文标识 |
|------|---------|---------|
| **作者** | Author(s) | 作者 |
| **标题** | Title | 题名 |
| **会议名称** | Conference/Proceedings | 会议/论文集名 |
| **年份** | Year | 年份 |
| **页码** | Pages | 页码 |
| 地点 | Location | 会议地点 |
| 日期 | Date | 会议日期 |
| DOI | DOI | DOI |

### 书籍 Book

| 字段 | 英文标识 | 中文标识 |
|------|---------|---------|
| **作者/编者** | Author(s)/Editor(s) | 作者/编者 |
| **书名** | Book Title | 书名 |
| **出版商** | Publisher | 出版社 |
| **出版年份** | Year | 出版年 |
| 版次 | Edition | 版次 |
| 出版地 | Location | 出版地 |
| ISBN | ISBN | ISBN |

### 书籍章节 Book Chapter

| 字段 | 英文标识 | 中文标识 |
|------|---------|---------|
| **章节作者** | Chapter Author(s) | 章节作者 |
| **章节标题** | Chapter Title | 章节题名 |
| **书名** | Book Title | 书名 |
| **编者** | Editor(s) | 编者 |
| **出版商** | Publisher | 出版社 |
| **年份** | Year | 出版年 |
| **页码** | Pages | 起止页码 |

### 学位论文 Thesis/Dissertation

| 字段 | 英文标识 | 中文标识 |
|------|---------|---------|
| **作者** | Author | 作者 |
| **标题** | Title | 题名 |
| **学位类型** | Degree (MA/MS/PhD) | 硕士/博士 |
| **授予机构** | Institution/University | 授予单位 |
| **地点** | Location | 所在地 |
| **年份** | Year | 授予年份 |
| 指导教师 | Advisor | 导师 |

### 预印本 Preprint (ArXiv, TechRxiv 等)

| 字段 | 英文标识 |
|------|---------|
| **作者** | Author(s) |
| **标题** | Title |
| **ArXiv ID 或等同标识** | ArXiv ID / Preprint ID |
| **年份** | Year |
| 版本号 | Version (v1, v2...) |

### 专利 Patent

| 字段 | 英文标识 | 中文标识 |
|------|---------|---------|
| **发明人/专利权人** | Inventor(s)/Assignee | 发明人/专利权人 |
| **专利标题** | Patent Title | 专利名称 |
| **专利号** | Patent Number | 专利号 |
| **授权日期** | Date | 公告日期 |
| 专利局 | Patent Office | 专利局 |

### 技术报告 Technical Report

| 字段 | 英文标识 | 中文标识 |
|------|---------|---------|
| **作者/机构** | Author(s)/Institution | 作者/机构 |
| **标题** | Title | 题名 |
| **报告号** | Report Number | 报告号 |
| **发布机构** | Institution | 发布机构 |
| **年份** | Year | 年份 |

### 报纸文章 Newspaper Article

| 字段 | 英文标识 | 中文标识 |
|------|---------|---------|
| **作者** | Author | 作者 |
| **标题** | Title | 题名 |
| **报纸名** | Newspaper | 报纸名 |
| **日期** | Date | 出版日期 |
| 版面 | Page/Section | 版次 |

### 网页/在线资源 Web Resource

| 字段 | 英文标识 | 中文标识 |
|------|---------|---------|
| **作者/组织** | Author/Organization | 作者/组织 |
| **标题** | Title | 题名 |
| **URL** | URL | 网址 |
| 发布日期 | Date | 发布日期 |
| 访问日期 | Access Date | 引用日期 |

### 标准 Standard

| 字段 | 英文标识 | 中文标识 |
|------|---------|---------|
| **标准组织** | Standards Body | 标准组织 |
| **标准号** | Standard Number | 标准号 |
| **标题** | Title | 标准名称 |
| **年份** | Year | 发布年份 |

---

## 判定标准

四种判定，按检查顺序分叉：

```
论文是否存在？
├─ 否 → ❌ 伪造引用 (Fabricated)
│      论文在任何数据库中均无法找到
└─ 是 → 元数据是否匹配？
         ├─ 否 → ⚠️ 信息有出入 (Metadata Mismatch)
         │      论文存在，但作者/年份/期刊/卷期/页码至少一项不对
         └─ 是 → 引用处的主张，论文内容是否支撑？
                  ├─ 否 → 📝 与引用地点不匹配 (Content Mismatch)
                  │      论文真实存在、信息正确，但论文实际内容
                  │      不支持引用处声称的观点
                  ├─ 是 → ✅ 通过 (Verified)
                  └─ 无法判定 → 🔍 无法验证
```

| 判定 | 含义 | 典型场景 |
|------|------|---------|
| ❌ 伪造引用 | 论文根本不存在 | LLM 编造了标题和作者 |
| ⚠️ 信息有出入 | 论文存在，元数据有误 | 作者写错、年份偏差、期刊名拼错 |
| 📝 与引用地点不匹配 | 论文存在、信息对，但内容不支撑 | 张冠李戴、夸大结论、曲解原文 |
| 🔍 无法验证 | 信息不足无法判断 | 中文文献无数据库覆盖、预印本无全文、仅模糊提及 |

---

## 工作流程

### Phase 1 — 提取与展示

1. 根据文件格式选择读取方式，提取全文
2. 分离两样东西：
   - **参考文献列表**：文末/文内的引用条目
   - **引用位置映射**：正文每个引用标记的位置(行号/段落号) + 周围上下文 ±3句
3. 识别每条引用的**类型**（对照上面的字段表）
4. 输出提取清单给用户确认：
   ```
   文件: thesis.tex (LaTeX, 156KB)
   提取到 12 条引用

   [1] 期刊论文 | Vaswani et al. (2017) "Attention Is All You Need"
       引用位置: Line 42, Line 108, Line 203
   [2] 会议论文 | Chen et al. (2020) ...
   ...

   开始逐条验证? (每轮 10 条)
   ```

### Phase 2 — 并行验证 (10 条/轮)

每条引用启用一个独立 `Explore` 子 Agent 做三层检查，全部并行：

#### Layer A：存在性验证

查证工具按优先级降级：

| 条件 | 工具 | 精确度 |
|------|------|--------|
| 有 DOI | `mcp__scholar__get_paper` (DOI 精确查询) | 最高 |
| 有 ArXiv ID | `mcp__arxiv__get_abstract` | 最高 |
| 英文文献有标题 | `mcp__scholar__search_papers` (标题精确匹配) | 高 |
| 英文文献仅作者+关键词 | `mcp__scholar__search_papers` (组合搜索) | 中 |
| 中文文献 | `mcp__paper-search__search_google_scholar` → WebSearch 知网/万方 | 中 |
| 其它 | WebSearch / WebFetch | 低 |

#### Layer B：元数据匹配

对照上面的**字段检查表**，根据引用类型逐字段比对。加粗字段必须匹配。

#### Layer C：内容支撑验证

1. 用 `mcp__scholar__read_paper` 或 `mcp__arxiv__download_paper` 获取论文内容（优先 Abstract + Introduction + Conclusion）
2. 从引用位置的上下文提取 **主张 (Claim)** — 原文引用该论文时想证明什么
3. 对比：论文实际内容是否支撑该主张
   - **支撑**：论文明确写了相同的结论
   - **夸大**：论文有相关讨论但不完全支持
   - **无关**：论文内容与主张无关
   - **矛盾**：论文结论与主张相反

每完成一条，用 `TaskUpdate` 更新进度。用户实时看到状态。

检索技巧：
- 标题搜索失败时，尝试去掉标点、用引号括起核心短语、只用前 5-8 个词
- 中文名注意拼音/汉字双向搜索
- 常见的 "et al." 缩略，尝试用第一作者全名 + 年份搜索

### Phase 3 — 汇总报告

```
# 引用验证报告

**文件**: thesis.tex (LaTeX, 156KB)
**引用总数**: 12
✅ 通过: 8 | ⚠️ 信息有出入: 1 | 📝 与引用地点不匹配: 1 | ❌ 伪造: 1 | 🔍 无法验证: 1

---

### [1] ✅ Vaswani et al. (2017) "Attention Is All You Need"
- **类型**: 会议论文 (NeurIPS 2017)
- **引用位置**: Line 42 (§2.1) / Line 108 (§3.2) / Line 203 (§5.1)
- **元数据**: 全部匹配
- **内容支撑**: Line 42 处主张"Transformer 取代 RNN 成为主流"→ 论文 Introduction 明确提及 → 支撑 ✓
  - Line 108 处主张"自注意力机制计算复杂度 O(n²)"→ 论文 Section 3.2 讨论 → 支撑 ✓
  - Line 203 处主张"Transformer 在 CV 领域同样有效"→ 论文未涉及 CV → 📝 此位置不匹配
- **证据**: https://arxiv.org/abs/1706.03762

### [2] ⚠️ Devlin et al. (2019) "BERT: Pre-training of..."
- **类型**: 期刊论文
- **引用位置**: Line 56 (§2.3)
- **元数据不匹配**: 实际年份为 2018 (非 2019)，实际发表于 NAACL 2019 (非期刊)
- **内容支撑**: Line 56 处主张正确描述了 BERT 预训练方法
- **证据**: https://arxiv.org/abs/1810.04805

### [3] 📝 Zhang & Williams (2023) "A Novel Framework..."
- **类型**: 期刊论文 (Nature Machine Intelligence, 2023)
- **引用位置**: Line 78 (§3.1)
- **元数据**: 匹配
- **内容不匹配**: Line 78 处主张"该方法在因果推理准确率上超越人类专家"
  → 论文实际仅声称"在特定数据集上提升了 5%"，未与人类比较 → 夸大结论
- **证据**: https://doi.org/10.xxxx/xxxxx

### [4] ❌ Thompson et al. (2022) "Atmospheric Carbon..."
- **类型**: 期刊论文 (Journal of Climate Engineering)
- **引用位置**: Line 156 (§4.2)
- **查证结果**: 期刊不存在 (Journal of Climate Engineering 非真实期刊)
  搜索 Semantic Scholar / Google Scholar / Web 均无此论文
- **判定**: 伪造引用

### [5] 🔍 王晓明 (2020) "基于深度学习的图像分割..."
- **类型**: 中文学位论文
- **引用位置**: Line 200 (§5.1)
- **查证结果**: 知网/万方不在 MCP 工具覆盖范围，WebSearch 未找到精确匹配
- **判定**: 无法验证
```

### 后续操作

报告输出后询问用户是否需要：
1. 导出为 Markdown 报告文件
2. 只列出有问题的引用（⚠️ + 📝 + ❌）
3. 对某条引用深入查看被引论文原文
