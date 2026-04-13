# Requirements: OASIS 通用实体关系行为传导网络框架重构

**Defined:** 2026-04-13
**Core Value:** 通过领域无关的抽象层，让同一套核心引擎能够模拟任何"实体-关系-行为"网络系统

## v1 Requirements

### 核心抽象层 (CORE)

- [ ] **CORE-01**: 定义 `Entity` 抽象基类，支持领域特定的状态字段和属性
- [ ] **CORE-02**: 定义 `Relation` 抽象基类，支持类型化、有向/无向、带权重的关系
- [ ] **CORE-03**: 定义 `Action` 抽象基类，包含前置条件、执行逻辑、后置效果
- [ ] **CORE-04**: 定义 `BehaviorStrategy` 接口，支持LLM驱动和规则驱动两种实现
- [ ] **CORE-05**: 定义 `DiscoveryStrategy` 接口，抽象推荐/发现机制
- [ ] **CORE-06**: 定义 `ConductionRule` 接口，描述行为如何在网络中传导
- [ ] **CORE-07**: 重构 `Environment` 核心，完全领域无关，通过配置驱动

### 社交领域插件 (SOCIAL)

- [ ] **SOCL-01**: 实现 `SocialEntity` 类，映射现有用户概念
- [ ] **SOCL-02**: 实现社交领域的关系类型：Follow, Block, Mute
- [ ] **SOCL-03**: 实现社交领域的23个行为：CreatePost, Like, Comment, Repost等
- [ ] **SOCL-04**: 实现社交领域的传导规则：关注者看到内容、点赞影响热度等
- [ ] **SOCL-05**: 实现社交领域的发现策略：兴趣推荐、热度排序、时间线
- [ ] **SOCL-06**: 创建社交领域配置文件，完全声明式定义领域模型
- [ ] **SOCL-07**: 现有所有社交模拟示例在新框架下运行，行为保持一致

### 配置与DSL (CONFIG)

- [ ] **CONF-01**: 设计领域配置Schema（JSON/YAML格式）
- [ ] **CONF-02**: 实现配置验证器，检查领域定义的完整性
- [ ] **CONF-03**: 实现配置加载器，将配置转换为运行时领域对象
- [ ] **CONF-04**: 支持实体类型定义（字段、类型、约束）
- [ ] **CONF-05**: 支持关系类型定义（方向性、属性、约束）
- [ ] **CONF-06**: 支持行为空间定义（可执行的操作列表）
- [ ] **CONF-07**: 支持传导规则定义（行为如何影响网络）

### 数据持久化抽象 (STORAGE)

- [ ] **STOR-01**: 抽象 `StateStorage` 接口，隔离具体数据库实现
- [ ] **STOR-02**: 实现 SQLite 存储后端（兼容现有）
- [ ] **STOR-03**: 实现 Neo4j 存储后端（兼容现有）
- [ ] **STOR-04**: 支持领域模型的动态Schema映射
- [ ] **STOR-05**: 支持实体状态、关系、行为的持久化与查询

### 行为传导引擎 (CONDUCTION)

- [ ] **COND-01**: 实现行为事件模型（ActionEvent）
- [ ] **COND-02**: 实现事件在关系网络中的传播机制
- [ ] **COND-03**: 实现传导规则的优先级和冲突解决
- [ ] **COND-04**: 支持传导链的可视化追踪
- [ ] **COND-05**: 支持传导效果的累积和衰减

### API兼容性 (COMPAT)

- [ ] **COMP-01**: 保留 `oasis.make()` 工厂函数签名
- [ ] **COMP-02**: 保留 `env.reset()`, `env.step()`, `env.close()` 生命周期
- [ ] **COMP-03**: 保留 `AgentGraph` 概念，泛化为 `EntityGraph`
- [ ] **COMP-04**: 现有示例代码无需修改即可运行（或最小修改）
- [ ] **COMP-05**: 提供迁移指南和兼容性层

## v2 Requirements

### 其他领域示例 (EXAMPLES)

- **EXAM-01**: 供应链网络模拟示例
- **EXAM-02**: 流行病传播模拟示例
- **EXAM-03**: 金融市场传导模拟示例
- **EXAM-04**: 知识传播/学习网络模拟示例

### 高级功能 (ADVANCED)

- **ADVN-01**: 可视化领域建模工具（图形化配置）
- **ADVN-02**: 领域模板市场（可分享的领域配置）
- **ADVN-03**: 运行时领域热切换
- **ADVN-04**: 多领域混合模拟（实体跨领域交互）

## Out of Scope

| Feature | Reason |
|---------|--------|
| 分布式模拟 | 单节点性能已支持百万代理，分布式增加复杂度 |
| 实时3D可视化 | 专注于网络结构模拟，可视化走导出/插件路线 |
| 新的LLM提供商 | 保持现有CAMEL集成，这是基础设施非核心 |
| Web UI管理界面 | 命令行和配置文件优先，Web界面作为v2+考虑 |

## Traceability

| Requirement | Phase | Status |
|-------------|-------|--------|
| CORE-01 | Phase 1 | Pending |
| CORE-02 | Phase 1 | Pending |
| CORE-03 | Phase 1 | Pending |
| CORE-04 | Phase 1 | Pending |
| CORE-05 | Phase 2 | Pending |
| CORE-06 | Phase 2 | Pending |
| CORE-07 | Phase 3 | Pending |
| SOCL-01 | Phase 4 | Pending |
| SOCL-02 | Phase 4 | Pending |
| SOCL-03 | Phase 4 | Pending |
| SOCL-04 | Phase 5 | Pending |
| SOCL-05 | Phase 5 | Pending |
| SOCL-06 | Phase 5 | Pending |
| SOCL-07 | Phase 6 | Pending |
| CONF-01 | Phase 2 | Pending |
| CONF-02 | Phase 2 | Pending |
| CONF-03 | Phase 2 | Pending |
| CONF-04 | Phase 3 | Pending |
| CONF-05 | Phase 3 | Pending |
| CONF-06 | Phase 3 | Pending |
| CONF-07 | Phase 3 | Pending |
| STOR-01 | Phase 1 | Pending |
| STOR-02 | Phase 4 | Pending |
| STOR-03 | Phase 4 | Pending |
| STOR-04 | Phase 4 | Pending |
| STOR-05 | Phase 4 | Pending |
| COND-01 | Phase 2 | Pending |
| COND-02 | Phase 2 | Pending |
| COND-03 | Phase 3 | Pending |
| COND-04 | Phase 6 | Pending |
| COND-05 | Phase 6 | Pending |
| COMP-01 | Phase 4 | Pending |
| COMP-02 | Phase 4 | Pending |
| COMP-03 | Phase 4 | Pending |
| COMP-04 | Phase 6 | Pending |
| COMP-05 | Phase 6 | Pending |

**Coverage:**
- v1 requirements: 36 total
- Mapped to phases: 36
- Unmapped: 0 ✓

---
*Requirements defined: 2026-04-13*
*Last updated: 2026-04-13 after initial definition*
