# DVA (Dimensional Variance Analysis / 变维拆分)

## 如果你是 AI Agent

写代码之前，先做变维分析。按场景选深度，大多数时候快扫就够。

**不要跳过分析直接写代码。** 也不要每次都做完整分析。

## 两个基元

- **实体（节点）**：持有状态，可以有简单的自述行为
- **关系（边）**：连接实体，系统的复杂行为在关系上运转

## 核心判据

**变化速率是否对齐。** 一段逻辑的变化节奏和实体本身一样吗？

- 对齐 → **内化**（收进实体）
- 不对齐 → **外化**（拆出去，变成独立实体或外部配置）

## 三个深度

| 深度 | 耗时 | 什么时候用 | 调用 |
|------|------|-----------|------|
| 快扫 | 30 秒 | 写任何代码之前 | `change-dim-scan` |
| 标准分析 | 2 分钟 | 新功能 / 新模块 | `change-dim-split`（标准模式） |
| 完整分析 | 10-15 分钟 | 新项目 / 重大重构 | `change-dim-split`（完整模式） |

## 设计模式

大部分设计模式是**关系外化的标准拓扑**：

| 关系的什么在变 | 拓扑形状 | 模式 |
|-------------|---------|------|
| 谁来做 | 星形 | Strategy / State / Visitor |
| 怎么创建 | 三角 | Factory / Builder / Prototype |
| 谁被通知 | 辐射 | Observer / Mediator |
| 什么时候做 | 瞬时→持久 | Command / Memento |
| 怎么连 | 插入中间节点 | Facade / Adapter / Bridge / Proxy / Decorator |
| 一个还是多个 | 树形 | Composite / Iterator |
| 按什么顺序 | 链式 | Template Method / Chain of Responsibility |

## 项目结构

```
CLAUDE.md             # 本文件
AGENTS.md             # 指向本文件
GEMINI.md             # Gemini 入口
docs/
  DVA.md              # 英文文档（AI 翻译）
变维拆分.md            # 中文文档（原文）
skills/
  using-dims/          # 入口 skill：判断用哪个深度
  change-dim-scan/     # 快扫 skill：30 秒分析
  change-dim-split/    # 标准 + 完整分析 skill
```

## 禁忌

- 不要每次都做完整分析。大多数时候快扫就够。
- 不要默认使用 DDD 或任何架构框架。
- 不要为假设的未来做设计。YAGNI。
- 不要把变化速率对齐的东西也外化。
- 不要用大炮打蚊子。能用配置文件解决的，不写 Strategy。
- 修改 skill 内容前，确保对齐两个基元。
- 不要给框架加新维度。两个基元（实体 + 关系）足够。
- 新建任何文件、接口、类之前，必须回答"为什么现有的不行"。答不上来就不新建。
