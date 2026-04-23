# 数据库设计文档

## 概述

砺文工作室后端系统使用 Cloudflare D1 (SQLite) 数据库，包含以下核心数据表：

## 数据表结构

### 1. services（服务项目表）

| 字段名 | 类型 | 约束 | 说明 |
|--------|------|------|------|
| id | INTEGER | PRIMARY KEY AUTOINCREMENT | 唯一标识 |
| name | TEXT | NOT NULL | 服务名称 |
| service_type | TEXT | NOT NULL | 服务类型 |
| short_description | TEXT | | 简短描述 |
| price | INTEGER | DEFAULT 0 | 价格 |
| price_unit | TEXT | DEFAULT '元/项' | 价格单位 |
| detail | TEXT | | 详细说明 |
| is_active | INTEGER | DEFAULT 1 | 是否上架 |
| sort_order | INTEGER | DEFAULT 0 | 排序顺序 |
| created_at | DATETIME | DEFAULT CURRENT_TIMESTAMP | 创建时间 |
| updated_at | DATETIME | | 更新时间 |

**服务类型枚举：**
- `paper_edit` - 论文润色
- `exam_tutor` - 考研辅导
- `paper_check` - 查重降重
- `thesis_proposal` - 开题报告
- `graduation` - 毕业论文指导
- `sci_publish` - SCI/SSCI发表
- `other` - 其他

### 2. cases（成功案例表）

| 字段名 | 类型 | 约束 | 说明 |
|--------|------|------|------|
| id | INTEGER | PRIMARY KEY AUTOINCREMENT | 唯一标识 |
| title | TEXT | NOT NULL | 案例标题 |
| description | TEXT | NOT NULL | 案例描述 |
| image_url | TEXT | | 图片链接 |
| service_type | TEXT | NOT NULL | 关联服务类型 |
| result | TEXT | | 成果/结果 |
| status | TEXT | DEFAULT 'published' | 状态 |
| view_count | INTEGER | DEFAULT 0 | 浏览次数 |
| created_at | DATETIME | DEFAULT CURRENT_TIMESTAMP | 创建时间 |
| updated_at | DATETIME | | 更新时间 |

**状态枚举：**
- `published` - 已发布
- `draft` - 草稿
- `deleted` - 已删除

### 3. contacts（用户留言表）

| 字段名 | 类型 | 约束 | 说明 |
|--------|------|------|------|
| id | INTEGER | PRIMARY KEY AUTOINCREMENT | 唯一标识 |
| name | TEXT | NOT NULL | 用户姓名 |
| phone | TEXT | NOT NULL | 联系电话 |
| email | TEXT | | 电子邮箱 |
| service_type | TEXT | | 咨询的服务类型 |
| message | TEXT | NOT NULL | 留言内容 |
| status | TEXT | DEFAULT 'pending' | 处理状态 |
| admin_note | TEXT | | 管理员备注 |
| ip_address | TEXT | | IP地址 |
| processed_at | DATETIME | | 处理时间 |
| created_at | DATETIME | DEFAULT CURRENT_TIMESTAMP | 提交时间 |

**状态枚举：**
- `pending` - 待处理
- `processing` - 处理中
- `completed` - 已处理
- `deleted` - 已删除

## ER 关系图

```
┌─────────────┐       ┌─────────────┐       ┌─────────────┐
│  services   │       │   cases     │       │  contacts   │
├─────────────┤       ├─────────────┤       ├─────────────┤
│ id (PK)     │       │ id (PK)     │       │ id (PK)     │
│ name        │       │ title       │       │ name        │
│ service_type│◄──────┤ service_type│       │ phone       │
│ price       │       │ status      │       │ service_type│
│ is_active   │       │ view_count  │       │ status      │
└─────────────┘       └─────────────┘       └─────────────┘
```

## 索引设计

```sql
-- 留言状态索引
CREATE INDEX idx_contacts_status ON contacts(status);

-- 留言时间索引
CREATE INDEX idx_contacts_created ON contacts(created_at DESC);

-- 服务类型索引
CREATE INDEX idx_services_type ON services(service_type);

-- 案例状态索引
CREATE INDEX idx_cases_status ON cases(status);
```

## 安全说明

1. **数据脱敏**：前端展示时对手机号进行部分隐藏（如：138****8888）
2. **IP记录**：自动记录提交者IP地址，用于安全防护
3. **软删除**：使用状态标记代替物理删除，保留数据可追溯性
4. **输入验证**：所有用户输入均经过服务端验证和XSS过滤
