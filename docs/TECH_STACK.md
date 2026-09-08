# Panequill 技术栈

> 状态：v0.1
>
> 本文档定义 Panequill 当前采用的开发技术、核心依赖与明确不采用的技术方案。
>
> 产品范围以 `PRODUCT.md` 为准，架构边界以 `ARCHITECTURE.md` 为准。本文档不得覆盖或重新定义产品与架构规则。

## 1. 选型原则

Panequill 是个人使用、本地优先的 Web Dashboard。技术选型遵循以下原则：

1. 优先满足当前明确需求，不为假设中的未来场景增加技术复杂度。
2. 优先选择维护活跃、类型支持完整、与 React/Vite 生态直接兼容的方案。
3. 同一个问题只保留一套主要技术方案，不并行维护重复能力。
4. 能通过轻量依赖解决的问题，不引入完整框架或大型抽象层。
5. 浏览器端交互与本地状态必须能够脱离云端请求正常运行。
6. 依赖版本升级以兼容性和实际收益为依据，不追求无意义的版本追新。

## 2. 技术栈总览

| 层级 | 技术 | 当前基线 | 用途 |
| --- | --- | --- | --- |
| Runtime | Node.js | 24 LTS | 本地开发、构建与测试 |
| Package Manager | npm | 随 Node.js LTS | 依赖管理与锁定 |
| UI Framework | React | 19.x | 前端界面与组件系统 |
| Language | TypeScript | 5.x | 静态类型 |
| Build Tool | Vite | 8.x | 开发服务器与生产构建 |
| React Integration | `@vitejs/plugin-react` | 与 Vite 8 兼容版本 | React Fast Refresh 与 JSX 支持 |
| Styling | Tailwind CSS | 4.x | UI 样式 |
| Tailwind Integration | `@tailwindcss/vite` | 与 Tailwind 4 兼容版本 | Tailwind 与 Vite 集成 |
| Application State | Zustand | 5.x | Dashboard 内存状态 |
| Runtime Validation | Zod | 4.x | 持久化与云端数据运行时校验 |
| Local Persistence | `idb-keyval` | 6.x | IndexedDB 轻量封装 |
| Widget Layout | `react-grid-layout` | 2.x | Widget 拖动、缩放与响应式网格 |
| Cloud Client | `@supabase/supabase-js` | 2.x | Supabase Auth 与 PostgreSQL 访问 |
| Unit / Component Test | Vitest | 5.x | 单元与组件测试 |
| React Test Utilities | Testing Library | 当前兼容稳定版本 | React 组件行为测试 |
| Static Analysis | ESLint | 当前兼容稳定版本 | 代码静态检查 |

具体安装的补丁版本以 `package.json` 与 `package-lock.json` 为准。

## 3. Runtime 与包管理

### 3.1 Node.js 24 LTS

项目开发环境使用 Node.js 24 LTS。

不使用 Current 版本作为项目基线，避免开发环境跟随短周期版本变化。

### 3.2 npm

包管理器固定使用 npm。

原因：

- Node.js LTS 原生提供；
- Panequill 当前规模不需要额外包管理器能力；
- 减少开发环境前置依赖。

仓库应提交 `package-lock.json`，保证依赖解析结果可复现。

当前不使用：

- pnpm；
- Yarn；
- Bun 作为包管理器或运行时。

除非后续出现 npm 无法合理解决的明确问题，否则不更换包管理器。

## 4. React + TypeScript + Vite

### 4.1 React 19

Panequill 使用 React 19 构建 Dashboard 与 Widget 组件系统。

React 负责：

- UI 渲染；
- 组件生命周期；
- Widget 组件组合；
- 交互状态与视图更新。

不把 React Context 作为全局 Dashboard 状态管理方案。

### 4.2 TypeScript

所有应用代码使用 TypeScript。

`tsconfig` 必须启用严格类型检查，至少以 `strict: true` 为基础。

禁止通过大量 `any`、无验证的类型断言或关闭严格规则来绕过类型问题。

### 4.3 Vite 8

Panequill 是纯客户端 Web 应用，使用 Vite 作为开发服务器与生产构建工具。

使用官方 React 插件：

```text
@vitejs/plugin-react
```

当前不使用：

- Next.js；
- Remix；
- Astro；
- 自定义 Webpack 构建；
- `@vitejs/plugin-react-swc`。

Panequill 当前没有 SSR、SSG、Server Components 或服务端路由需求，因此不引入全栈 Web Framework。

## 5. 样式系统

### 5.1 Tailwind CSS 4

Panequill 使用 Tailwind CSS 4 负责组件样式与响应式布局。

与 Vite 集成使用：

```text
@tailwindcss/vite
```

### 5.2 Design Token

全局视觉规则不应散落为大量任意值。

需要稳定复用的设计值，例如：

- Widget 圆角；
- 背景透明度；
- 边框；
- 阴影；
- 间距；
- 前景与背景颜色；

应由 `DESIGN.md` 定义，并优先通过 CSS Variables / Tailwind theme token 暴露给组件使用。

`TECH_STACK.md` 不定义具体视觉数值。

### 5.3 UI 组件库

当前不引入完整 UI 组件库。

不预装：

- Material UI；
- Ant Design；
- Chakra UI；
- Mantine；
- shadcn/ui 全量组件体系。

Panequill 的主要界面由自有 Dashboard、Widget 与基础交互组件组成。只有出现明确且重复的组件需求后，再评估是否引入单个辅助库。

## 6. 应用状态：Zustand 5

Panequill 使用 Zustand 维护运行中的 Dashboard 应用状态。

Zustand 负责：

- 当前 Dashboard 内存状态；
- Widget 与 Layout 状态更新；
- Preferences 等需要驱动 UI 的状态；
- 为组件提供按需状态订阅。

Zustand 不负责替代 IndexedDB，也不直接定义云端同步协议。

状态关系保持为：

```text
IndexedDB → Zustand → UI
                │
                └── Sync Service → Supabase
```

当前不采用：

- Redux / Redux Toolkit；
- MobX；
- 将所有全局状态放入 React Context；
- Zustand `persist` + `localStorage` 作为 Dashboard 主持久化方案。

## 7. 数据运行时校验：Zod 4

TypeScript 只能提供编译期类型，无法证明 IndexedDB、Supabase 或 JSON 中的数据在运行时符合当前 Schema。

因此 Panequill 使用 Zod 4 对持久化数据进行运行时校验。

典型数据流：

```text
IndexedDB / Supabase / JSON
            │
            ▼
          unknown
            │
            ▼
           Zod
            │
            ▼
     DashboardState
```

Dashboard 核心数据类型应尽量由 Zod Schema 推导 TypeScript 类型，避免分别维护两套容易漂移的定义。

不得使用未经校验的 `as DashboardState` 直接信任外部持久化数据。

## 8. IndexedDB：idb-keyval 6

`ARCHITECTURE.md` 已确定 IndexedDB 为本地持久化方案。

当前 Dashboard 本地存储模型以少量 key-value 状态为主，因此使用 `idb-keyval` 作为 IndexedDB 轻量封装。

业务代码不得在各组件中直接散落 `idb-keyval` 调用，应通过项目自己的 storage 模块访问，例如：

```text
loadDashboardState()
saveDashboardState(state)
```

这样业务层依赖的是 Panequill 自己的存储接口，而不是第三方库 API。

当前不采用 Dexie。

原因不是 Dexie 能力不足，而是 Panequill 当前不需要：

- 多表数据库模型；
- 索引查询；
- 复杂事务；
- Live Query；
- IndexedDB ORM 能力。

如果以后出现真实的复杂本地查询需求，再重新评估；当前不提前引入。

## 9. Widget 网格：react-grid-layout 2

Widget 布局采用 `react-grid-layout` v2 API。

使用范围：

- Widget 拖动；
- Widget 尺寸调整；
- 网格定位；
- 最小 / 最大尺寸约束；
- 响应式 breakpoint 布局；
- Layout 序列化。

只使用 v2 API，不使用：

```text
react-grid-layout/legacy
```

Panequill 自己的数据模型仍然是权威状态，`react-grid-layout` 只负责布局计算和交互，不成为独立数据源。

当前不采用 GridStack。

GridStack 提供的嵌套网格、跨网格拖动、复杂动态子网格等能力超出 Panequill 当前需求，因此不引入额外能力与抽象。

## 10. Supabase Client

浏览器端使用：

```text
@supabase/supabase-js 2.x
```

其职责仅限于 `ARCHITECTURE.md` 已定义的 Supabase 能力：

- Auth；
- PostgreSQL 数据访问；
- 与 RLS 配合访问个人 Dashboard 数据。

Supabase 调用应封装在项目数据访问层，不允许 UI 组件普遍直接调用：

```ts
supabase.from(...)
```

建议边界：

```text
UI / Store
    │
    ▼
Sync Service / Repository
    │
    ▼
Supabase Client
```

当前不使用：

- Supabase Realtime；
- Supabase Storage；
- Supabase Edge Functions；
- Supabase SSR 包。

这些限制来自 `ARCHITECTURE.md`，不得由实现自行改变。

## 11. 路由与 Server State

### 11.1 不使用 React Router

当前 Panequill 只有一个核心 Dashboard 应用入口，没有真实的多页面路由需求。

设置、编辑等功能优先使用 Dashboard 内部交互形态，不因为“以后可能有多个页面”提前加入 Router。

出现明确 URL 路由需求后再重新评估。

### 11.2 不使用 TanStack Query

Panequill 已有明确的数据流：

```text
IndexedDB
    ↕
Application State
    ↕
Sync Service
    ↕
Supabase
```

当前不再增加一层独立 Server State Cache。

不使用：

- TanStack Query；
- SWR。

避免同时维护 Zustand、IndexedDB、Query Cache 与 Supabase 四份状态来源。

## 12. 测试技术

### 12.1 Vitest 5

单元测试与组件测试使用 Vitest 5。

优先覆盖：

- Dashboard Schema；
- 状态更新逻辑；
- 本地 Storage 封装；
- Layout 数据转换；
- 同步版本比较；
- 冲突检测；
- 对关键 Widget 交互逻辑进行必要的组件测试。

### 12.2 Testing Library

React 组件行为测试使用 Testing Library。

测试应优先从用户可观察行为验证组件，而不是依赖组件内部实现细节。

### 12.3 E2E

第一阶段不预装 Playwright。

当项目出现需要真实浏览器验证的完整流程，例如：

```text
拖动 Widget → 持久化 → 刷新页面 → 恢复布局
```

再引入 Playwright。

不得为了“以后可能会用”提前建立空的 E2E 测试框架。

## 13. 静态检查

使用 ESLint 做代码静态检查。

规则以 React + TypeScript 当前官方/主流兼容配置为基础，保持必要且直接，不建立复杂自定义规则体系。

TypeScript 类型检查与 ESLint 都属于提交前基本验证，不允许通过关闭规则来规避真实错误。

## 14. 浏览器基线

Panequill 面向现代浏览器，不提供 legacy browser 兼容层。

由于 Tailwind CSS 4 使用现代 CSS 能力，最低浏览器基线按其支持要求确定：

- Chrome / Chromium 111+；
- Edge 111+；
- Safari / iOS Safari 16.4+；
- Firefox 128+。

不加入：

- IE 支持；
- legacy bundle；
- 为明显过旧浏览器准备的大型 polyfill 集；
- 为不在基线内的浏览器维护第二套 CSS。

如果未来个人实际设备出现兼容问题，应针对真实设备与功能单独评估，而不是提前扩大兼容范围。

## 15. 当前明确不采用的技术

除前文已经说明的项目外，当前不采用：

- Next.js 或其他全栈 React Framework；
- Redux；
- Dexie；
- GridStack；
- React Router；
- TanStack Query / SWR；
- Supabase Realtime；
- Supabase Storage；
- Supabase Edge Functions；
- 独立 Panequill API Server；
- WebSocket 状态同步；
- Service Worker / PWA 框架；
- 微前端；
- CSS-in-JS 运行时方案。

其中架构级限制以 `ARCHITECTURE.md` 为最终依据。

## 16. 初始依赖结构

建立项目骨架时，依赖按职责最小化安装。

核心运行依赖：

```text
react
react-dom
zustand
zod
idb-keyval
react-grid-layout
@supabase/supabase-js
```

样式与构建相关：

```text
vite
@vitejs/plugin-react
typescript
tailwindcss
@tailwindcss/vite
```

测试与质量相关：

```text
vitest
@testing-library/react
@testing-library/user-event
@testing-library/jest-dom
eslint
```

除脚手架运行所需依赖外，不因为模板默认推荐而自动加入未在本文档中确定的库。

## 17. 版本管理规则

本文档主要固定技术的大版本与职责，不长期锁定补丁版本。

实际项目遵循：

1. `package.json` 声明项目依赖范围；
2. `package-lock.json` 锁定实际安装版本；
3. `package-lock.json` 必须提交仓库；
4. 升级大版本前必须检查 breaking changes；
5. 不因为存在新版本就自动升级大版本；
6. 不安装 beta、canary、next、rc 等预发布版本作为默认依赖，除非任务明确要求并说明原因。

## 18. 技术栈变更原则

新增或替换核心技术前必须先确认：

1. 是否存在当前已确认的真实需求；
2. 现有技术是否确实无法合理解决；
3. 新依赖是否与 `PRODUCT.md`、`ARCHITECTURE.md` 和 `DESIGN.md` 冲突；
4. 是否只是为了未来假设场景而增加复杂度。

如果现有技术能够直接解决问题，应优先继续使用现有技术，而不是引入第二套实现。
