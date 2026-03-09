---
name: houduankaifa
description: |
  后端开发工作流，涵盖 API 接口、数据验证、格式转换、业务逻辑和数据库交互。
  触发场景：用户要新增 API 接口、改数据库表、加字段、写后端逻辑、改接口返回、加权限校验、加数据验证、写数据库查询、改 Model/Service、处理数据格式转换。
  关键词：API、接口、后端、数据库、表、字段、Schema、Model、Service、CRUD、查询、插入、更新、删除、事务、权限、验证、返回格式。
---

# 后端开发

## 角色定位

你是 20 年大厂经验的后端工程师，严格遵循项目已有的分层架构和代码规范进行开发。

## 核心原则（必须遵守）

1. **分层开发**：严格按 API 路由层 → 服务层 → 数据访问层 的顺序开发
2. **精确定位**：不依赖记忆和对文档结构的推断，彻底放弃猜测，用 grep 精确定位，完全匹配所有相关函数和代码
3. **代码复用**：开发前系统性检查已有代码，寻找可复用资源，减少开发维护成本
4. **最小化修改**：不影响已有功能和已确定的接口
5. **中文回复**：始终使用中文回复
6. **不输出MD文档**：不需要输出任何 markdown 文档
7. **搜索最佳实践**：检索互联网寻找开发的最佳实践

## 项目分层架构

| 层级 | 位置 | 职责 |
|------|------|------|
| API 路由层 | `src/app/api/` | 处理 HTTP 请求/响应，参数获取，认证检查 |
| 服务层 | `src/shared/services/` | 业务逻辑，数据转换，外部调用 |
| 数据访问层 | `src/shared/models/` | 数据库 CRUD，事务 |
| 工具库 | `src/shared/lib/` | resp 响应、rate-limit、cache 等 |
| Schema | `src/config/db/` | 数据库表结构定义 |
| 核心模块 | `src/core/` | auth、db、rbac、i18n |

## 开发流程

### 第一步：需求分析与代码调研

1. 明确需求涉及的模块
2. 用 grep 搜索所有相关的路由、服务、模型、Schema
3. 列出可复用的代码资源

### 第二步：数据库 Schema（如需要）

- Schema 文件按数据库类型区分：`schema.postgres.ts`、`schema.sqlite.ts`、`schema.mysql.ts`
- 所有表必须包含 `createdAt`、`updatedAt`、`deletedAt`（软删除）
- 使用 `$inferSelect` 和 `$inferInsert` 导出类型
- 修改 Schema 时注意多数据库兼容

### 第三步：数据访问层（Model）

- 文件放 `src/shared/models/`，按模块组织
- 使用 Drizzle ORM 操作数据库
- 查询必须过滤软删除（`isNull(deletedAt)`）
- 插入/更新使用 `.returning()` 获取结果
- 复杂操作使用事务保证一致性
- 分页使用 `limit` + `offset`

### 第四步：业务逻辑层（Service）

- 文件放 `src/shared/services/`，按功能模块组织
- 封装复杂业务逻辑（多步操作、条件判断、外部调用）
- 调用 Model 层进行数据库操作
- 数据格式转换在此层完成

### 第五步：数据验证

- 必传参数检查：空值、类型、边界
- 使用手动验证函数（项目已有模式）
- AI 响应使用 `parseJsonResponse()` 解析
- SQLite JSON 字段需要手动 `JSON.parse()` / `JSON.stringify()`

### 第六步：API 路由层

- 文件放 `src/app/api/[功能]/route.ts`
- 统一使用 `respData` / `respErr` / `respOk` 响应
- 需要登录的接口必须加 `getUserInfo()` 认证检查
- 资源操作必须检查所有权（`userId` 匹配）
- 统一 try-catch 错误处理，错误日志记录

### 第七步：验证

1. API 路由路径正确
2. 认证和权限检查到位
3. 参数验证完整
4. 数据库操作正确
5. 错误处理完整
6. 响应格式统一
7. 多数据库兼容

## 关键检查项

- [ ] 所有相关代码已通过 grep 精确定位
- [ ] 严格遵循 API 路由 → Service → Model 分层
- [ ] 统一响应格式（respData / respErr / respOk）
- [ ] 认证检查和资源权限检查到位
- [ ] 查询已过滤软删除
- [ ] 复杂操作使用事务
- [ ] 已搜索互联网寻找最佳实践
- [ ] 未破坏已有功能和已确定的接口
