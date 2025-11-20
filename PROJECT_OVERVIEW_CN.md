# Maybe 项目详细文档

## 目录
- [1. 项目概述](#1-项目概述)
- [2. 技术栈](#2-技术栈)
- [3. 核心功能](#3-核心功能)
- [4. 项目结构](#4-项目结构)
- [5. 数据库架构](#5-数据库架构)
- [6. API 架构](#6-api-架构)
- [7. 认证与授权](#7-认证与授权)
- [8. 数据集成](#8-数据集成)
- [9. 后台任务](#9-后台任务)
- [10. 前端架构](#10-前端架构)
- [11. 开发指南](#11-开发指南)
- [12. 部署说明](#12-部署说明)

---

## 1. 项目概述

### 什么是 Maybe？

Maybe 是一个**开源的个人财务管理应用**，用户可以自行托管，完全掌控自己的财务数据和隐私。它是商业个人理财平台的开源替代方案，采用 AGPLv3 许可证发布。

### 解决的问题

- 提供注重隐私的财务数据管理
- 实现完整的账户聚合，无需依赖第三方数据提供商
- 支持自托管，用户拥有完整的数据所有权
- 提供实时银行同步和投资追踪
- 提供消费洞察、预算管理和财务分析工具

### 应用模式

- **托管模式（Managed）**：Maybe 团队为用户运营服务器
- **自托管模式（Self-Hosted）**：用户通过 Docker 在自己的基础设施上托管

---

## 2. 技术栈

### 后端技术

- **Rails 7.2.2**（Ruby 3.4.4）
- **PostgreSQL 9.3+**（支持自定义类型和枚举）
- **Redis 5.4**（用于缓存和后台任务）
- **Puma**（Web 服务器）
- **Sidekiq + Sidekiq-Cron**（后台任务处理）

### 前端技术

- **Hotwire 栈**：Turbo Rails + Stimulus.js
- **ViewComponent**（可复用 UI 组件）
- **Tailwind CSS v4.x**（自定义设计系统）
- **D3.js**（财务数据可视化）
- **Biome**（JavaScript/TypeScript 代码检查和格式化）

### 核心依赖

| 类别 | 依赖库 |
|------|--------|
| **认证** | Doorkeeper (OAuth2)、bcrypt、ROTP (多因素认证) |
| **数据集成** | Plaid SDK、CSV 导入/导出 |
| **API 安全** | Rack Attack (限流)、JWT |
| **AI** | ruby-openai (OpenAI 集成) |
| **支付** | Stripe |
| **监控** | Sentry、Skylight、Logtail、Vernier |
| **文件存储** | AWS S3、Cloudflare R2、本地磁盘 |
| **图像处理** | Image Processing、libvips |
| **工具** | Jbuilder (JSON)、pagy (分页)、activerecord-import (批量操作) |

---

## 3. 核心功能

### 3.1 账户管理

- 支持 8+ 种账户类型：
  - 存款账户（Depository）
  - 投资账户（Investment）
  - 加密货币账户（Crypto）
  - 房产（Property）
  - 车辆（Vehicle）
  - 信用卡（Credit Card）
  - 贷款（Loan）
  - 其他资产/负债
- 实时 Plaid 同步，支持 12,000+ 家金融机构
- 手动 CSV 导入交易、交易记录和余额
- 账户状态追踪（活跃、草稿、禁用、待删除）
- 账户对账和余额追踪

### 3.2 交易管理

- 分层类别分类（收入/支出）
- 自定义商家检测和标签
- 交易标签系统
- 批量编辑和过滤
- 交易类型：
  - `standard`：标准交易
  - `funds_movement`：资金转移
  - `cc_payment`：信用卡还款
  - `loan_payment`：贷款还款
  - `one_time`：一次性交易
- AI 自动分类
- 基于规则的交易自动化

### 3.3 投资追踪

- 证券持仓追踪
- 交易执行记录（数量和价格）
- 证券价格历史（每日市场数据）
- 持仓对账
- 支持多个交易所和货币

### 3.4 财务分析

- 资产负债表生成
- 损益表生成
- 净资产随时间追踪
- 消费趋势和分析
- 多币种支持和自动汇率转换
- 房产、车辆等资产估值追踪

### 3.5 预算管理

- 按时间段创建预算
- 基于类别的预算分配
- 预算与实际支出对比
- 收入预测

### 3.6 数据导入/导出

- CSV 导入（自定义字段映射）
- 交易导入
- 交易/持仓导入
- 家庭数据导出功能
- 导入模板和可复用配置

### 3.7 用户与家庭管理

- 多用户家庭支持
- 用户角色：成员、管理员、超级管理员
- 家庭成员邮件邀请
- 多因素认证（MFA）
- 个人资料管理（包括图片上传）
- 用户停用/清除

### 3.8 AI 功能

- 财务洞察聊天界面
- AI 驱动的自动分类
- AI 商家检测
- 数据分析工具调用

### 3.9 API

- 外部 REST API（`/api/v1`）
  - OAuth2 和 API 密钥认证
  - 每个 API 密钥的速率限制
  - 作用域权限（read、read_write）
  - 账户、交易、聊天和使用情况端点
- 内部基于 Turbo 的 JSON API（用于 Web UI）

---

## 4. 项目结构

```
maybe/
├── app/
│   ├── assets/              # Tailwind CSS、字体、图片
│   ├── channels/            # WebSocket 频道
│   ├── components/          # ViewComponents（UI + 设计系统）
│   │   ├── DS/             # 设计系统基础组件（按钮、对话框等）
│   │   └── UI/             # 应用特定组件
│   ├── controllers/         # Rails 控制器
│   │   ├── api/v1/         # 外部 API 控制器
│   │   └── *.rb            # Web UI 控制器
│   ├── data_migrations/     # 数据迁移任务
│   ├── helpers/            # Rails 视图助手
│   ├── javascript/         # Stimulus 控制器
│   ├── jobs/               # Sidekiq 后台任务
│   ├── mailers/            # ActionMailer 类
│   ├── models/             # ActiveRecord 模型（199 个文件）
│   │   ├── concerns/       # 共享模块
│   │   ├── account/        # 账户特定模型
│   │   ├── balance/        # 余额相关逻辑
│   │   ├── chat/           # AI 聊天相关
│   │   ├── holding/        # 持仓和证券
│   │   └── [其他领域模型]
│   ├── services/           # API 速率限制器（按惯例最少使用）
│   └── views/              # ERB 模板
├── config/
│   ├── routes.rb           # 路由定义
│   ├── initializers/       # Rails 初始化器
│   └── environments/       # 环境特定配置
├── db/
│   ├── migrate/            # 数据库迁移
│   ├── schema.rb           # 数据库模式定义
│   └── seeds.rb            # 数据库种子数据
├── lib/                    # 自定义库（货币处理）
├── test/                   # Minitest 测试文件
├── bin/                    # 可执行文件（setup、dev、rails）
└── .devcontainer/          # 开发容器配置
```

---

## 5. 数据库架构

### 核心层次结构

```
User（用户）→ belongs_to Family（家庭）
Family → has_many Users, Accounts, Transactions, Categories, Tags, Rules, Budgets

Account（账户）→ belongs_to Family
  ├── has_many Entries（条目）
  ├── has_many Balances（余额）
  └── has_many Holdings（持仓）

Entry（条目）→ belongs_to Account
  ├── entryable_type: Transaction, Trade, Valuation

Transaction（交易）
  ├── belongs_to Category（类别）
  ├── belongs_to Merchant（商家）
  ├── has_many Tags（标签）
  └── kind: standard, funds_movement, cc_payment, loan_payment, one_time

Trade（交易记录）
  ├── belongs_to Security（证券）
  └── 追踪数量、价格、货币

Valuation（估值）
  └── 非流动资产的时点价值

Security（证券）→ has_many Holdings, Trades, SecurityPrices
Holding（持仓）→ 追踪某日的数量、价格、金额

Category（类别）→ 分层（父/子），分类（收入/支出）
Budget（预算）→ has_many BudgetCategories，按日期范围
```

### 关键数据表

| 类别 | 表名 |
|------|------|
| **用户与认证** | `users`, `families`, `sessions` |
| **账户与余额** | `accounts`, `balances` |
| **交易数据** | `entries`, `transactions`, `trades`, `valuations` |
| **投资** | `securities`, `security_prices`, `holdings` |
| **分类与标签** | `categories`, `tags`, `taggings`, `merchants` |
| **预算** | `budgets`, `budget_categories` |
| **导入** | `imports`, `import_rows`, `import_mappings` |
| **Plaid 集成** | `plaid_items`, `plaid_accounts` |
| **同步** | `syncs`（带状态机的异步数据同步） |
| **API 认证** | `api_keys`, `oauth_*` 表 |
| **订阅** | `subscriptions`, `family_exports` |
| **规则自动化** | `rules`, `rule_conditions`, `rule_actions` |
| **AI 功能** | `chats`, `messages`, `tool_calls` |

---

## 6. API 架构

### 外部 API (`/api/v1`)

**基础控制器**：`Api::V1::BaseController`

**认证方式**：
1. **OAuth2**（通过 Doorkeeper）
   - 作用域：`read`、`read_write`
2. **API 密钥**（JWT 令牌）
   - Header：`X-Api-Key`
   - 每个密钥的速率限制

**端点**：
- `POST /auth/signup`, `/auth/login`, `/auth/refresh`
- `GET /accounts`（仅索引）
- `GET/POST/PUT/DELETE /transactions`
- `GET /usage`（速率限制信息）
- `GET/POST/PUT/DELETE /chats`（嵌套消息）
- AI 消息重试功能

**响应格式**：通过 Jbuilder 模板生成 JSON

**速率限制 Headers**：
- `X-RateLimit-Limit`
- `X-RateLimit-Remaining`
- `X-RateLimit-Reset`

### 内部 API（Turbo）

- 控制器返回 JSON 用于 Turbo Frame 替换
- 使用 Rails 助手进行认证
- 基于会话的认证，使用 `Current.user` 上下文

---

## 7. 认证与授权

### Web 认证

- 基于 `Session` 模型的会话认证
- 邮箱 + 密码（bcrypt 哈希）
- 邮箱确认工作流
- 密码重置令牌（15 分钟过期）
- 邮箱更改带确认

### 多因素认证（MFA）

- 基于 TOTP（基于时间的一次性密码）
- 通过 ROTP gem 实现
- 为验证器应用生成 QR 码
- 备份码（8 个码，单次使用）
- 每个用户可选强制启用

### API 认证

1. **OAuth2**（Doorkeeper）：
   - 授权码流程
   - 带作用域的访问令牌
   - 刷新令牌
   - 通过 OAuth 应用支持移动设备

2. **API 密钥**：
   - 用户生成的密钥
   - 作用域：read、read_write
   - 每个密钥的速率限制
   - 最后使用时间戳追踪
   - 撤销功能

### 授权

- 基于家庭的访问控制（用户只能看到自己家庭的数据）
- 基于角色：member（成员）、admin（管理员）、super_admin（超级管理员）
- API 访问的作用域（分层：`read_write` 包含 `read`）

---

## 8. 数据集成

### 8.1 Plaid 集成

**目的**：实时银行账户同步

**流程**：
1. 用户通过 Plaid Link UI 创建 `PlaidItem`
2. 系统存储访问令牌和 item ID
3. 定时同步获取交易、余额、投资数据
4. 数据存储在 `PlaidAccount` 中并同步到 `Account` + `Entry`

**模型**：
- `PlaidItem`：Plaid 连接项
- `PlaidAccount`：Plaid 账户
- `PlaidEntry`：Plaid 条目

**功能**：
- 支持多个地区（美国、欧盟）
- 机构元数据（名称、URL、颜色）
- 多个产品（交易、投资、负债）
- 错误恢复和重试逻辑

**Webhooks**：处理实时交易更新和 item 状态变更

### 8.2 CSV 导入系统

**导入类型**：
- `TransactionImport`：交易导入
- `TradeImport`：交易记录导入
- `AccountImport`：账户导入
- `MintImport`：Mint 数据导入

**流程**：
1. 上传 CSV 文件
2. 配置列映射
3. 设置解析选项（日期格式、符号约定、数字格式）
4. 预览和验证
5. 发布以创建条目
6. 可回滚功能

**功能**：
- 自定义字段映射和转换规则
- 模板保存以供复用
- 使用 `activerecord-import` 批量导入
- 数据验证和错误报告
- 导入行追踪用于审计

### 8.3 汇率与市场数据

- **Synth API**：提供历史汇率和证券价格
- **ExchangeRate 模型**：追踪多币种转换率
- **SecurityPrice 模型**：证券历史价格
- **自动同步**：后台任务导入市场数据

### 8.4 第三方服务

- **Stripe**：支付处理和订阅管理
- **OpenAI**：AI 聊天和自动分类
- **Synth**：汇率和市场数据提供商

---

## 9. 后台任务

### Sidekiq 任务类型

| 任务 | 用途 |
|------|------|
| `SyncJob` | 编排账户/Plaid 数据同步 |
| `SyncCleanerJob` | 标记过期同步，清理 |
| `ImportJob` | 处理 CSV 导入 |
| `RevertImportJob` | 回滚导入数据 |
| `AutoCategorizeJob` | AI 驱动的交易分类 |
| `AutoDetectMerchantsJob` | 通过 AI 检测商家 |
| `RuleJob` | 应用自动化规则到交易 |
| `AssistantResponseJob` | 生成 AI 聊天响应 |
| `FamilyDataExportJob` | 导出家庭财务数据 |
| `FamilyResetJob` | 清除家庭数据 |
| `ImportMarketDataJob` | 同步证券价格和汇率 |
| `SecurityHealthCheckJob` | 验证证券数据 |
| `StripeEventHandlerJob` | 处理 Stripe webhooks |
| `UserPurgeJob` | 删除已停用用户数据 |
| `DataCacheClearJob` | 使聚合缓存失效 |
| `DestroyJob` | 异步模型销毁 |

### 调度

- 通过 `sidekiq-cron` 的定时任务（在 `config/schedule.yml` 中配置）
- 通过 `perform_later` 入队异步执行
- 错误报告到 Sentry

---

## 10. 前端架构

### 10.1 Hotwire 优先方法

- **Turbo Frames**：用于部分页面更新
- **Turbo Streams**：通过 WebSockets 实时更新
- **ViewComponents**：可复用 UI
- **原生 HTML 元素**：优先使用 `<dialog>`、`<details>` 而非 JS 组件

### 10.2 Stimulus 控制器（28+ 个）

| 控制器 | 用途 |
|--------|------|
| `app_layout_controller` | 主布局交互 |
| `auto_submit_form_controller` | 变更时自动提交 |
| `bulk_select_controller` | 多选操作 |
| `category_controller` | 类别管理 UI |
| `chat_controller` | 聊天界面 |
| `donut_chart_controller` | D3.js 甜甜圈图可视化 |
| `file_upload_controller` | 文件处理 |
| `import_controller` | 导入工作流 |
| `modal_controller` | 对话框处理 |
| `bulk_update_controller` | 批量操作 |
| 图表控制器 | line、donut、sankey |
| 表单助手 | auto-submit、日期选择器等 |

### 10.3 ViewComponents

**设计系统（DS）**：30+ 个基础组件
- Button、Link、Dialog、Disclosure、Menu、Tabs
- Icon、Avatar、Badge、Tooltip、Alert

**UI 组件**：页面特定组件
- AccountPage、TransactionsList、BudgetForm

### 10.4 样式

- **Tailwind CSS v4.x**
- 自定义设计令牌在 `maybe-design-system.css` 中
- 功能性令牌：
  - `text-primary`（而非 `text-white`）
  - `bg-container`（而非 `bg-white`）
  - `border-primary`（而非 `border-gray-200`）
- 暗黑模式通过数据属性：`[data-theme=dark]`
- 图标系统通过 Lucide Icons 助手

### 10.5 关键模式

- 服务端渲染 + Turbo 增强
- 使用查询参数管理状态
- 服务端货币/日期格式化
- 渐进式增强（无 JS 也能工作）

---

## 11. 开发指南

### 11.1 常用命令

**开发服务器**：
```bash
bin/dev                    # 启动开发服务器（Rails + Sidekiq + Tailwind）
bin/rails server          # 仅启动 Rails 服务器
bin/rails console         # 打开 Rails 控制台
```

**测试**：
```bash
bin/rails test                              # 运行所有测试
bin/rails test:db                           # 重置数据库并运行测试
bin/rails test:system                       # 仅运行系统测试
bin/rails test test/models/account_test.rb # 运行特定测试文件
bin/rails test test/models/account_test.rb:42 # 运行特定行的测试
```

**代码检查与格式化**：
```bash
bin/rubocop               # 运行 Ruby 代码检查
npm run lint              # 检查 JavaScript/TypeScript 代码
npm run lint:fix          # 修复 JavaScript/TypeScript 问题
npm run format            # 格式化 JavaScript/TypeScript 代码
bin/brakeman              # 运行安全分析
```

**数据库**：
```bash
bin/rails db:prepare      # 创建并迁移数据库
bin/rails db:migrate      # 运行待处理的迁移
bin/rails db:rollback     # 回滚上一次迁移
bin/rails db:seed         # 加载种子数据
```

**初始设置**：
```bash
bin/setup                 # 初始项目设置（安装依赖、准备数据库）
```

### 11.2 PR 前的 CI 工作流

在开启 Pull Request 前**始终**运行这些命令：

1. **测试**（必需）：
   ```bash
   bin/rails test           # 运行所有测试（始终必需）
   bin/rails test:system    # 运行系统测试（仅在适用时）
   ```

2. **代码检查**（必需）：
   ```bash
   bin/rubocop -f github -a                # Ruby 代码检查自动修正
   bundle exec erb_lint ./app/**/*.erb -a  # ERB 代码检查自动修正
   ```

3. **安全检查**（必需）：
   ```bash
   bin/brakeman --no-pager  # 安全分析
   ```

**只有在所有检查通过后才能创建 PR**。

### 11.3 开发规范

#### 规范 1：最小化依赖
- 在添加新依赖前尽力使用 Rails
- 新依赖需要有强有力的技术/业务理由
- 优先选择成熟可靠的而非新潮的

#### 规范 2：瘦控制器，胖模型
- 业务逻辑放在 `app/models/` 文件夹，避免 `app/services/`
- 使用 Rails concerns 和 POROs 组织代码
- 模型应该回答关于自己的问题：`account.balance_series` 而非 `AccountSeries.new(account).call`

#### 规范 3：Hotwire 优先的前端
- **优先使用原生 HTML** 而非 JS 组件
  - 使用 `<dialog>` 做模态框，`<details><summary>` 做折叠
- **利用 Turbo frames** 进行页面区块更新
- **使用查询参数管理状态** 而非 localStorage/sessions
- **服务端格式化** 货币、数字、日期
- **始终使用 `icon` 助手**，**永不直接使用 `lucide_icon`**

#### 规范 4：优化简洁性
- 优先考虑良好的 OOP 领域设计而非性能
- 仅在关键/全局区域关注性能（避免 N+1 查询，注意全局布局）

#### 规范 5：数据库 vs ActiveRecord 验证
- 简单验证（空值检查、唯一索引）在数据库层
- ActiveRecord 验证用于表单便利性（优先考虑客户端）
- 复杂验证和业务逻辑在 ActiveRecord

### 11.4 认证上下文

- 使用 `Current.user` 获取当前用户（**不使用** `current_user`）
- 使用 `Current.family` 获取当前家庭（**不使用** `current_family`）

### 11.5 测试理念

- **始终使用 Minitest + fixtures**（永不使用 RSpec 或 factories）
- 保持 fixtures 最小化（每个模型 2-3 个基础用例）
- 在测试上下文中动态创建边缘用例
- **仅测试关键和重要的代码路径**
- **正确测试边界**：
  - 命令：测试它们被调用时使用正确的参数
  - 查询：测试输出
  - 不要测试其他类的实现细节

---

## 12. 部署说明

### 12.1 Docker 支持

- 提供 Dockerfile 用于容器化
- `compose.example.yml` 用于 Docker Compose 设置
- 支持 S3 和 Cloudflare R2 文件存储
- SMTP 配置用于邮件
- 需要 PostgreSQL 数据库

### 12.2 自托管配置

- 所有配置使用环境变量
- 可选的敏感数据加密密钥
- 凭证系统用于机密信息
- Redis 用于缓存（必需）

### 12.3 监控与可观测性

- **Sentry**：错误追踪
- **Skylight**：性能监控
- **Logtail**：日志聚合
- **Vernier**：性能分析
- 请求日志用于审计追踪

---

## 13. 同步与数据管理系统

### 同步架构

- **状态机**：pending（待处理）、syncing（同步中）、completed（已完成）、failed（失败）、stale（过期）
- 父子同步关系（并行操作）
- 基于窗口的同步（支持日期范围的增量同步）
- `Sync` 模型追踪所有异步数据操作
- 24 小时过期超时，带清理任务
- 家庭同步时间戳用于缓存失效

### 同步流程

1. 创建 `Sync` 记录为 pending 状态
2. 通过 Sidekiq 入队 `SyncJob`
3. Job 将同步标记为 syncing 并调用 `syncable.perform_sync(sync)`
4. 完成/失败时，转换到相应状态
5. 触发 `perform_post_sync` 进行最后步骤
6. 通过 Turbo 广播同步完成

### 可同步对象

- `Family`：主同步协调
- `Account`：单个账户同步
- `PlaidItem`：Plaid 数据获取

---

## 14. 附加功能

### 多币种支持

- 所有货币值以基础货币存储（用户的主要货币）
- 从 Synth API 获取汇率
- `Money` 对象处理货币转换和格式化
- 历史汇率用于准确报告

### 规则与自动化

- 匹配的条件树（AND/OR 逻辑）
- 操作：分类、商家分配、标签
- 基于日期的生效日期
- 手动或自动应用到交易

### 商家

- 家庭商家（每个家庭自定义）
- 提供商商家（来自 Plaid/数据提供商）
- 通过 AI 自动检测
- Logo/网站 URL 元数据

### 数据丰富

- 通用丰富框架
- 来源：rule、plaid、synth、ai
- 用于交易/账户元数据增强

### 对账

- 账户级对账状态
- 账户间转账匹配
- 拒绝的转账追踪

---

## 15. 项目文件结构总结

```
maybe/
├── README.md              # 项目概述
├── CLAUDE.md              # 开发指南
├── CONTRIBUTING.md        # 贡献指南
├── Gemfile                # Ruby 依赖
├── package.json           # Node 依赖（Biome）
├── Rakefile               # Rake 任务
├── Dockerfile             # 容器镜像
├── compose.example.yml    # Docker Compose 模板
├── .env.example           # 环境变量模板
│
├── app/                   # 应用代码（14 个目录）
├── config/                # 配置文件
├── db/                    # 数据库模式和迁移
├── lib/                   # 自定义库
├── bin/                   # 可执行文件（setup、dev、rails）
├── test/                  # 测试文件
├── .cursor/rules/         # Cursor AI 项目规则
└── .devcontainer/         # 开发容器配置
```

---

## 16. 快速开始

### 首次设置

```bash
# 1. 克隆仓库
git clone <repository-url>
cd maybe

# 2. 运行设置脚本
bin/setup

# 3. 启动开发服务器
bin/dev
```

### 访问应用

- Web 界面：`http://localhost:3000`
- Lookbook（组件库）：`http://localhost:3000/lookbook`
- Sidekiq 控制台：`http://localhost:3000/sidekiq`（需要管理员权限）

---

## 17. 贡献指南

1. Fork 项目
2. 创建特性分支（`git checkout -b feature/amazing-feature`）
3. 提交更改（`git commit -m 'Add some amazing feature'`）
4. 推送到分支（`git push origin feature/amazing-feature`）
5. 开启 Pull Request

**注意**：在创建 PR 前，务必运行所有 CI 检查（测试、代码检查、安全检查）。

---

## 18. 许可证

该项目采用 AGPLv3 许可证发布。详见 LICENSE 文件。

---

## 19. 联系方式

- GitHub Issues：报告 bug 和功能请求
- 文档：查看 CLAUDE.md 获取详细开发指南

---

**最后更新**：2025-11-20
