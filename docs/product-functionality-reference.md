# Element Web/Desktop — 产品功能交互说明文档

> 基于 Element Web v1.12.21 源码分析整理，用于类似端产品自主开发的参考。

---

## 目录

1. [产品概述与架构](#1-产品概述与架构)
2. [认证与登录](#2-认证与登录)
3. [主界面布局](#3-主界面布局)
4. [聊天消息](#4-聊天消息)
5. [房间管理](#5-房间管理)
6. [空间（Spaces/社区）](#6-空间spaces社区)
7. [右侧面板](#7-右侧面板)
8. [语音与视频通话](#8-语音与视频通话)
9. [语音消息](#9-语音消息)
10. [通知系统](#10-通知系统)
11. [搜索功能](#11-搜索功能)
12. [设置与偏好](#12-设置与偏好)
13. [Widget 小部件与集成](#13-widget-小部件与集成)
14. [桌面应用（Electron）](#14-桌面应用electron)
15. [斜杠命令](#15-斜杠命令)
16. [线程（Threads）](#16-线程threads)
17. [文件处理](#17-文件处理)
18. [Markdown 与富文本](#18-markdown-与富文本)
19. [表情与回应](#19-表情与回应)
20. [无障碍](#20-无障碍)
21. [定制化与扩展](#21-定制化与扩展)

---

## 1. 产品概述与架构

### 1.1 产品定位

Element 是基于 Matrix 协议的全功能即时通讯客户端，支持 Web 浏览器和 Electron 桌面应用两种形态。它支持端到端加密、群聊、语音/视频通话、文件共享、空间社区等功能。

### 1.2 技术架构

| 层级 | 技术选型 |
|------|---------|
| 前端框架 | React 19 + TypeScript 6 |
| 状态管理 | Flux 模式（MatrixDispatcher + AsyncStore） |
| UI 组件库 | Compound Design System（@vector-im/compound-web） |
| 构建工具 | Webpack（web 端）、Electron Builder（桌面端） |
| 测试 | Vitest（单元）、Playwright（E2E） |
| 包管理 | pnpm monorepo + Nx 任务编排 |

### 1.3 Monorepo 结构

```
apps/web/          — Element Web 应用
apps/desktop/      — Element Desktop（Electron 壳）
packages/          — 共享组件库与模块 API
modules/           — 内建扩展模块
docs/              — 项目文档
```

### 1.4 核心架构模式

- **Dispatcher 模式**：所有跨组件通信通过 `MatrixDispatcher` 中央调度，定义了 70+ 种 Action 类型
- **Store 模式**：Store 继承 `AsyncStore` / `AsyncStoreWithClient`，注册到 Dispatcher，变更时发射事件
- **MVVM 迁移**：新组件使用 ViewModel 模式（`useViewModel(vm)`），视图组件在 `packages/shared-components/`
- **路由**：基于 URL Hash 的简单路由（`#/room/!roomId`, `#/login`, `#/welcome` 等）
- **Context 注入**：`MatrixClientContext`、`RoomContext`、`SDKContext` 等 React Context 提供依赖注入

---

## 2. 认证与登录

### 2.1 启动流程

```
浏览器打开 → index.ts（检查浏览器特性）→ 并行加载：
  ├── 应用配置（config.json）
  ├── 多语言资源
  ├── 主题
  ├── Olm 加密库
  └── 插件/模块
→ init.tsx → app.tsx → MatrixChat 根组件 → initSession()
```

### 2.2 登录方式

| 方式 | 说明 |
|------|------|
| **密码登录** | 支持 Matrix ID、邮箱、手机号三种标识符 |
| **SSO 单点登录** | 支持 CAS、OAuth/SAML 等多类 SSO 提供商 |
| **OIDC 原生登录** | OpenID Connect 原生认证，支持标准 OIDC Provider |
| **Token 登录** | 通过 SSO 返回的 loginToken 完成登录 |
| **QR 码登录** | 基于 MSC4108 的扫码登录，使用 rendezvous 安全通道 |
| **访客模式** | 可选支持，用于预览公开房间 |

### 2.3 完整登录界面

#### 欢迎页（/#/welcome）
- 品牌 Logo + 背景图
- 操作按钮：登录、注册、QR 登录、浏览公开房间
- 语言选择器
- 底部链接（Blog、GitHub、Mastodon 等）

#### 登录页（/#/login）
- 用户名/邮箱/手机号切换
- 密码输入（含 zxcvbn 强度检测）
- "忘记密码?" 链接
- SSO 按钮（OIDC、SAML、CAS 等）
- 服务器选择器

#### 注册页（/#/register）
- 用户名（异步校验可用性）
- 密码（强度评分 ≥3/5）+ 确认密码
- 邮箱（可选/必填，视服务器要求）
- 手机号（可选/必填，含国家代码）
- 交互式认证阶段（邮件验证、手机验证、reCAPTCHA、条款同意）
- OIDC 原生注册（`prompt=create`）

#### 忘记密码（/#/forgot_password）
多阶段流程：
1. 输入邮箱 → 发送验证邮件
2. 检查邮箱提示（可重新发送、修改邮箱）
3. 输入新密码 + 确认
4. 可选"登出其他设备"
5. 完成提示 → 返回登录

#### 软登出（Soft Logout）
当服务端 Token 失效但本地数据仍在时触发：
- 保留设备 ID 重新认证（密码或 SSO）
- 或"清除所有数据"完全登出

#### 加密设置（首次登录后）
- **CompleteSecurity**：验证新设备（交叉签名）
- **E2eSetup**：初始化加密设置（设置密钥备份、验证设备）
- **LoginSplashView**：等待首次同步完成的加载动画

### 2.4 会话管理

| 功能 | 说明 |
|------|------|
| 会话恢复 | 从 localStorage/IndexedDB 恢复 Token |
| Token 刷新 | OIDC 会话自动刷新 access_token |
| 会话锁 | 多标签页间协调，防止冲突 |
| 登出 | OIDC 用户调用 OP 撤销 Token；普通用户调用 /logout API |
| 登出后处理 | 清除所有本地存储，可选重定向到外部 URL |

### 2.5 认证配置项

```json
{
  "disable_guests": false,          // 禁用访客
  "disable_3pid_login": false,      // 禁用邮箱/手机登录
  "sso_redirect_options": {         // SSO 自动跳转策略
    "immediate": false,
    "on_welcome_page": false,
    "on_login_page": false
  },
  "logout_redirect_url": null,      // 登出重定向
  "branding": {
    "auth_header_logo_url": "",     // Logo
    "welcome_background_url": "",   // 背景图
    "auth_footer_links": []         // 底部链接
  },
  "oidc_static_clients": {},        // 静态 OIDC 客户端 ID
  "embedded_pages": {
    "welcome_url": "",              // 自定义欢迎页
    "login_for_welcome": false       // 直接跳转登录
  }
}
```

---

## 3. 主界面布局

### 3.1 LoggedInView 整体布局

```
┌──────────┬───────────────────────┬──────────────────┐
│ 空间面板  │      主内容区          │    右侧面板       │
│          │                       │   （可收起）      │
│ Space    │  RoomView / HomePage  │   RightPanel     │
│ Panel    │  / UserView           │                  │
│          │                       │                  │
│          │  ┌─────────────────┐  │                  │
│          │  │ 消息时间线       │  │                  │
│          │  │                 │  │                  │
│          │  │                 │  │                  │
│          │  └─────────────────┘  │                  │
│          │  ┌─────────────────┐  │                  │
│          │  │ 消息输入框       │  │                  │
│          │  └─────────────────┘  │                  │
│          │                       │                  │
└──────────┴───────────────────────┴──────────────────┘
```

**左侧面板** 最小宽度 224px，可通过拖拽调整。
**右侧面板** 默认宽度 320px，最小 320px，最大 50% 窗口，左侧边缘可拖拽调整。

### 3.2 主界面组件层次

```
LoggedInView
├── SpacePanel                  // 空间导航栏（最左）
├── LeftPanel（可拖拽调整）
│   ├── RoomSearch              // 搜索/筛选框
│   ├── Breadcrumbs             // 最近访问面包屑
│   └── RoomListPanel           // 房间列表
│       ├── RoomSublist (Invites)
│       ├── RoomSublist (Favourites)
│       ├── RoomSublist (People/DM)
│       ├── RoomSublist (Rooms)
│       └── RoomSublist (LowPriority)
├── MainSplit（可拖拽调整）
│   ├── 主内容区
│   │   ├── RoomView            // 房间视图
│   │   ├── HomePage            // 首页
│   │   └── UserView            // 用户视图
│   └── RightPanel（可收起）
│       ├── RoomSummaryCard     // 房间摘要
│       ├── MemberList          // 成员列表
│       ├── ThreadPanel         // 线程列表
│       ├── FilePanel           // 文件列表
│       └── ...
├── ToastContainer              // 紧急通知
├── NonUrgentToastContainer     // 非紧急通知
├── UploadBar                   // 上传进度
└── PipContainer                // 画中画（通话/Widget）
```

### 3.3 快捷键（主界面）

| 快捷键 | 功能 |
|--------|------|
| Ctrl/Cmd + K | 全局搜索/筛选房间 |
| Ctrl/Cmd + ` | 切换用户菜单 |
| Ctrl/Cmd + Shift + D | 切换空间面板 |
| Ctrl/Cmd + , | 打开用户设置 |
| Ctrl/Cmd + / | 显示快捷键列表 |
| Alt + ↑↓ | 切换房间 |
| Shift + Alt + ↓ | 下一未读房间 |
| Ctrl/Cmd + [ / ] | 前进/后退导航 |
| Ctrl/Cmd + F6 / Shift+F6 | 地标导航（面板间跳转） |

---

## 4. 聊天消息

### 4.1 房间视图布局（RoomView）

```
┌─────────────────────────────────┐
│ RoomHeader                      │  ← 房间名、头像、通话按钮
├─────────────────────────────────┤
│ PinnedMessageBanner             │  ← 置顶消息横幅
├─────────────────────────────────┤
│ TopUnreadMessagesBar            │  ← 跳转到首条未读
│ JumpToBottomButton              │  ← 跳转到底部按钮
│                                 │
│ TimelinePanel（消息时间线）        │
│ ├── 日期分隔符                     │
│ ├── EventTile (消息1)            │
│ ├── EventTile (消息2 续)         │  ← 同作者5分钟内连发
│ ├── 阅读标记线                    │
│ └── ...                          │
│                                 │
│ WhoIsTypingTile                 │  ← 正在输入提示
├─────────────────────────────────┤
│ StatusBar / UploadBar           │  ← 上传进度
├─────────────────────────────────┤
│ MessageComposer                 │  ← 消息输入框
│ ├── ReplyPreview (回复时)        │
│ ├── VoiceRecordComposerTile      │  ← 语音录制
│ ├── 文本输入区 / 富文本编辑器       │
│ ├── Emoji 按钮                   │
│ ├── 附件/表情/投票/贴纸按钮         │
│ └── 发送按钮                     │
└─────────────────────────────────┘
```

### 4.2 消息类型总览

| 消息类型 | msgtype / type | 渲染组件 |
|----------|---------------|---------|
| 普通文本 | `m.text` | `TextualBodyFactory` — 支持 Markdown、@提及、代码块、剧透 |
| 通知消息 | `m.notice` | `TextualBodyFactory` — 服务器通知样式 |
| 动作消息 | `m.emote` | `TextualBodyFactory` — `/me` 消息 |
| 图片 | `m.image` | `ImageBodyFactory` — 缩略图、模糊占位、灯箱预览 |
| 文件 | `m.file` | `FileBodyFactory` — 下载链接/iframe预览、加密状态 |
| 音频 | `m.audio` | `MAudioBody` — 音频播放器 |
| 视频 | `m.video` | `VideoBodyFactory` — 视频播放器 + 缩略图 |
| 贴纸 | `m.sticker` (event) | `MStickerBody` — 动漫贴纸 |
| 投票 | `m.poll.start` | `MPollBody` — 交互式投票 |
| 位置 | `m.location` | `MLocationBody` — MapLibre 地图 |
| 实时位置 | `m.beacon_info` | `MBeaconBody` — 实时位置共享 |
| 语音消息 | `m.audio` (带 voice flag) | `MVoiceMessageBody` — 波形播放 |
| 通话事件 | `m.call.*` / `m.room.rtc_notification` | `CallEvent` / `LegacyCallEvent` |
| 加密验证 | `m.key.verification.request` | `MKeyVerificationRequestView` |
| 已撤回 | Redacted | `RedactedBodyFactory` — "消息已删除" |
| 解密失败 | 无法解密 | `DecryptionFailureBodyFactory` — 错误提示 |

### 4.3 事件卡片（EventTile）

每条消息渲染为一个 EventTile，结构如下：

```
EventTile
├── 头像 + 发送者名称（非连续消息时显示）
├── 时间戳
├── 消息正文（MBody）
├── 已读回执（接收者头像）
└── 脚注
    ├── 加密锁图标
    ├── 操作按钮（悬停显示：回应、回复、编辑、更多）
    ├── 回复预览
    ├── 线程信息
    └── 表情回应
```

**消息连续性**：同一发送者 5 分钟内的连续消息会合并显示（隐藏头像和发送者名）。

**已读回执**：
- 每个已读的用户头像显示在消息末尾
- 头像支持动画过渡
- 回执发送频率：500ms 防抖

**阅读标记线**：视觉上标识最后阅读位置，与 TopUnreadMessagesBar 联动。

### 4.4 消息输入框（MessageComposer）

支持两种模式：

| 模式 | 说明 |
|------|------|
| **纯文本模式** | 默认模式，支持 Markdown 语法 |
| **富文本模式** | WYSIWYG 所见即所得，通过 `feature_wysiwyg_composer` 实验功能开启 |

**输入框功能**：
- 发送消息（Enter / Ctrl+Enter 可选）
- @提及用户和房间（自动补全）
- 斜杠命令（`/join`, `/me`, `/topic` 等 40+ 条）
- 回复消息（显示引用预览）
- 编辑器状态持久化（localStorage 缓存）
- 发送历史（↑↓ 键切换）

**附件按钮**：
- 文件上传
- 图片/视频上传
- 贴纸选择
- 投票创建
- 位置分享
- 语音消息录制

### 4.5 置顶消息（Pinned Messages）

- 时间线上方显示横幅，展示最后一条置顶消息
- 多条置顶消息时，点击横幅可循环切换
- 显示计数（如 "2 / 5"）和位置指示点
- "查看全部" 按钮打开右侧面板的置顶消息列表
- 有权限的用户可取消置顶

### 4.6 拖拽上传

- 文件拖拽到聊天区域显示上传覆盖层
- 支持多文件同时上传
- 图片自动生成缩略图
- 加密房间自动加密附件
- 上传进度通过 UploadBar 显示

---

## 5. 房间管理

### 5.1 房间列表

#### 标签分类（Tag System）

房间按以下分类排序显示：

| 排序 | 分类 | 说明 |
|------|------|------|
| 1 | 邀请 | 待处理的房间邀请 |
| 2 | 收藏 | 用户标记为收藏的房间 |
| 3 | 私聊 | 私聊 / 直接消息 |
| 4 | 普通房间 | 未分类的房间 |
| 5 | 低优先级 | 标记为低优先级的房间 |
| 6 | 服务器通知 | 服务器公告 |
| 7 | 历史 | 存档/已遗忘房间 |

每个分类是一个可折叠、可拖拽调整高度的 `RoomSublist`。

#### 排序算法

- **ImportanceAlgorithm（默认）**：按通知优先级排序
  1. 未发送消息
  2. 高亮（@提及/@room）
  3. 通知（未读消息）
  4. 活跃（粗体）
  5. 无（已读/空闲）
  6. 已静音（底部）

- **RecentAlgorithm**：按最近活动时间排序
- **AlphabeticAlgorithm**：按字母顺序排序

#### 筛选

- 搜索框即时筛选房间
- **主筛选**：所有频道 / 私聊 / 全部

### 5.2 创建房间

**CreateRoomDialog** 功能：

| 选项 | 说明 |
|------|------|
| 房间名称 | 必填，异步校验 |
| 房间类型 | 普通房间 / 视频房间 |
| 可见性 | 仅邀请 / 空间成员 / 公开 / 敲门加入 |
| 加密 | 端到端加密开关（可被服务器策略强制） |
| 公开别名 | 公开房间必须设置 |
| 话题 | 可选描述 |
| 高级设置 | 联邦开关 |
| 父空间 | 在空间内创建时，自动关联 |

**公开房间**需要 `AllowCreatingPublicRooms` 设置开启。

### 5.3 加入房间

| 入口 | 说明 |
|------|------|
| 房间别名 | `#roomname:server` |
| 房间 ID | `!roomid:server` |
| Matrix.to 链接 | 含 via 参数 |
| 空间层级 | 浏览空间的子房间 |
| 公开目录 | 浏览/搜索公开房间列表 |
| 邀请 | 接受/拒绝/拒绝并拉黑 |
| 敲门 | 请求加入，管理员审核 |

### 5.4 房间预览（未加入时）

`RoomPreviewBar` 根据不同状态显示：

| 状态 | 显示 |
|------|------|
| 受邀 | 接收/拒绝/拒绝并拉黑（显示邀请原因） |
| 公开房间预览 | 加入按钮（含提示文字） |
| 被踢出 | 忘记房间 / 重新加入 |
| 被禁止 | 忘记房间 |
| 敲门中 | 原因输入框 + 发送请求 |
| 请求被拒 | 忘记房间 |
| 房间不存在 | 错误提示 |
| 加载中 | 旋转加载器 |

### 5.5 房间设置（RoomSettingsDialog）

| 标签 | 功能 |
|------|------|
| **常规** | 房间名称、话题、头像、别名管理、离开房间 |
| **人员** | 敲门请求审核（需 feature_ask_to_join） |
| **语音视频** | 通话设置（需 feature_group_calls） |
| **安全与隐私** | 加入规则、访客访问、历史可见性、加密开关、未验证设备拦截 |
| **角色与权限** | 各类事件的权限级别、添加管理员/版主、封禁用户 |
| **通知** | 铃声设置、通知级别（全部/仅提及/关闭） |
| **桥接** | 桥接状态（需 feature_bridge_state） |
| **投票历史** | 所有投票列表 |
| **高级** | 房间升级、前身房间链接 |

### 5.6 房间操作

| 操作 | 说明 |
|------|------|
| 收藏/取消收藏 | 点击星标切换 |
| 邀请成员 | 支持 MXID、邮箱、ID Server 查询 |
| 离开房间 | 含警告（唯一管理员、最后一人等） |
| 忘记房间 | 删除本地记录 |
| 复制链接 | 复制 Matrix.to 链接 |
| 分享房间 | 在空间内分享 |
| 导出聊天 | 导出消息历史 |
| 举报房间 | 举报违规内容 |

### 5.7 房间升级

- 检测房间协议版本是否需要升级
- 创建新版本房间，设置 `m.room.tombstone` 墓碑事件
- 自动追踪升级链，离开时处理所有版本

---

## 6. 空间（Spaces/社区）

### 6.1 空间概念

Space 是一种特殊的房间（`RoomType.Space`），用于组织和管理多个房间及子空间，形成层级结构。类似"群组"或"社区"概念。

### 6.2 内建元空间（Meta Spaces）

| 元空间 | 说明 |
|--------|------|
| 首页 | 显示所有房间 或 仅已加入房间（可配置） |
| 收藏 | 仅显示标记为收藏的房间 |
| 私聊 | 仅显示私聊/直接消息 |
| 未分类 | 未分配到任何空间的孤儿房间 |
| 视频房间 | 视频通话房间 |

### 6.3 空间面板

- 最左侧导航栏，可折叠为图标模式或展开显示名称
- 显示通知徽章
- 空间可拖拽排序
- 子空间支持折叠/展开
- 右键/长按弹出上下文菜单

### 6.4 创建空间

两步流程：
1. **选择可见性**：公开或私有（含说明文字和图标）
2. **填写信息**：名称（自动生成别名）、别名（公开空间）、描述、头像

空间创建时的特殊设置：
- 默认事件发送权限 100（admin only，防止刷屏）
- 公开空间历史可见性设为 `WorldReadable`
- 支持 MSC3827（公开空间目录）

### 6.5 空间过滤

- 选择空间后，房间列表只显示该空间及其子空间中的房间
- SpaceStore 维护父子空间映射
- 递归展平空间层级获取所有房间

---

## 7. 右侧面板

### 7.1 面板状态机

右侧面板基于**卡片历史栈**的状态机：

```
[RoomSummary] → [MemberList] → [MemberInfo]
                              → [EncryptionPanel]
[RoomSummary] → [ThreadPanel] → [ThreadView]
[RoomSummary] → [PinnedMessages]
[RoomSummary] → [FilePanel]
[RoomSummary] → [NotificationPanel]
[RoomSummary] → [Extensions] → [Widget]
```

每个卡片有返回按钮，点击返回上一层。

### 7.2 所有面板卡片

| 面板 | 说明 |
|------|------|
| **RoomSummaryCard** | 房间信息：头像、名称、别名、话题、加密状态徽章、操作菜单 |
| **MemberList** | 虚拟滚动成员列表，含搜索和在线状态 |
| **MemberInfo** | 用户资料 + 权力级别管理 |
| **EncryptionPanel** | 加密验证面板（QR 码或 SAS 表情验证） |
| **ThreadPanel** | 线程列表（"我的"/"全部"筛选 + 全部已读） |
| **ThreadView** | 单个线程对话（含消息输入框） |
| **PinnedMessages** | 置顶消息列表（含取消置顶控件） |
| **FilePanel** | 文件/图片/视频消息过滤列表 |
| **NotificationPanel** | 全局通知时间线 |
| **Widget** | 嵌入的小部件（iframe）|
| **Extensions** | 小部件列表（含图钉/最大化控件） |
| **Timeline** | 完整时间线（视频房间聊天用） |

### 7.3 RoomSummaryCard 操作菜单

- 切换收藏
- 邀请成员
- 查看成员列表
- 查看线程
- 查看置顶消息（含计数徽标）
- 查看文件
- 查看扩展（小部件）
- 复制链接
- 投票历史
- 导出聊天
- 房间设置
- 举报房间
- 离开房间

---

## 8. 语音与视频通话

### 8.1 双轨通话架构

Element 维护两套通话系统：

| 系统 | 适用场景 | 实现方式 |
|------|---------|---------|
| **Legacy 1:1 通话** | 两人一对一 Matrix 原生 WebRTC | LegacyCallHandler + 原生 WebRTC |
| **Group 群组通话** | 多人通话 | 基于 Widget 实现（Element Call 或 Jitsi） |

### 8.2 Legacy 1:1 通话

**LegacyCallHandler** 功能：

- 支持最多 2 个并发通话（其中一个为活跃，其余保持）
- 来电铃声播放（ring.mp3/ogg）
- 铃声/回铃/忙音/挂断音效
- 静音/取消静音
- 通话转移（转到其他 Matrix 用户或电话号码）
- 画廊侧边栏（多个视频源时）
- PSTN 拨号（拨号盘）

**通话视图**：
- 视频/音频源显示（`VideoFeed`/`AudioFeed`）
- 控制按钮：麦克风开关、摄像头开关、屏幕共享、侧边栏、拨号盘、更多、挂断
- 通话时长显示
- 全屏模式支持
- 画中画模式（离开房间时）
- 通话状态事件分组（振铃/已接通/已结束等）

### 8.3 Group 群组通话

**Element Call**（基于 MatrixRTC MSC4143）：
- 虚拟 Widget 内嵌 Element Call 应用
- 通过 MatrixRTC Session 追踪参与者
- 支持 LiveKit SFU
- 参与者加入/离开状态

**Jitsi**：
- Jitsi Meet 作为 iframe Widget 嵌入
- 域名优先级：.well-known > SdkConfig > meet.element.io
- 支持 1:1 通话和群组通话
- 可配置 docked/undocked 布局

### 8.4 来电通知

- **弹窗提示**（IncomingCallToast）：显示来电房间、参与者头像、"加入"和"拒绝"按钮
- 视频通话可选择"仅音频加入"
- 铃声播放（最长 90 秒）
- 自动过期（通话结束、被拒绝、被撤回、其他设备接听）

### 8.5 画中画（PiP）

- 离开通话房间时自动进入 PiP 模式
- 可拖拽窗口（默认 336×232px）
- 松开后自动吸附到最近角落
- 平滑动画过渡

### 8.6 通话设置

| 设置 | 说明 |
|------|------|
| 音频输入设备 | 麦克风选择 |
| 音频输出设备 | 扬声器选择 |
| 视频输入设备 | 摄像头选择 |
| 自动增益控制 | 自动调节麦克风音量 |
| 回声消除 | AEC 回声消除 |
| 噪音抑制 | 背景噪音过滤 |
| WebRTC P2P | 强制 P2P 连接或允许中继 |
| 镜像本地视频 | 水平翻转本地画面 |

---

## 9. 语音消息

### 9.1 录制流程

```
点击录制按钮 → 开始录制
  ├── 实时波形显示（LiveRecordingWaveform）
  ├── 计时器（LiveRecordingClock，红色闪烁指示灯）
  └── 最长 15 分钟（10 秒前 EndingSoon 警告）
→ 停止录制
  ├── 预览播放（RecordingPlayback）
  │   ├── 播放/暂停按钮
  │   ├── 波形 + 进度条
  │   └── 时间显示
  ├── 删除按钮
  └── 发送按钮
→ 上传 → 发送消息内容（含持续时间、文件大小、加密信息、缩略波形）
```

### 9.2 录音技术

| 参数 | 设置 |
|------|------|
| 采样率 | 48kHz（WebRTC 标准） |
| 声道 | 单声道 |
| 编码 | Opus（ogg 容器） |
| 语音质量 | 24kbps Opus (voice app) |
| 高音质 | 96kbps Opus (full band) |
| 降噪 | 启用时使用语音模式 |
| 引擎 | AudioWorklet（主力）/ ScriptProcessorNode（Safari 降级） |

### 9.3 播放功能

- 波形播放器（39 个采样条）
- 进度条拖拽定位
- 自动连续播放（PlaybackQueue：播放完当前后自动播放下一条未播放的语音消息）
- 全局播放管理（PlaybackManager：只允许一个播放实例）
- 5 秒快进/快退（← → 方向键）
- 空格键播放/暂停

---

## 10. 通知系统

### 10.1 通知层级

```
Unsent (未发送, 5) > Highlight (高亮 @提及, 4) > Notification (通知, 3) > Activity (活跃, 2) > None (无, 1) > Muted (静音, 0)
```

### 10.2 桌面通知

| 功能 | 说明 |
|------|------|
| 弹窗通知 | 使用浏览器 Notification API（Electron 使用系统通知） |
| 通知内容 | 发送者 + 消息摘要（含剧透处理） |
| 头像 | 发送者头像 |
| 通知音 | 可自定义铃声（MXC 文件） |
| 通知抑制 | 正在查看该房间/线程时自动抑制 |
| 加密消息 | 等待解密后再显示通知 |
| 已读后关闭 | 房间被阅读后自动关闭通知 |

### 10.3 通知设置

**会话级设置**：
- 启用桌面通知（需浏览器权限）
- 通知消息体预览
- 音频通知

**全局通知模式**：
- 所有消息
- 仅提及和关键词
- 关闭

**声音设置**：
- 提及/关键词铃声
- 私聊铃声
- 通话铃声

**其他活动**：
- 邀请通知
- 房间活跃通知
- 机器人通知

**提及与关键词**：
- @room 通知
- @用户通知
- 自定义关键词
- 粗体显示（仅提及）

### 10.4 推送通知

- 支持邮箱推送（Pusher）
- 每个验证过的邮箱可独立开关
- 支持其他推送目标（移动设备等）

### 10.5 房间通知徽章

| 类型 | 显示 |
|------|------|
| 无 | 不显示 |
| 粗体（活动） | 灰色圆点 |
| 未读消息 | 灰色数字徽章 |
| 高亮（提及） | 红色数字徽章 |
| 敲门请求 | 敲门图标 |
| 未发送 | "!" 符号 |

### 10.6 Toast 系统

- **ToastStore**：紧急/半紧急通知，按优先级排序
- **NonUrgentToastStore**：非紧急通知
- 桌面通知启用提示 Toast（priority 30）
- 来电 Toast（priority 100）

---

## 11. 搜索功能

### 11.1 全局搜索（SpotlightDialog）

通过 **Ctrl/Cmd + K** 激活，搜索来源：

| 来源 | 说明 |
|------|------|
| 本地房间 | 所有已加入/受邀房间 |
| 房间成员 | 所有已加入房间的成员 |
| 用户目录 | 服务器端用户搜索 |
| 公开房间 | 公开房间目录搜索 |
| 公开空间 | 空间目录搜索（MSC3827） |
| 历史记录 | 最近 10 条搜索记录 |
| 最近访问 | Breadcrumbs 面包屑 |

**结果分类**：人员 / 房间 / 空间 / 建议 / 公开结果

**功能**：
- 防抖搜索
- 搜索历史持久化
- 键盘导航（Roving Tab Index 无障碍模式）
- 未读通知徽章
- 复制邀请链接
- 群聊邀请
- 分析埋点

### 11.2 房间内消息搜索

**双源搜索架构**：

| 搜索源 | 适用场景 |
|--------|---------|
| 服务端搜索 | 非加密房间，使用 Matrix /search API |
| 本地索引搜索 | 加密房间，使用本地事件索引（Seshat/EventIndex） |

**合并搜索（Combined）**：
- 采用滑动窗口算法合并两个来源的结果
- 支持分页加载更多
- 加密消息解密后重新搜索

**搜索结果显示**：
- 匹配词高亮
- 上下文消息展示
- 跨房间搜索时显示房间名头部
- 线程感知（正确处理线程关系）

### 11.3 事件索引

- 平台相关：桌面端使用 Seshat（native 模块），Web 端使用 IndexedDB
- 仅索引加密房间（非加密房间依赖服务器搜索）
- 实时索引新事件 + 后台爬取历史事件
- 支持文件事件查询（FilePanel 使用）
- 支持撤回事件处理
- 检查点机制支持增量索引

---

## 12. 设置与偏好

### 12.1 设置层级（优先级从高到低）

| 层级 | 存储位置 | 说明 |
|------|---------|------|
| DEVICE | localStorage | 设备级设置 |
| ROOM_DEVICE | localStorage | 设备级、按房间 |
| ROOM_ACCOUNT | 房间账户数据 | 账户级、按房间 |
| ACCOUNT | 账户数据 | 服务端账户数据 |
| ROOM | 房间状态事件 | 管理员设置 |
| PLATFORM | Electron 存储 | 桌面端平台设置 |
| CONFIG | config.json | 部署配置（只读） |
| DEFAULT | 硬编码 | 默认值 |

### 12.2 用户设置标签

| 标签 | 设置内容 |
|------|---------|
| **账户** | 邮件、手机、密码管理、显示名、头像、停用账户 |
| **会话管理** | 设备列表、会话管理、QR 登录 |
| **外观** | 主题、布局（IRC/Group/Bubble）、字体缩放、图片大小、系统字体 |
| **通知** | 桌面通知、音频、消息预览、通知级别、提及关键词 |
| **偏好** | 语言、拼写检查、时间线偏好、时间格式、表情、代码块、正在输入、输入框 |
| **键盘** | 键盘快捷键一览（按分类组织） |
| **侧边栏** | 元空间开关、首页包含所有房间 |
| **语音视频** | 设备选择、音频处理、P2P 开关、（条件显示） |
| **安全隐私** | 密钥备份、忽略列表、分析、集成管理、身份服务器、消息搜索、脱水设备 |
| **加密** | 交叉签名管理、恢复设置 |
| **实验室** | Beta 功能和实验性功能开关（条件显示） |
| **帮助** | 版本信息、更新检查、清除缓存、Bug 报告、法律信息 |

### 12.3 外观设置详情

| 设置 | 选项 |
|------|------|
| **主题** | Light / Light High Contrast / Dark / 自定义主题 |
| **匹配系统主题** | 自动跟随 OS 暗色/亮色模式 |
| **布局** | IRC（现代） / Group（传统群组） / Bubble（气泡） |
| **字体缩放** | 相对于浏览器默认字号的增量 |
| **图片大小** | Normal / Large |
| **自定义主题** | 自定义 CSS 变量颜色、字体、Compound design tokens |

### 12.4 偏好设置详情

| 设置 | 说明 |
|------|------|
| 显示已读回执 | 消息末尾显示已读用户头像 |
| 显示加入/离开 | 显示成员加入/离开通知 |
| 显示头像变更 | 显示成员头像变更 |
| 显示显示名称变更 | 显示成员名称变更 |
| 显示撤回消息 | 显示已撤回消息的占位 |
| 时间格式 | 12 小时 / 24 小时制 |
| 表情自动补全 | 输入 `:` 触发 |
| 大表情 | 单独发送时放大显示 |
| 替换纯文本表情 | 将 `:)` 等转为 emoji |
| 代码行号 | 代码块显示行号 |
| 代码默认展开 | 代码块默认展开 |
| 代码语法高亮 | 代码块语法高亮 |
| 发送正在输入通知 | 向他人显示正在输入 |
| 显示正在输入通知 | 查看他人的正在输入 |
| Ctrl+Enter 发送 | 使用 Ctrl+Enter 发送消息 |
| 消息预览 | 房间列表显示最新消息预览 |
| 聊天效果 | 启用/禁用 `/fireworks` 等效果 |
| 邀请规则 | 配置谁可以邀请你 |
| 面包屑 | 显示最近访问房间 |
| 时区 | 发布时区到个人资料 |

### 12.5 实验性功能（Labs）

按 LabGroup 分组：

| 分组 | 示例功能 |
|------|---------|
| 消息 | 富文本编辑器、LaTeX 数学 |
| 空间 | 视频房间、动态房间前任 |
| 线程 | 线程通知、线程活动中心 |
| 语音视频 | Element Call 视频房间 |
| 管理 | Mjolnir 反滥用 |
| 主题 | 自定义主题 |
| 加密 | 免信任设备脱水 |
| 开发 | 开发者模式 |
| UI | 新房间列表、简化滑动同步 |

---

## 13. Widget 小部件与集成

### 13.1 Widget 类型

| 类型 | 说明 |
|------|------|
| Jitsi | 视频会议 |
| StickerPicker | 贴纸选择器 |
| IntegrationManager | 集成管理器（Scalar 兼容）|
| Custom | 自定义小部件 |
| Call | 通话小部件 |

### 13.2 Widget 管理

- **WidgetStore**：管理所有房间的 Widget 映射
- **ActiveWidgetStore**：追踪当前活跃的 Widget（持久化/停靠）
- **WidgetEchoStore**：乐观更新（本地立即显示，服务端确认后同步）
- **WidgetLayoutStore**：Widget 布局管理（顶部/中间/右侧容器）

**AppsDrawer**：
- 顶部容器最多 3 个固定 Widget
- 支持垂直拖拽调整高度
- 支持水平分配宽度
- 最大化的 Widget 占据中间区域

### 13.3 集成管理器

- 从三个来源发现集成管理器：配置 / 账户数据 / .well-known
- 使用 `ScalarAuthClient` 进行 OpenID Token 交换认证
- 通过 `ScalarMessaging` 的 postMessage 桥接通信
- 支持 Widget 创建/更新/删除、成员管理、机器人管理

### 13.4 Widget 权限

- 基于 URL 模式的自动授权
- 能力声明（AlwaysOnScreen、StickerSending 等）
- 身份 Token 自动批准
- 预加载自动批准

---

## 14. 桌面应用（Electron）

### 14.1 核心特性

| 特性 | 说明 |
|------|------|
| 系统托盘 | Windows/Linux 系统托盘图标 + 上下文菜单 |
| 自启动 | 支持启用/最小化/禁用三种状态 |
| 自动更新 | macOS/Windows Squirrel 自动更新 |
| 原生通知 | 点击通知聚焦窗口 |
| 任务栏徽章 | 未读计数覆盖图标 |
| 窗口闪烁 | 紧急通知时任务栏闪烁 |
| 菜单栏 | 完整应用菜单（可隐藏） |
| 安全存储 | 使用 safeStorage 加密 pickle key |
| 拼写检查 | 多语言拼写检查 + 自定义词典 |
| 屏幕共享 | 系统级屏幕/窗口选择器 |
| 深度链接 | 自定义 URI scheme（`element://`） |
| 内容保护 | Windows 阻止截屏 |
| 硬件加速 | 可开关的 GPU 加速 |
| macOS 标题栏 | 原生红绿灯按钮 + 拖拽区域 |

### 14.2 桌面设置

| 设置 | 类型 | 说明 |
|------|------|------|
| 自启动 | enabled/minimised/disabled | 开机自启 |
| 退出前确认 | boolean | 关闭前弹窗确认 |
| 始终显示菜单栏 | boolean | 非 macOS 平台 |
| 显示托盘图标 | boolean | 非 macOS 平台 |
| 硬件加速 | boolean | 需要重启生效 |
| 内容保护 | boolean | Windows 阻止截屏 |

### 14.3 多账户支持

- `--profile` CLI 参数指定账户
- `--profile-dir` 指定配置目录
- 环境变量 `ELEMENT_PROFILE_DIR`
- 单实例锁（每个账户一个实例）
- SSO 回调自动路由到对应账户窗口

### 14.4 PWA 特性

- Service Worker 支持（MSC3916 认证媒体 URL 重写）
- App Badge API（任务栏未读徽章）
- Web Platform 的 Pickle Key IndexedDB 存储

---

## 15. 斜杠命令

### 15.1 消息类命令

| 命令 | 说明 |
|------|------|
| `/spoiler <msg>` | 发送剧透消息 |
| `/shrug <msg>` | 追加 `¯\_(ツ)_/¯` |
| `/tableflip <msg>` | 追加 `(╯°□°）╯︵ ┻━┻` |
| `/unflip <msg>` | 追加 `┬──┬ ノ( ゜-゜ノ)` |
| `/lenny <msg>` | 追加 `( ͡° ͜ʖ ͡°)` |
| `/plain <msg>` | 发送纯文本 |
| `/html <msg>` | 发送 HTML |
| `/rainbow <msg>` | 彩虹色文本 |
| `/rainbowme <msg>` | 彩虹色 /me |
| `/me <msg>` | 动作消息 |

### 15.2 操作类命令

| 命令 | 说明 |
|------|------|
| `/join <room>` | 加入房间（别名/ID/permalink） |
| `/goto <room>` | 查看房间（不加入） |
| `/invite <user> [reason]` | 邀请用户 |
| `/part [room]` | 离开房间 |
| `/nick <name>` | 设置全局显示名称 |
| `/myroomnick <name>` | 设置房间显示名称 |
| `/msg <user> [msg]` | 发送私聊 |
| `/query <user>` | 打开私聊 |
| `/ignore <user>` | 忽略用户 |
| `/unignore <user>` | 取消忽略 |
| `/status <emoji> <text>` | 设置状态（MSC4426） |

### 15.3 管理类命令

| 命令 | 说明 |
|------|------|
| `/topic [topic]` | 获取/设置话题 |
| `/roomname <name>` | 设置房间名称 |
| `/roomavatar [mxc]` | 设置房间头像 |
| `/myroomavatar [mxc]` | 设置房间内头像 |
| `/myavatar [mxc]` | 设置全局头像 |
| `/op <user> [level]` | 提升权限（默认 50） |
| `/deop <user>` | 重置权限 |
| `/remove <user> [reason]` | 踢出用户 |
| `/ban <user> [reason]` | 封禁用户 |
| `/unban <user>` | 解封用户 |
| `/addwidget <url>` | 添加 Widget |
| `/upgraderoom <version>` | 升级房间 |
| `/converttodm` | 转为私聊 |
| `/converttoroom` | 转为普通房间 |

### 15.4 高级命令

| 命令 | 说明 |
|------|------|
| `/devtools` | 打开开发工具 |
| `/rageshake <desc>` | 提交 Bug 报告 |
| `/whois <user>` | 查看用户信息 |
| `/jumptodate <date>` | 跳转到指定日期 |
| `/verify <device> <fp>` | 手动验证设备指纹 |
| `/discardsession` | 强制丢弃 Olm 会话 |
| `/holdcall` / `/unholdcall` | 保持/恢复通话 |
| `/help` | 显示帮助 |

### 15.5 特效命令

`/fireworks`, `/snowfall`, `/rain`, `/spaceinvaders`, `/hearts`, `/confetti` 等聊天特效。

---

## 16. 线程（Threads）

### 16.1 线程交互

- 对消息使用"在话题中回复"创建新线程
- 线程消息在主时间线中显示为线程摘要（回复计数 + 最后回复者）
- 点击线程摘要可在右侧面板打开完整线程

### 16.2 线程面板（ThreadPanel）

- 显示房间中所有线程的列表
- 筛选："所有" / "我的"（我参与过的）
- "全部已读"按钮
- 每条显示最新回复预览

### 16.3 线程视图（ThreadView）

- 完整消息时间线（仅线程内消息）
- 独立的消息输入框
- 文件上传支持
- 线程内消息关系追踪（`m.thread` 关系类型）

---

## 17. 文件处理

### 17.1 上传流程

```
选择文件 / 拖拽
  → 文件大小校验（检查服务器上传限制）
  → 上传确认对话框（可批量确认）
  → 启动上传：
      ├── 图片：加载、生成缩略图、检测长图/动图
      ├── 音频：加载、提取时长
      ├── 视频：加载、生成缩略海报、提取时长
      └── 通用文件：直接上传
  → 加密房间：使用 attachment 加密，上传为 application/octet-stream
  → UploadBar 显示进度（可取消）
  → 发送消息（含 msgtype、元数据、缩略图、文件大小）
```

### 17.2 支持的媒体类型

| 类型 | msgtype | 特殊处理 |
|------|---------|---------|
| 图片 | `m.image` | 缩略图 + 灯箱预览 |
| 音频 | `m.audio` | 音频播放器 + 时长 |
| 视频 | `m.video` | 视频播放器 + 缩略海报 |
| 文件 | `m.file` | 下载链接 + 文件图标 |

### 17.3 缩略图优化

- 小于 32KB 的图片跳过缩略图
- 缩略图未显著减小时（>1MB 且缩减 <10%）跳过
- 始终为 AVIF/WebP/SVG 生成缩略图（兼容性）
- 按 `devicePixelRatio` 缩放（Retina 显示）

### 17.4 下载

- 媒体文件通过 `Media` 类管理，提供 HTTP URL
- 加密文件自动解密
- 支持 Electron 下载管理器（含下载完成 Toast）

---

## 18. Markdown 与富文本

### 18.1 支持的 Markdown 语法

- 标题（# ## ### 等）
- 粗体/斜体/删除线
- 引用块
- 代码块（语法高亮）
- 行内代码
- 链接
- 无序/有序列表
- 表格
- 水平线

### 18.2 特殊渲染

| 功能 | 实现 |
|------|------|
| @提及（User Pill） | 用户头像 + 名称，可点击跳转 |
| @room 提及 | 特殊样式标签 |
| 关键词高亮 | 通知关键词渲染为 Pill |
| 剧透内容 | `<span data-mx-spoiler>` → 点击显示 |
| LaTeX 数学 | KaTeX 渲染（`<div data-mx-maths>` 和 `<span data-mx-maths>`） |
| 代码高亮 | highlight.js 语法高亮 |
| 链接提示 | URL 不等于显示文本时，悬停显示真实 URL |

### 18.3 富文本编辑器（WYSIWYG）

通过 `feature_wysiwyg_composer` 实验功能开启：
- 所见即所得编辑
- 格式工具栏（粗体、斜体、引用、代码、链接）
- 与纯文本模式可切换
- 编辑器状态按房间/线程持久化

---

## 19. 表情与回应

### 19.1 表情选择器

- 分类导航：Recent / People / Nature / Food / Activity / Places / Objects / Symbols / Flags
- 搜索过滤（按名称/字符/关键词）
- 虚拟滚动渲染
- 键盘导航（WAI-ARIA Grid 模式）
- 使用频率追踪（Recent 按使用次数排序，最多 100 个）

### 19.2 表情回应（Reactions）

- 每条消息可添加/移除表情回应
- 点击已添加的表情可取消（撤回回应事件）
- 快速回应栏（👍👎😄🎉😕❤️🚀👀）
- 显示回应计数和用户列表

### 19.3 快捷方式

- `:emoji_name:` 在编辑器中自动补全
- `+ :emoji:` 快捷发送回应

---

## 20. 无障碍

### 20.1 键盘快捷键系统

定义了 50+ 个键盘快捷键操作，分类如下：

| 分类 | 操作数 | 示例 |
|------|--------|------|
| 导航 | 15 | 切换面板、房间导航、空间切换 |
| 编辑器 | 17 | 发送、格式化、撤销、表情 |
| 房间 | 8 | 搜索、上传、滚动、跳转 |
| 房间列表 | 6 | 上下导航、选择、折叠 |
| 自动补全 | 5 | 导航选项、确认 |
| 通话 | 2 | 开关麦克风/摄像头 |
| 无障碍 | 14 | Esc、Enter、Space、方向键 |

平台适配：Mac 使用 ⌘⌥⇧ 符号，Windows 使用 Ctrl+Alt+Shift。

### 20.2 地标导航（Landmark）

通过 F6/Shift+F6 在主要 UI 区域间跳跃：
```
活动空间按钮 → 房间搜索 → 房间列表 → 消息输入框 / 首页
```

### 20.3 Roving Tab Index

- WAI-ARIA Roving Tabindex 模式
- 用于表情选择器、工具栏、菜单等
- 支持方向键导航和 Home/End

### 20.4 其他无障碍特性

- 通知徽章含 `aria-label`
- ARIA 角色标注（`role="toolbar"`, `role="group"`, `role="tabpanel"`）
- `aria-controls` 关联 UI 元素
- 焦点管理（发送消息后自动聚焦输入框）
- 屏幕阅读器友好的事件渲染

---

## 21. 定制化与扩展

### 21.1 内建定制化点（`src/customisations/`）

| 定制化点 | 说明 |
|----------|------|
| ComponentVisibility | 控制 UI 组件显隐（邀请、创建房间/空间、探索、集成等） |
| Media | 覆盖媒体对象创建（URL 解析、缩略图、加密） |
| Lifecycle | 登出/清除存储时的钩子 |
| RoomList | 过滤哪些房间可显示 |
| UserIdentifier | 自定义用户 ID 显示方式 |
| Alias | 别名显示偏好 |
| Director | 房间发布额外限制 |
| WidgetPermissions | 预批准 Widget 能力 |
| WidgetVariables | 提供自定义 Widget URL 变量 |
| ChatExport | 强制导出参数 |

### 21.2 Module API 扩展系统

外部模块通过 `@element-hq/element-web-module-api` 包提供的 API 接口集成：

| API 能力 | 说明 |
|----------|------|
| config | 读取配置值（按域名作用域） |
| i18n | 注册翻译字符串 |
| navigation | 导航到房间、Matrix.to 链接 |
| dialog | 打开模态对话框 |
| profile | 获取/监听用户资料 |
| customComponents | 注册自定义消息渲染器、房间预览栏、登录组件 |
| builtins | 渲染房间头像、房间视图、通知装饰 |
| stores | 访问 RoomListStore 等 |
| client | 账户数据读/写 |
| widgetLifecycle | 注册 Widget 预加载/身份/能力审批器 |
| widget | 获取/移动 Widget |
| customisations | 注册 UI 组件显示控制 |
| composer | 添加文件上传选项、插入文本 |
| extras | 添加空间面板项目、房间头按钮 |

### 21.3 内建扩展模块

| 模块 | 功能 |
|------|------|
| widget-lifecycle | 按 URL 模式自动批准 Widget 权限 |
| widget-toggles | 在房间头添加 Widget 开关按钮 |
| restricted-guests | 访客限制系统（替换预览栏、隐藏 UI 组件） |
| banner | 品牌导航横幅（含 Univention 动态菜单支持） |

---

## 附录

### A. 键盘快捷键速查

| 快捷键 | 功能 | 分类 |
|--------|------|------|
| `Ctrl/Cmd + K` | 全局搜索 | 导航 |
| `Ctrl/Cmd + ,` | 打开设置 | 导航 |
| `Ctrl/Cmd + /` | 快捷键帮助 | 导航 |
| `Ctrl/Cmd + `` ` | 用户菜单 | 导航 |
| `Ctrl/Cmd + Shift + D` | 切换空间面板 | 导航 |
| `Ctrl/Cmd + Shift + U` | 上传文件 | 房间 |
| `Enter` | 发送消息 | 编辑器 |
| `Shift + Enter` | 换行 | 编辑器 |
| `Ctrl/Cmd + B` | 粗体 | 编辑器 |
| `Ctrl/Cmd + I` | 斜体 | 编辑器 |
| `Ctrl/Cmd + Shift + X` | 删除线 | 编辑器 |
| `Escape` | 取消回复/编辑 | 编辑器 |
| `PageUp / PageDown` | 滚动时间线 | 房间 |
| `Alt + ↑↓` | 切换房间 | 房间列表 |
| `Ctrl/Cmd + D` | 切换麦克风 | 通话 |
| `Ctrl/Cmd + E` | 切换摄像头 | 通话 |
| `F6 / Shift+F6` | 地标导航 | 无障碍 |

### B. 主要路由表

| URL Hash | 视图 |
|----------|------|
| `#/welcome` | 欢迎页 |
| `#/login` | 登录 |
| `#/register` | 注册 |
| `#/forgot_password` | 忘记密码 |
| `#/soft_logout` | 软登出 |
| `#/home` | 首页 |
| `#/room/:roomId` | 房间视图 |
| `#/room/:roomId/:eventId` | 定位到特定消息 |
| `#/user/:userId` | 用户视图 |

### C. 技术依赖

| 依赖 | 用途 |
|------|------|
| matrix-js-sdk | Matrix 客户端核心 SDK |
| @vector-im/compound-web | Compound 设计系统 UI 组件 |
| @vector-im/matrix-wysiwyg | 富文本编辑器 |
| maplibre-gl | 地图渲染（位置消息） |
| katex | LaTeX 数学公式渲染 |
| opus-recorder | 语音消息录制 |
| matrix-widget-api | Widget 通信协议 |
| @sentry/browser | 错误监控 |
| posthog-js | 分析埋点 |
| oidc-client-ts | OIDC 认证 |

---

> 本文档基于 Element Web v1.12.21 源码分析整理，覆盖功能完整度约 95%。部分实验性功能和平台特定功能可能随版本更新而变化。
