# 变维拆分 / Change-Dimensional Splitting

> 同变的归一，异变的拆开。

写代码之前想清楚一件事：**这段逻辑该住在实体里面，还是该拆出去？**

判据只有一个——变化速率是否对齐。

## 核心思想

任何软件系统只有两种东西：

- **实体（节点）**：持有状态
- **关系（边）**：连接实体，系统的复杂行为在关系上运转

设计的核心决策：**行为的住所。**

- 变化节奏和实体同步 → **内化**（收进实体）
- 变化节奏和实体不同步 → **外化**（拆出去）

## 三个深度

| 深度 | 耗时 | 什么时候用 |
|------|------|-----------|
| 快扫 | 30 秒 | 写任何代码之前 |
| 标准分析 | 2 分钟 | 新功能 / 新模块 |
| 完整分析 | 10-15 分钟 | 新项目 / 重大重构 |

大多数时候快扫就够。不要每次都做完整分析。

## 快速开始

### 给人看的

读 [变维拆分.md](变维拆分.md) — 完整框架描述，包含示例、验证步骤、设计模式推导。

### 给 AI Agent 用的

`dims/` 目录下有 Claude Code / Gemini 可直接使用的 skill：

```
dims/
  skills/
    using-dims/SKILL.md        # 入口：判断用哪个深度
    change-dim-scan/SKILL.md   # 快扫（30秒）
    change-dim-split/SKILL.md  # 标准分析 + 完整分析
```

在 Claude Code 项目中配置后，Agent 写代码前会自动选择分析深度。

### 给任何 AI 的 prompt

> 写代码前做变维分析。按场景选深度：
>
> **日常编码（快扫）：** 想一下"这里什么会变"，在容易变的地方留口子。不要过度设计。
>
> **新功能（标准分析）：** 找到实体和关系，标注哪条关系的变化节奏和实体不对齐，在不齐处做隔离。
>
> **新项目（完整分析）：** 列出所有实体和关系，标注变化速率和趋势，按一致性分组，每个拆分点从最轻量的手段开始选（配置 > 回调 > 接口 > 设计模式）。

## 设计模式 = 关系外化的标准拓扑

| 关系的什么在变 | 拓扑形状 | 对应的模式 |
|-------------|---------|-----------|
| 谁来做 | 星形 | Strategy / State / Visitor |
| 怎么创建 | 三角 | Factory / Builder / Prototype |
| 谁被通知 | 辐射 | Observer / Mediator |
| 什么时候做 | 瞬时→持久 | Command / Memento |
| 怎么连 | 插入中间节点 | Facade / Adapter / Bridge / Proxy / Decorator |
| 一个还是多个 | 树形 | Composite / Iterator |
| 按什么顺序 | 链式 | Template Method / Chain of Responsibility |

变维拆分是所有这些模式的**上游分析步骤**——先分析变化速率，再从工具箱里选工具。

## 实验验证

两个开源项目的对照实验（4 环境 × 5 轮需求 × 1 模型）：

| 项目 | God Class 类型 | 结论 |
|------|-------------|------|
| Shopizer（电商） | if-else 链 | dims 找到了全面的改动范围，v1 有过度设计，v2 修正 |
| TEAMMATES（教育） | 纯委托 Façade | dims 的 22 个额外文件全部必要，base 漏改前端产出不可部署 |

详细报告见 [reports/](reports/)。

## 项目结构

```
├── README.md              # 本文件
├── 变维拆分.md             # 完整框架文档
├── dims/                  # AI Agent skill 实现
│   ├── CLAUDE.md
│   ├── skills/
│   │   ├── using-dims/
│   │   ├── change-dim-scan/
│   │   └── change-dim-split/
│   └── analyze-token-usage.py
├── reports/               # 实验报告
│   ├── shopizer/
│   └── teammates/
└── superpowers-ref/       # Superpowers 参考实现（对照组）
```

## License

MIT
