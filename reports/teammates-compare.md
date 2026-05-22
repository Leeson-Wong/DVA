# TEAMMATES 变维拆分实验报告（第二轮）

## 实验信息

- **项目**: TEAMMATES/teammates (commit `91086fa`)
- **模型**: GLM-5.1
- **日期**: 2026-05-22
- **四环境**: base（无方法论）/ dims（变维拆分 v2）/ superpowers / dims+superpowers

## 变量控制改进（vs 第一轮）

1. 四个环境收到**完全相同的任务描述**
2. Prompt 模板统一（只有嵌入的方法论不同）
3. 初始代码库从同一 commit 复制，各自独立 git repo
4. metrics.sh 统一采集 before/after
5. 报告格式完全一致

## 五轮需求

| 轮次 | 需求 | 来源 Issue | 类型 |
|------|------|-----------|------|
| R01 | 提取通知方法到 NotificationsApi | #13959 | 结构拆分 |
| R02 | MCQ/MSQ 支持空权重 | #12547 | 类型行为扩展 |
| R03 | 移除 Notification.shown 属性 | #13944 | 属性删减 |
| R04 | 通知"全部标记已读" | #13471 | 新功能（基于 R03） |
| R05 | 允许跨 Section 同名 Team | #2618 | 跨实体改动 |

---

## 一、执行效率（真实数据）

### 按轮次 × 环境

| 轮次 | 环境 | 耗时(秒) | 工具调用数 | 修改文件数 |
|------|------|---------|-----------|-----------|
| R01 | base | 204 | 25 | 1 new + 1 mod |
| R01 | dims | 301 | 29 | 1 new + 1 mod |
| R01 | superpowers | 249 | 28 | 1 new + 1 mod |
| R01 | dims+super | 303 | 30 | 1 new + 1 mod |
| R02 | base | 309 | 43 | 2 mod |
| R02 | dims | **793** | **84** | **13 mod** |
| R02 | superpowers | 397 | 49 | 2+2 test |
| R02 | dims+super | 448 | 58 | 2+2 test |
| R03 | base | 364 | 45 | 19 mod |
| R03 | dims | **832** | **83** | **37 mod** |
| R03 | superpowers | 610 | 83 | 21 mod |
| R03 | dims+super | 537 | 53 | 17 mod |
| R04 | base | 246 | 29 | 4 mod |
| R04 | dims | 322 | 32 | 3 mod |
| R04 | superpowers | 329 | 39 | 4+1 test |
| R04 | dims+super | 348 | 39 | 4+1 test |
| R05 | base | 513 | 59 | 7 mod |
| R05 | dims | **1190** | **121** | **46 mod** |
| R05 | superpowers | 599 | 67 | 6+2 test |
| R05 | dims+super | **1357** | **130** | 7+1 test |

### 累计效率

| 环境 | 总耗时(秒) | 总工具调用 | 平均每轮耗时 |
|------|-----------|-----------|------------|
| base | 1,636 | 201 | 327s |
| dims | **3,448** | **349** | **690s** |
| superpowers | 2,184 | 266 | 437s |
| dims+super | **2,993** | **310** | **599s** |

---

## 二、代码改动量

### R01: 提取通知方法

| 指标 | base | dims | superpowers | dims+super |
|------|------|------|-------------|------------|
| NotificationsApi 行数 | 77 | 128 | 121 | 119 |
| Logic.java 变化 | +0行 | +1行 | +0行 | -1行 |
| 新增文件 | 1 | 1 | 1 | 1 |
| 总行数变化 | +77 | +134 | +136 | +127 |

### R02: MCQ/MSQ 空权重

| 指标 | base | dims | superpowers | dims+super |
|------|------|------|-------------|------------|
| 修改生产文件 | 2 | **13** | 2 | 2 |
| 新增测试文件 | 0 | 0 | 2 | 2 |
| 含前端改动 | 无 | **有(4TS)** | 无 | 无 |
| 总行数变化 | +4 | +124 | +4 | +2 |

### R03: 移除 shown 属性

| 指标 | base | dims | superpowers | dims+super |
|------|------|------|-------------|------------|
| 修改文件数 | 19 | **37** | 21 | 17 |
| 含前端改动 | 无 | **有(7TS)** | 无 | 无 |
| 含 migration | 有 | 有 | 有 | 有 |
| 总行数变化 | -29 | -29 | -29 | -33 |

### R04: 全部标记已读

| 指标 | base | dims | superpowers | dims+super |
|------|------|------|-------------|------------|
| 修改文件数 | 4 | 3 | 5 | 5 |
| 含测试 | 无 | 无 | 有 | 有 |
| NotificationsLogic 新增方法 | 1 | 1 | 1 | 1 |
| 总行数变化 | +73 | +50 | +53 | +64 |

### R05: 跨 Section Team

| 指标 | base | dims | superpowers | dims+super |
|------|------|------|-------------|------------|
| 修改文件数 | 7 | **46** | 6+2 test | 7+1 test |
| 引入复合键 | 无 | **有** | 无 | **有** |
| 总行数变化 | -21 | +42 | -27 | -21 |

---

## 三、关键发现

### 1. dims 环境一贯做最全面的改动

dims 在 R02(13文件)、R03(37文件)、R05(46文件) 都远超其他环境。变维分析引导 agent 识别出更完整的变化范围，包括前端 TypeScript 文件和其他环境忽略的关联代码。

### 2. 全面改动 ≠ 过度设计

第一轮实验中 dims 的 R05（运费计算）被认为是过度设计。但第二轮中，dims 的全面改动是有根据的：
- R02: MCQ/MSQ 权重确实涉及前端统计组件和 TypeScript 类型
- R03: shown 属性确实被前端组件引用
- R05: Team 唯一性确实影响反馈系统的 Team 查找逻辑

**变维分析的"查现有"步骤帮助 agent 找到了真实受影响的代码路径。**

### 3. "新建前必答"约束有效

四个环境中，只有 dims_superpowers 在 R04 明确记录了"未创建新文件"的决策理由：
> "所有更改都在现有类中内部化（维度分析确认变化率与现有方法一致）"

这表明更新后的 SKILL 的"新建前必答"约束在 dims+super 组合中生效。

### 4. superpowers 的测试优势

superpowers 和 dims+super 在 R02、R04、R05 都新增了测试文件，而 base 和 dims 没有。Superpowers 流程的 TDD 步骤确实促进了测试编写。

### 5. dims 单独使用时最慢但最全面

| 维度 | base | dims | superpowers | dims+super |
|------|------|------|-------------|------------|
| 速度 | 最快 | 最慢 | 中等 | 较慢 |
| 改动范围 | 最小 | **最全面** | 中等 | 中等偏上 |
| 测试覆盖 | 无 | 无 | **有** | **有** |
| 文档产出 | 无 | **变维表** | **日志** | **两者** |

### 6. 效率 vs 质量权衡

```
效率(快) → base > superpowers > dims+super > dims ← 全面(慢)
质量(测试) → superpowers ≈ dims+super > dims > base ← 无测试
全面(范围) → dims > dims+super > superpowers > base ← 最窄
```

---

## 四、与第一轮实验对比

| 维度 | Shopizer (R1) | TEAMMATES (R2) |
|------|-------------|---------------|
| God Class 类型 | if-else 链 | 纯委托 Façade |
| R01 任务 | 拆全部分发 | 只提取通知域 |
| dims 过度设计 | R05 运费（过度） | R02/R03/R05（合理全面） |
| "查现有" SKILL | 未使用 | **已使用，有效** |
| token 统计 | 自估 | 自估（API 不提供） |
| 超时失败 | 无 | R01 v1 全部超时（任务太大） |

### SKILL 改进效果

更新后的 dims SKILL（含"查现有"+"新建前必答"）在第二轮中表现更好：
- 没有出现第一轮 R05 的过度设计问题
- dims 的全面改动都有明确的变化速率分析支撑
- dims+super 的"新建前必答"约束产生了可追溯的决策记录

---

## 五、Token 使用（自估，非精确）

> **注意**: API 不向主 session 暴露子 agent 的 token 统计。以下数据来自各 agent 的自我估算，仅供参考。

| 轮次 | base | dims | superpowers | dims+super |
|------|------|------|-------------|------------|
| R01 | ~12K | ~13K | ~19K | ~11K |
| R02 | ~10K | **~57K** | ~43K | ~23K |
| R03 | ~18K | ~57K | ~23K | ~33K |
| R04 | ~20K | ~18K | ~19K | ~20K |
| R05 | N/A | N/A | N/A | N/A |

**代理指标（可靠）**: 总工具调用数 base(201) < superpowers(266) < dims+super(310) < dims(349)。

---

## 六、结论

1. **变维拆分引导更全面的分析**，但代价是耗时更长（2x base）
2. **变维拆分 v2 的"查现有"步骤有效**，没有出现第一轮的过度设计
3. **Superpowers 的 TDD 步骤促进了测试编写**，是其他环境缺少的
4. **dims+superpowers 组合**兼顾了全面分析和流程纪律，是质量-效率的最佳平衡点
5. **API token 统计缺失**是当前实验的硬伤，需要工具侧支持才能解决

---

## 附录：实验目录

```
F:\exam\experiment\teammates\
  PROTOCOL.md
  metrics.sh
  requirements\req-01.md ~ req-05.md
  envs\
    base\CLAUDE.md + repo\
    dims\CLAUDE.md + repo\
    superpowers\CLAUDE.md + repo\
    dims_superpowers\CLAUDE.md + repo\
  results\glm-5.1\
    <env>\round-01\ ~ round-05\
```
