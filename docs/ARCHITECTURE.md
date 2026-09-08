# Panequill 架构设计

> 状态：v0.1
>
> 本文档定义 Panequill 当前实现架构。涉及“怎么实现”时，以本文档为准；产品范围以 `PRODUCT.md` 为准。

## 1. 架构目标

Panequill 的架构只需要满足个人 Dashboard 的实际需求：

1. 页面打开速度优先；
2. 本地交互不依赖网络往返；
3. 同一个人的多设备配置能够同步；
4. 云端暂时不可用时仍可使用本地已有状态；
5. 部署和维护成本尽可能低；
6. 不为公开服务、多租户或团队协作预留复杂架构。

## 2. 总体架构

```text
┌─────────────────────────────┐
│           Browser           │
│                             │
│  Panequill Web Dashboard    │
│           │                 │
│           ▼                 │
│  Application State          │
│           │                 │
│     ┌─────┴─────┐           │
│     ▼           ▼           │
│ IndexedDB    UI Render      │
│     │                       │
└─────┼───────────────────────┘
      │
      │ ordinary HTTPS sync
      ▼
┌─────────────────────────────┐
│          Supabase           │
│                             │
│  Auth                       │
│  PostgreSQL                 │
│  Row Level Security         │
└─────────────────────────────┘

Microsoft Bing Daily Wallpaper
             │
             └───────────────► Browser
```

核心关系是：

- 浏览器本地状态负责日常运行；
- IndexedDB 负责本地持久化；
- Supabase 负责身份验证和跨设备云端状态；
- Bing Daily Wallpaper 是独立外部数据源。

## 3. 状态模型

Panequill 当前维护一个统一的 Dashboard 状态对象。

建议基础结构：

```json
{
  "schemaVersion": 1,
  "widgets": [],
  "layout": {},
  "shortcuts": [],
  "preferences": {}
}
```

字段含义：

- `schemaVersion`：Dashboard 数据结构版本，用于未来必要的数据迁移；
- `widgets`：Widget 实例及其持久化配置；
- `layout`：Widget 的位置、尺寸和布局信息；
- `shortcuts`：快捷入口；
- `preferences`：界面及行为偏好。

不把临时 UI 状态、请求状态、缓存结果等无须跨设备持久化的数据写入该对象。

## 4. 本地存储

### 4.1 选择 IndexedDB

Dashboard 的持久化本地状态使用 IndexedDB。

原因：

- 比 `localStorage` 更适合保存结构化状态；
- 不阻塞主线程进行同步 I/O；
- 容量和扩展性更合适；
- 后续需要增加本地缓存时不必更换存储方案。

`localStorage` 仅可用于非常小且无需结构化管理的启动级标记；没有实际需求时不额外使用。

### 4.2 本地数据职责

IndexedDB 至少保存：

```text
dashboard_state
├── data
├── remote_version
├── updated_at
└── sync_status
```

其中：

- `data`：完整 Dashboard 状态；
- `remote_version`：最后一次确认的云端版本号；
- `updated_at`：本地最后修改时间，仅用于本地状态管理；
- `sync_status`：是否存在待同步修改。

浏览器应能够仅凭本地 `data` 渲染已有 Dashboard。

## 5. Supabase

### 5.1 使用范围

Supabase 第一阶段只使用：

- Auth；
- PostgreSQL；
- Row Level Security。

第一阶段不使用：

- Realtime；
- Storage；
- Edge Functions；
- 复杂 RPC；
- 多角色权限模型。

除非后续出现明确需求，否则不增加这些能力。

### 5.2 数据表

第一阶段使用单一核心表即可：

```text
dashboard_state
├── user_id       uuid
├── data          jsonb
├── version       bigint
└── updated_at    timestamptz
```

建议约束：

- `user_id` 为主键或唯一键；
- 每个账号只保留一条当前 Dashboard 状态；
- `data` 保存完整 Dashboard JSON；
- `version` 每次成功云端写入后递增；
- `updated_at` 由数据库更新。

不把 Widget、快捷入口、偏好分别拆成多张表，除非以后出现必须单独查询或独立生命周期的真实需求。

## 6. 身份验证与访问控制

Supabase Auth 仅用于访问个人数据。

数据库必须启用 Row Level Security。

核心安全规则：

```text
auth.uid() = user_id
```

要求：

- 匿名用户不能读取或修改 Dashboard；
- 已登录用户只能访问自己的 Dashboard 行；
- 前端查询条件不能替代 RLS；
- Supabase `service_role` key 不得出现在浏览器端；
- 浏览器只能使用允许公开暴露的 Supabase 客户端配置。

由于 Panequill 不提供公开注册，账号创建方式属于部署配置，不需要为此建设产品内注册流程。

## 7. 启动流程

页面启动遵循“本地先显示，云端后同步”。

```text
打开 Panequill
      │
      ▼
读取 IndexedDB
      │
      ├── 有本地状态 ──► 立即渲染
      │
      └── 无本地状态 ──► 使用初始默认状态
      │
      ▼
检查登录状态
      │
      ▼
读取 Supabase 云端状态
      │
      ▼
比较本地与云端版本
      │
      ▼
按同步规则更新状态
```

不允许为了等待 Supabase 请求而阻塞已有 Dashboard 的首屏渲染。

## 8. 本地修改流程

用户发生需要持久化的修改时：

```text
用户修改
   │
   ▼
更新内存状态
   │
   ▼
立即更新 UI
   │
   ▼
写入 IndexedDB
   │
   ▼
标记待同步
   │
   ▼
debounce
   │
   ▼
同步 Supabase
```

Widget 拖动、尺寸调整等高频操作不能每个事件都直接请求 Supabase。

同步写入应进行 debounce，在连续修改停止后再提交完整状态。

具体 debounce 时长属于实现细节，不在架构文档中固定为某个毫秒值。

## 9. 云端同步模型

### 9.1 基本原则

Panequill 采用“本地优先 + 单份云端快照”的同步方式。

不实现事件日志、CRDT、操作级同步或实时协作。

### 9.2 版本号

云端 `dashboard_state.version` 是跨设备比较的权威版本。

本地记录最后一次确认的 `remote_version`。

每次成功写入云端后，云端版本递增，本地更新对应的 `remote_version`。

版本号用于判断云端状态是否发生过其他设备修改，不依赖不同设备的本地系统时间排序。

### 9.3 正常拉取

当本地没有待同步修改时：

- 云端版本高于本地 `remote_version`：使用云端状态更新本地；
- 云端版本等于本地 `remote_version`：无需处理；
- 首次设备且云端已有状态：直接使用云端状态初始化本地。

### 9.4 正常推送

存在本地待同步修改时，客户端提交：

- 完整 Dashboard 状态；
- 当前已知的 `remote_version`。

云端只在当前版本仍等于客户端已知版本时接受更新，并将版本递增。

这样可以检测“另一台设备在此期间已经修改过云端状态”。

### 9.5 冲突处理

Panequill 是个人使用工具，并发修改属于低频情况，因此第一阶段不实现自动字段级合并。

检测到版本冲突时：

1. 不静默覆盖云端状态；
2. 保留当前本地待同步状态；
3. 拉取最新云端状态；
4. 由界面明确提示存在其他设备的新版本；
5. 允许用户选择保留本地版本或采用云端版本。

不实现 CRDT、三方合并或复杂冲突引擎。

## 10. 同步触发时机

第一阶段同步触发点保持有限：

- 应用启动并完成身份确认后拉取；
- 本地持久化数据修改后 debounce 推送；
- 页面重新获得焦点时，可在距上次拉取已有合理间隔的情况下检查云端版本。

不使用持续轮询。

不使用 Supabase Realtime。

## 11. 离线与异常

### 11.1 云端不可用

Supabase 请求失败时：

- 不影响当前本地 Dashboard 使用；
- 本地修改继续保存到 IndexedDB；
- 保持待同步状态；
- 后续合适的同步触发点再次尝试。

### 11.2 首次使用且无本地数据

如果首次使用时 Supabase 同时不可用，则使用应用内默认状态启动。

### 11.3 数据损坏

读取本地或云端状态时必须进行数据结构校验。

不能直接假定 JSON 数据始终满足当前 Schema。

校验失败时不得用错误数据覆盖另一份有效状态。

## 12. Schema 版本

Dashboard 状态包含：

```json
{
  "schemaVersion": 1
}
```

仅在数据结构发生不兼容变化时增加版本。

迁移逻辑只处理实际存在的历史版本，不提前建立通用插件化迁移框架。

## 13. Bing Daily Wallpaper

Bing Daily Wallpaper 独立于 Dashboard 同步系统。

原则：

- 浏览器按需要获取每日壁纸；
- 壁纸图片本身不写入 Supabase；
- 不使用 Supabase Storage 缓存壁纸；
- Dashboard 数据只在确有必要时保存与壁纸展示有关的小型状态，例如当前显示日期；
- 不引入其他壁纸提供商抽象层。

如果浏览器直接请求 Bing 存在跨域或稳定性限制，再针对该明确问题增加最小代理层；在问题出现之前不预先建设壁纸服务。

## 14. 前端与后端边界

前端负责：

- Dashboard 渲染；
- Widget 生命周期；
- 本地状态管理；
- IndexedDB；
- 同步调度；
- Supabase 客户端访问；
- Bing Daily Wallpaper 展示。

Supabase 负责：

- Auth；
- Dashboard 云端快照；
- RLS 权限边界；
- 云端版本号和更新时间。

当前不设置独立 Panequill 应用服务器。

## 15. 不采用的架构

当前明确不采用：

- 自建 PostgreSQL；
- 自建 Auth 服务；
- 独立 REST API 后端；
- WebDAV 主同步；
- Supabase Realtime；
- CRDT；
- 消息队列；
- 微服务；
- 多数据库；
- Redis；
- 对象存储保存 Bing 壁纸；
- 多租户数据模型。

这些方案只有在出现当前架构无法解决的明确需求时才重新评估。

## 16. 数据流总结

### 启动

```text
IndexedDB → 立即渲染 → Supabase 拉取 → 必要时刷新本地
```

### 修改

```text
UI → 内存状态 → IndexedDB → debounce → Supabase
```

### 离线

```text
UI → 内存状态 → IndexedDB → 保留待同步 → 后续重试
```

### 壁纸

```text
Bing Daily Wallpaper → Browser
```

## 17. 架构变更原则

修改架构前首先判断需求是否属于 `PRODUCT.md` 定义的产品范围。

如果需求不在产品范围内，应先更新产品文档，而不是直接扩展实现。

技术实现应优先选择当前需求所需的最小方案，不提前为假设中的公开服务、多用户或商业场景增加复杂度。
