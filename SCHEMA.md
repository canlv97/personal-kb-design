# Wiki Schema

> 多域知识库。原始素材不分类（frontmatter 标记），知识按成熟度流转。

## 架构

```
Economics/                    ← Obsidian 库
│
├── raw/                      ← 第1层：原始素材（不可变，平铺存放）
│   └── *.md                  ← 分类靠 frontmatter，不建子文件夹
│
├── wiki/                     ← 第2层：加工后的知识
│   ├── literature/           ← 文献笔记（单篇提取，自己话写）
│   ├── concepts/             ← 永久笔记（综合多篇的概念）
│   ├── entities/             ← 永久笔记（人、公司、模型、标的）
│   ├── comparisons/          ← 永久笔记（A vs B）
│   ├── index.md              ← 全局目录
│   └── log.md                ← 操作日志
│
├── projects/                 ← 项目笔记（临时，项目结束归档或删除）
│   └── {项目名}/
│
├── SCHEMA.md                 ← 本文件
│
└── 原有的个人笔记              ← 已完成内容，不混入体系
```

## 知识流转

```
灵感闪现
    ↓
wiki/literature/xx.md   status: fleeting    ← 快速记，不追求完整
    ↓ 补充来源、上下文
wiki/literature/xx.md   status: draft       ← 有了引用链
    ↓ 综合 2+ 来源，形成独立观点，升格
wiki/concepts/xx.md     status: done        ← 永久笔记
    ↓ 为特定任务服务
projects/某项目/                          ← 项目笔记，结束即死
```

规则：

| 升格条件 | 操作 |
|----------|------|
| 初读 raw 素材 → 用自己的话写要点 | 创建 `wiki/literature/`，status=fleeting |
| 加了引用、补充了上下文 | status → draft |
| 综合 2+ 来源，形成独立观点 | 移到 `wiki/concepts/` 或 `entities/` 或 `comparisons/`，status=done |
| fleeting 超过 30 天未动 | lint 时提醒：要么发展，要么删 |

## 文件约定

- 文件名：中文不用空格，英文用连字符（`龙头战法.md`、`cortex-m3-nvic.md`）
- 每个 wiki 页面至少链 2 个其他页面
- raw/ 里的文件永不修改，补充和纠正在 wiki/ 层做
- 新建或删除 wiki 页面时，必须更新 `wiki/index.md`
- 每次操作必须追加 `wiki/log.md`
- 同一文件在流转过程中不搬家——通过修改 status 和移动目录完成升格

## Frontmatter

### raw/ 文件

```yaml
---
type: article | paper | transcript
domain: [交易, 嵌入式]            ← 可多个
source_url: https://...
ingested: YYYY-MM-DD
---
```

### wiki/ 文件

```yaml
---
title: 页面标题
created: YYYY-MM-DD
updated: YYYY-MM-DD
type: concept | entity | comparison | literature
domain: [交易, 嵌入式]
status: fleeting | draft | done     ← literature 层必填，永久笔记默认 done
tags: [关键词, ...]
sources: [raw/xxx.md]
confidence: high | medium | low
---
```

- `domain` 数组：标所属域，可多个
- `status`：literature 层记录成熟度，永久笔记默认 done 可不写
- `confidence`：单来源或主观内容标 medium 或 low
- `tags`：自由关键词，描述内容主题（不是域，域已在 domain 字段）

## 域标签

`domain` 字段可用值（可多选）：

| 值 | 含义 |
|----|------|
| `交易` | 金融交易、A股、量化 |
| `AI` | 大模型、Agent、工作流 |
| `嵌入式` | STM32、PCB、单片机 |
| `光电` | 半导体物理、光谱、太阳能电池 |
| `自媒体` | 内容创作、ASMR、平台运营 |

新增域先在 SCHEMA.md 这里登记，再用。

## 页面创建阈值

- **创建新页面**：实体/概念出现在 2+ 素材中，或在一篇素材中处于核心位置
- **追加到已有页面**：素材内容被已有页面覆盖
- **不创建**：一笔带过的提及
- **拆分**：页面超过 ~200 行时分拆
- **归档**：内容被完全替代时移到 `_archive/`，从 index 移除

## 更新策略

遇到矛盾内容时：
1. 检查日期，新素材一般覆盖老素材
2. 确实矛盾：两方都保留，标注日期和来源
3. frontmatter 加 `contested: true` 和 `contradictions: [页面名]`
4. 标记等待人工审核

## lint 检查项

- 断链（wikilink 指向不存在的页面）
- 孤儿页面（无入链）
- fleeting 超过 30 天未动
- 缺 frontmatter
- 页面超过 200 行未拆分
- 矛盾标记未处理
- log.md 超 500 条未轮转

## 个人笔记规则

vault 根目录下的 `.md` 文件（非 `raw/`、`wiki/`、`projects/` 下的）是郭京个人笔记——Hermes 读但不主动改，只在用户明确要求时操作。
