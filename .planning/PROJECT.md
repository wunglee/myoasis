# OASIS 架构重构：通用实体关系行为传导网络模拟框架

## What This Is

将现有的OASIS社交模拟器重构为一个**通用的实体关系行为传导网络模拟框架**。

新框架将支持：
- **抽象实体（Entity）**：不再局限于"用户"，可以是任何具有状态和行为能力的对象
- **抽象关系（Relation）**：不再局限于"关注"，可以是任何实体间的连接类型
- **抽象行为（Action）**：不再局限于"发帖/点赞"，可以是任何领域特定的操作
- **抽象环境（Environment）**：不再局限于"社交平台"，可以是任何传导网络场景

社交关系模拟（Twitter/Reddit风格）将成为框架的一个**可配置领域插件**。

## Core Value

**领域无关的通用性**：通过清晰的抽象层次和插件机制，让同一套核心引擎能够模拟任何"实体-关系-行为"网络系统，而不需要重写核心逻辑。

## Requirements

### Validated (现有能力，需保留并抽象化)

- ✓ 异步多代理执行与并发控制（已有，需抽象为实体调度器）
- ✓ 基于消息传递的实体间通信（已有，需抽象为传导通道）
- ✓ LLM驱动的行为决策（已有，需抽象为行为策略接口）
- ✓ 可插拔的推荐/发现机制（已有，需抽象为发现策略）
- ✓ 图结构关系管理（已有，需抽象为关系图谱）
- ✓ 数据持久化（已有，需抽象为状态存储）

### Active (本次重构目标)

- [ ] **领域抽象层设计**：定义Entity、Relation、Action、Environment核心抽象接口
- [ ] **社交领域插件**：将现有社交功能封装为首个领域实现
- [ ] **配置驱动架构**：通过配置而非代码来定义领域模型
- [ ] **行为传导机制**：将行为效果建模为网络中的信号传导
- [ ] **领域特定语言（DSL）**：简化新领域定义的配置语法

### Out of Scope

- 暂不实现除社交外的其他完整领域示例（保持聚焦）
- 暂不改变存储后端（保持SQLite/Neo4j）
- 暂不引入新的LLM提供商（保持现有集成）

## Context

### 现有架构痛点

当前OASIS深度耦合于社交媒体领域：
- 数据库表结构硬编码（user, post, follow, like...）
- Agent行为逻辑内嵌社交平台语义
- 难以扩展到非社交场景（如：供应链网络、流行病传播、金融市场等）

### 目标架构愿景

```
┌─────────────────────────────────────────────────────────┐
│                    领域配置层 (Domain Config)              │
│         实体定义 | 关系类型 | 行为空间 | 传导规则           │
├─────────────────────────────────────────────────────────┤
│                    抽象核心层 (Core Framework)            │
│    实体调度器 | 关系图谱 | 行为引擎 | 传导网络 | 发现机制    │
├─────────────────────────────────────────────────────────┤
│                    基础设施层 (Infrastructure)            │
│      LLM集成 | 数据存储 | 并发控制 | 消息通道 | 可视化      │
└─────────────────────────────────────────────────────────┘
```

### 关键设计原则

1. **倒置依赖**：核心框架不依赖具体领域，领域依赖核心框架
2. **组合优于继承**：通过配置组合行为，而非继承覆盖
3. **事件驱动**：行为触发事件，事件在网络中传导
4. **策略模式**：可插拔的行为策略、发现策略、传导策略

## Constraints

- **兼容性**：现有社交模拟功能必须通过新框架实现，行为保持一致
- **性能**：重构后性能不应显著下降（<20%）
- **渐进式**：支持分阶段迁移，而非一次性重写

## Key Decisions

| Decision | Rationale | Outcome |
|----------|-----------|---------|
| 社交功能外置为插件 | 验证框架通用性 | — Pending |
| 引入DSL配置 | 降低新领域接入门槛 | — Pending |
| 保留PettingZoo风格API | 用户接口稳定性 | — Pending |

## Evolution

This document evolves at phase transitions and milestone boundaries.

**After each phase transition** (via `/gsd-transition`):
1. Requirements invalidated? → Move to Out of Scope with reason
2. Requirements validated? → Move to Validated with phase reference
3. New requirements emerged? → Add to Active
4. Decisions to log? → Add to Key Decisions
5. "What This Is" still accurate? → Update if drifted

**After each milestone** (via `/gsd-complete-milestone`):
1. Full review of all sections
2. Core Value check — still the right priority?
3. Audit Out of Scope — reasons still valid?
4. Update Context with current state

---
*Last updated: 2026-04-13 after initialization*
