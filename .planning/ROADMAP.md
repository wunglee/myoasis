# Roadmap: OASIS 通用实体关系行为传导网络框架重构

**Created:** 2026-04-13
**Granularity:** Fine
**Total Requirements:** 36 v1 requirements
**Total Phases:** 6

---

## Phase 1: 核心抽象层基础

**Goal:** 建立领域无关的核心抽象接口

**Requirements:** CORE-01, CORE-02, CORE-03, CORE-04, STOR-01

**Success Criteria:**
1. Entity, Relation, Action, BehaviorStrategy 抽象基类已定义并可通过单元测试
2. 抽象层完全领域无关，不包含任何社交领域概念
3. 存储抽象接口已定义，可支持多种后端
4. 代码通过类型检查，接口文档完整

**Phase Dependencies:** 无（基础层）

---

## Phase 2: 配置系统与传导规则

**Goal:** 实现领域配置DSL和传导机制基础

**Requirements:** CORE-05, CORE-06, CONF-01, CONF-02, CONF-03, COND-01, COND-02

**Success Criteria:**
1. 领域配置Schema定义完成，支持JSON/YAML格式
2. 配置验证器可检测领域定义的错误和缺失
3. DiscoveryStrategy 和 ConductionRule 接口已定义
4. 行为事件模型和基础传播机制实现
5. 至少一个示例配置可通过验证并加载

**Phase Dependencies:** Phase 1（依赖核心抽象）

---

## Phase 3: 环境重构与配置驱动

**Goal:** 重构Environment为完全配置驱动，完成配置系统

**Requirements:** CORE-07, CONF-04, CONF-05, CONF-06, CONF-07, COND-03

**Success Criteria:**
1. Environment 核心完全领域无关，通过配置注入领域逻辑
2. 支持完整的领域配置：实体类型、关系类型、行为空间、传导规则
3. 冲突解决机制实现（多规则优先级）
4. 配置加载器可将配置转换为运行时对象图
5. 核心框架可独立运行（无需社交插件）

**Phase Dependencies:** Phase 1, Phase 2

---

## Phase 4: 社交领域插件与存储实现

**Goal:** 将现有社交功能封装为首个领域插件，实现存储后端

**Requirements:** SOCL-01, SOCL-02, SOCL-03, STOR-02, STOR-03, STOR-04, STOR-05, COMP-01, COMP-02, COMP-03

**Success Criteria:**
1. SocialEntity 实现 Entity 接口，包含现有用户所有属性
2. 社交关系类型（Follow/Block/Mute）实现 Relation 接口
3. 社交行为（CreatePost, Like等23个）实现 Action 接口
4. SQLite和Neo4j存储后端实现 StateStorage 接口
5. 动态Schema映射支持社交领域数据模型
6. 工厂函数和生命周期API保持向后兼容
7. EntityGraph 作为 AgentGraph 的泛化替代

**Phase Dependencies:** Phase 1, Phase 2, Phase 3

---

## Phase 5: 社交传导与发现策略

**Goal:** 实现社交领域的传导规则和发现机制

**Requirements:** SOCL-04, SOCL-05, SOCL-06

**Success Criteria:**
1. 社交传导规则实现：关注者接收内容、点赞提升热度、转发扩散等
2. 社交发现策略实现：兴趣推荐、热度排序、时间线算法
3. 完整的社交领域配置文件，声明式定义所有领域概念
4. 传导效果可追踪和验证（单元测试覆盖主要场景）

**Phase Dependencies:** Phase 4

---

## Phase 6: 兼容性验证与迁移完成

**Goal:** 确保现有功能100%兼容，完成迁移文档

**Requirements:** SOCL-07, COND-04, COND-05, COMP-04, COMP-05

**Success Criteria:**
1. 所有现有examples/目录下的示例在新框架下运行正常
2. 行为一致性测试通过（新旧框架输出对比）
3. 传导链可视化工具可用（调试/分析用途）
4. 完整的迁移指南文档
5. 性能测试：重构后性能下降<20%

**Phase Dependencies:** Phase 4, Phase 5

---

## Summary

| Phase | Name | Requirements | Est. Effort | Status |
|-------|------|--------------|-------------|--------|
| 1 | 核心抽象层基础 | 5 | Medium | Not Started |
| 2 | 配置系统与传导规则 | 7 | Medium | Not Started |
| 3 | 环境重构与配置驱动 | 6 | High | Not Started |
| 4 | 社交领域插件与存储 | 10 | High | Not Started |
| 5 | 社交传导与发现策略 | 3 | Medium | Not Started |
| 6 | 兼容性验证与迁移 | 5 | Medium | Not Started |

**Total:** 36 requirements across 6 phases

---

## Dependency Graph

```
Phase 1 (核心抽象)
    ↓
Phase 2 (配置系统)
    ↓
Phase 3 (环境重构)
    ↓
Phase 4 (社交插件)
    ↓
├── Phase 5 (传导策略)
│       ↓
└── Phase 6 (兼容性验证)
```

---

*Last updated: 2026-04-13*
