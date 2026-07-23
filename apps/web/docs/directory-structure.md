# Element Web 目录结构说明

## 项目概述

Element 是基于 [Matrix](https://matrix.org) 协议的开源 Web 与桌面客户端，前身为 Vector/Riot。支持端到端加密（E2EE）私密通信、多平台运行（Web/Desktop/Android/iOS），并提供模块化扩展能力（Module API）。

---

## 一、`apps/web/` — 应用根目录

```
apps/web/
├── src/                        # 核心源码（详见第二部分）
├── res/                        # 静态资源
│   ├── css/                    # 全局样式（PostCSS）
│   ├── themes/                 # 主题包（dark/light/legacy 等）
│   ├── img/                    # 图片资源
│   ├── fonts/                  # 字体
│   ├── media/                  # 媒体文件
│   └── vector-icons/           # 图标字体
├── test/                       # 测试代码
│   ├── unit-tests/             # 单元测试
│   ├── app-tests/              # 应用级测试
│   └── test-utils/             # 测试工具
├── playwright/                 # Playwright E2E 测试
│   ├── e2e/                    # 端到端测试用例（按功能分子目录）
│   ├── pages/                  # 页面对象模型（Page Object）
│   └── plugins/                # Playwright 插件
├── webapp/                     # 构建产物（webpack 输出目录）
├── module_system/              # 模块系统（动态加载自定义模块）
│   ├── scripts/install.ts      # 模块安装脚本
│   ├── BuildConfig.ts          # 构建配置
│   └── installer.ts            # 模块安装器
├── element.io/                 # element.io 官方部署配置
├── scripts/                    # 构建 / 部署脚本
│   ├── package.sh              # 打包脚本
│   ├── ci_package.sh           # CI 打包脚本
│   └── deploy.py               # 部署脚本
├── docker/                     # Docker 部署配置
│   ├── docker-entrypoint.d/    # 容器入口脚本
│   └── nginx-templates/        # Nginx 配置模板
├── @types/                     # 全局 TypeScript 类型声明
├── __mocks__/                  # Jest Mock 文件
├── webpack.config.ts           # Webpack 构建配置
├── project.json                # Nx 任务定义（build、start、lint 等）
├── config.json                 # 运行时配置（homeserver、主题、特性开关等）
├── config.sample.json          # 配置模板
├── build_config.sample.yaml    # 构建配置模板
├── babel.config.cjs            # Babel 配置
├── jest.config.ts              # Jest 测试配置
├── vitest.config.ts            # Vitest 测试配置
├── playwright.config.ts        # Playwright 测试配置
├── tsconfig.json               # TypeScript 配置
└── package.json                # 依赖与 NPM 脚本
```

---

## 二、`apps/web/src/` — 核心源码

### 2.1 应用入口与生命周期

| 路径 | 说明 |
|------|------|
| `src/vector/` | 应用入口、平台初始化、Webpack entry |
| `src/Lifecycle.ts` | 应用生命周期管理（启动、登录、登出、重连等） |
| `src/BasePlatform.ts` | 平台抽象基类（Web / Electron 差异化处理） |
| `src/PlatformPeg.ts` | 平台实例单例挂载点 |
| `src/MatrixClientPeg.ts` | Matrix 客户端单例挂载点 |
| `src/SdkConfig.ts` | SDK 配置解析（读取 config.json） |
| `src/Views.ts` | 页面路由枚举（登录、注册、房间、设置等） |

### 2.2 UI 组件

| 路径 | 说明 |
|------|------|
| `src/components/structures/` | **页面级结构组件**（顶层布局容器） |
| ├─ `MatrixChat.tsx` | 应用主容器 |
| ├─ `HomePage.tsx` | 首页 |
| ├─ `RoomView.tsx` | 房间视图 |
| ├─ `LoggedInView.tsx` | 已登录主界面 |
| ├─ `SpacePanel.tsx` | 空间面板 |
| ├─ `UserView.tsx` | 用户视图 |
| ├─ `GroupView.tsx` | 群组视图 |
| └─ `ContextMenu.tsx` | 右键菜单 |
| `src/components/views/` | **功能视图组件**（按功能分子目录） |
| ├─ `auth/` | 登录 / 注册 / 认证页面 |
| ├─ `rooms/` | 房间相关视图（消息列表、成员列表、房间头部等） |
| ├─ `settings/` | 设置面板（通用、安全、通知、实验室等） |
| ├─ `dialogs/` | 弹窗组件（确认框、输入框、邀请等） |
| ├─ `elements/` | 基础 UI 元素（Avatar、Button、Input、Tooltip 等） |
| ├─ `messages/` | 消息渲染组件（文本、图片、文件、音频等） |
| ├─ `right_panel/` | 右侧面板（房间信息、成员列表、文件等） |
| ├─ `spaces/` | 空间（Space）视图 |
| ├─ `toasts/` | 提示消息组件 |
| ├─ `voip/` | 语音 / 视频通话界面 |
| ├─ `beacon/` | 位置共享 |
| ├─ `polls/` | 投票 |
| ├─ `location/` | 地图 / 位置 |
| ├─ `threads/` | 消息线程 |
| ├─ `emojipicker/` | 表情选择器 |
| ├─ `presence/` | 在线状态 |
| └─ `beta/` | 实验性功能 |
| `src/components/viewmodels/` | **ViewModel 层**（MVVM 架构的 VM） |

### 2.3 数据与状态管理

| 路径 | 说明 |
|------|------|
| `src/stores/` | **状态存储（Store）**，按功能分子目录 |
| ├─ `RoomListStore.ts` | 房间列表状态 |
| ├─ `RoomStore.ts` | 房间状态 |
| ├─ `WidgetStore.ts` | Widget 状态 |
| ├─ `NotificationStore.ts` | 通知状态 |
| ├─ `SpaceStore.ts` | 空间状态 |
| ├─ `RightPanelStore.ts` | 右侧面板状态 |
| ├─ `SettingStore.ts` | 设置状态 |
| └─ `...` | 更多 Store |
| `src/models/` | 数据模型 |
| ├─ `Call.ts` | 通话模型 |
| ├─ `LocalRoom.ts` | 本地房间模型 |
| ├─ `RoomUpload.ts` | 房间上传模型 |
| └─ `notificationsettings/` | 通知设置模型 |
| `src/dispatcher/` | 事件分发器 |
| ├─ `actions.ts` | Action 类型枚举 |
| ├─ `dispatcher.ts` | 分发器实例 |
| └─ `payloads/` | 39 个 Action Payload 类型定义 |
| `src/actions/` | Action 创建器 |
| ├─ `actionCreators.ts` | 全局 Action 创建器 |
| ├─ `MatrixActionCreators.ts` | Matrix 相关 Action |
| └─ `RoomListActions.ts` | 房间列表 Action |
| `src/settings/` | 设置系统 |
| ├─ `Settings.tsx` | 设置定义 |
| ├─ `SettingsStore.ts` | 设置存储 |
| ├─ `SettingLevel.ts` | 设置层级（设备/账户/房间/配置） |
| ├─ `controllers/` | 19 个设置控制器 |
| ├─ `handlers/` | 12 个设置处理器 |
| └─ `watchers/` | 设置监听器 |
| `src/contexts/` | React Context |
| ├─ `MatrixClientContext.tsx` | MatrixClient 上下文 |
| ├─ `RoomContext.ts` | 房间上下文 |
| ├─ `SDKContext.ts` | SDK 上下文 |
| ├─ `ToastContext.tsx` | Toast 上下文 |
| └─ `...` | 更多 Context |

### 2.4 功能模块

| 路径 | 说明 |
|------|------|
| `src/hooks/` | **React Hooks**（48 个，按功能分子目录） |
| ├─ `room/` | 房间相关 Hook |
| ├─ `right-panel/` | 右侧面板 Hook |
| ├─ `spotlight/` | Spotlight 搜索 Hook |
| ├─ `useSettings.ts` | 设置 Hook |
| ├─ `useProfileInfo.ts` | 用户信息 Hook |
| ├─ `useRoomMembers.ts` | 房间成员 Hook |
| ├─ `useTheme.ts` | 主题 Hook |
| ├─ `useCall.ts` | 通话 Hook |
| ├─ `useEncryptionStatus.ts` | 加密状态 Hook |
| └─ `...` | 更多 Hook |
| `src/modules/` | 模块扩展 API |
| ├─ `ModuleRunner.ts` | 模块运行器 |
| ├─ `ModuleFactory.ts` | 模块工厂 |
| ├─ `Api.ts` / `ClientApi.ts` | 模块 API |
| ├─ `Dialog.tsx` | 模块对话框 |
| └─ `components/` / `models/` | 模块组件与模型 |
| `src/editor/` | 富文本编辑器（WYSIWYG 底层实现） |
| ├─ `model.ts` | 编辑器模型 |
| ├─ `render.ts` | 渲染器 |
| ├─ `serialize.ts` / `deserialize.ts` | 序列化 / 反序列化 |
| ├─ `commands.tsx` | 编辑器命令 |
| ├─ `history.ts` | 撤销 / 重做 |
| ├─ `operations.ts` | 编辑操作 |
| └─ `...` | 更多编辑器模块 |
| `src/autocomplete/` | 自动补全系统 |
| ├─ `Autocompleter.ts` | 补全器 |
| ├─ `UserProvider.tsx` | @用户补全 |
| ├─ `RoomProvider.tsx` | #房间补全 |
| ├─ `EmojiProvider.tsx` | :emoji: 补全 |
| ├─ `CommandProvider.tsx` | /命令补全 |
| ├─ `SpaceProvider.tsx` | 空间补全 |
| └─ `NotifProvider.tsx` | @room 通知补全 |
| `src/notifications/` | 通知系统 |
| ├─ `ContentRules.ts` | 内容匹配规则 |
| ├─ `NotificationUtils.ts` | 通知工具 |
| ├─ `PushRuleVectorState.ts` | Push 规则状态 |
| └─ `VectorPushRulesDefinitions.ts` | Push 规则定义 |
| `src/events/` | 事件渲染工厂 |
| ├─ `EventTileFactory.tsx` | 事件瓦片工厂（消息类型 → 组件） |
| ├─ `RelationsHelper.ts` | 事件关系辅助 |
| └─ `forward/` / `location/` | 转发 / 位置事件 |
| `src/slash-commands/` | 斜杠命令 |
| ├─ `Command.ts` | 命令基类 |
| └─ 具体命令实现（/me、/kick、/ban、/invite 等） |
| `src/widgets/` | Widget 小部件集成 |
| `src/integrations/` | 集成管理器 |
| ├─ `IntegrationManagers.ts` | 集成管理器 |
| └─ `IntegrationManagerInstance.ts` | 集成管理器实例 |
| `src/effects/` | 视觉特效 |
| ├─ `confetti/` | 彩带特效 |
| ├─ `fireworks/` | 烟花特效 |
| ├─ `hearts/` | 爱心特效 |
| ├─ `snowfall/` | 雪花特效 |
| ├─ `rainfall/` | 雨滴特效 |
| └─ `spaceinvaders/` | 太空侵略者彩蛋 |
| `src/audio/` | 音频 / 语音消息 |
| ├─ `PlaybackManager.ts` | 播放管理器 |
| ├─ `VoiceRecording.ts` | 语音录制 |
| ├─ `VoiceMessageRecording.ts` | 语音消息录制 |
| ├─ `RecorderWorklet.ts` | 录制 Worklet |
| └─ `...` | 更多音频模块 |
| `src/emojipicker/` | 表情选择器历史记录 |
| `src/device-listener/` | 设备监听 |
| ├─ `DeviceListener.ts` | 设备监听器 |
| ├─ `DeviceState.ts` | 设备状态 |
| └─ `...` | 更多设备模块 |

### 2.5 工具与基础设施

| 路径 | 说明 |
|------|------|
| `src/utils/` | **工具函数**（105 个文件，最丰富的目录） |
| ├─ `arrays.ts` / `sets.ts` / `maps.ts` | 集合工具 |
| ├─ `strings.ts` / `numbers.ts` | 字符串 / 数字工具 |
| ├─ `date.ts` / `DateUtils.ts` | 日期工具 |
| ├─ `colors.ts` | 颜色工具 |
| ├─ `exportUtils.ts` | 导出工具 |
| ├─ `FileUtils.ts` | 文件工具 |
| ├─ `Image.ts` / `ImageUtils.ts` | 图片工具 |
| ├─ `MediaUtils.ts` | 媒体工具 |
| ├─ `UrlUtils.ts` | URL 工具 |
| ├─ `FormattingUtils.ts` | 格式化工具 |
| ├─ `membership.ts` | 成员状态工具 |
| ├─ `permissions.ts` | 权限工具 |
| ├─ `StorageUtils.ts` | 存储工具 |
| ├─ `ThreepidUtils.ts` | 第三方标识工具 |
| ├─ `units.ts` | 单位转换 |
| ├─ `oidc/` | OIDC 认证工具 |
| ├─ `Validator.ts` | 验证器 |
| ├─ `WellKnownUtils.ts` | Well-Known 配置工具 |
| ├─ `WidgetUtils.ts` | Widget 工具 |
| ├─ `exportUtils/` | 导出工具 |
| ├─ `location/` | 位置工具 |
| ├─ `beacon/` | Beacon 工具 |
| └─ `...` | 更多工具函数 |
| `src/customisations/` | 定制化钩子 |
| ├─ `ComponentVisibility.ts` | 组件可见性控制 |
| ├─ `Media.ts` | 媒体定制 |
| ├─ `Directory.ts` | 目录定制 |
| ├─ `WidgetPermissions.ts` | Widget 权限 |
| └─ `...` | 更多定制化钩子 |
| `src/i18n/strings/` | 国际化翻译文件（41 种语言） |
| ├─ `en_EN.json` | 英文源文件 |
| ├─ `zh_Hans.json` | 简体中文 |
| ├─ `zh_Hant.json` | 繁体中文 |
| └─ `...` | 38 种其他语言 |
| `src/renderer/` | 特殊内容渲染器 |
| ├─ `code-block.tsx` | 代码块渲染 |
| ├─ `pill.tsx` | Pill（@提及）渲染 |
| ├─ `spoiler.tsx` | Spoiler 折叠渲染 |
| ├─ `link-tooltip.tsx` | 链接提示 |
| └─ `utils.tsx` | 渲染工具 |
| `src/workers/` | Web Workers |
| ├─ `blurhash.worker.ts` | Blurhash 解码 Worker |
| ├─ `indexeddb.worker.ts` | IndexedDB Worker |
| ├─ `opus-decoder.worker.ts` | Opus 解码 Worker |
| ├─ `opus-encoder.worker.ts` | Opus 编码 Worker |
| └─ `...` | 更多 Worker |
| `src/accessibility/` | 无障碍 |
| ├─ `RovingTabIndex.tsx` | Roving Tab Index |
| ├─ `Toolbar.tsx` | 工具栏 |
| ├─ `KeyboardShortcuts.ts` | 键盘快捷键 |
| ├─ `LandmarkNavigation.ts` | 地标导航 |
| └─ `context_menu/` / `roving/` | 上下文菜单 / Roving 实现 |
| `src/indexing/` | 事件索引 |
| ├─ `EventIndex.ts` | 事件索引 |
| ├─ `EventIndexPeg.ts` | 事件索引挂载点 |
| └─ `BaseEventIndexManager.ts` | 事件索引管理器 |
| `src/mjolnir/` | 封禁列表管理 |
| ├─ `BanList.ts` | 封禁列表 |
| ├─ `ListRule.ts` | 列表规则 |
| └─ `Mjolnir.ts` | Mjolnir 主模块 |
| `src/performance/` | 性能监控 |
| ├─ `index.ts` | 性能监控入口 |
| └─ `entry-names.ts` | 性能条目名称 |
| `src/rageshake/` | 错误反馈提交 |
| `src/toasts/` | Toast 提示管理 |
| `src/resizer/` | 面板拖拽调整大小 |
| `src/serviceworker/` | Service Worker 注册 |
| `src/async-components/` | 异步加载组件 |
| `src/ContentView.ts` | 内容视图 |
| `src/Preferences.ts` | 用户偏好 |

### 2.6 全局核心文件

| 路径 | 说明 |
|------|------|
| `src/Login.ts` | 登录流程 |
| `src/Registration.tsx` | 注册流程 |
| `src/theme.ts` | 主题系统 |
| `src/ContentMessages.ts` | 消息内容发送 |
| `src/Rooms.ts` | 房间工具函数 |
| `src/Unread.ts` | 未读消息计数 |
| `src/Notifier.ts` | 桌面通知 |
| `src/Linkify.ts` | 链接解析 |
| `src/Markdown.ts` | Markdown 渲染 |
| `src/Modal.tsx` | 模态框系统 |
| `src/Keyboard.ts` | 键盘快捷键管理 |
| `src/KeyBindingsManager.ts` | 快捷键绑定 |
| `src/verification.ts` | 设备 / 用户验证 |
| `src/SecurityManager.ts` | 安全管理 |
| `src/Searching.ts` | 搜索功能 |
| `src/SendHistoryManager.ts` | 发送历史管理 |
| `src/SlidingSyncManager.ts` | 滑动同步管理 |
| `src/TimezoneHandler.ts` | 时区处理 |
| `src/WhoIsTyping.ts` | 正在输入检测 |
| `src/DraftCleaner.ts` | 草稿清理 |
| `src/DecryptionFailureTracker.ts` | 解密失败追踪 |
| `src/PosthogAnalytics.ts` | 埋点分析 |
| `src/sentry.ts` | Sentry 错误追踪 |
| `src/branding.ts` | 品牌化配置 |
| `src/favicon.ts` | Favicon 管理 |
| `src/languageHandler.tsx` | 语言处理 |
| `src/createRoom.ts` | 创建房间 |
| `src/Resend.ts` | 消息重发 |
| `src/Roles.ts` | 角色定义 |
| `src/Terms.ts` | 服务条款 |
| `src/SupportedBrowser.ts` | 浏览器兼容检测 |
| `src/UserActivity.ts` | 用户活动检测 |
| `src/WorkerManager.ts` | Worker 管理器 |

---

## 三、快速定位指南

| 想找什么 | 去这里 |
|----------|--------|
| 登录 / 注册页面 | `src/components/views/auth/` |
| 消息列表 / 聊天界面 | `src/components/views/rooms/` |
| 设置页面 | `src/components/views/settings/` |
| 右侧面板（房间信息、成员） | `src/components/views/right_panel/` |
| 空间（Space） | `src/components/views/spaces/` |
| 语音 / 视频通话 | `src/components/views/voip/` |
| 表情选择器 | `src/components/views/emojipicker/` |
| 消息线程 | `src/components/views/threads/` |
| 投票 | `src/components/views/polls/` |
| 位置共享 | `src/components/views/beacon/` / `src/components/views/location/` |
| 基础 UI 组件 | `src/components/views/elements/` |
| 弹窗组件 | `src/components/views/dialogs/` |
| 状态管理 | `src/stores/` |
| 事件分发 / Action | `src/dispatcher/` |
| 自定义 Hook | `src/hooks/` |
| 工具函数 | `src/utils/` |
| 翻译文案 | `src/i18n/strings/en_EN.json` |
| 主题样式 | `res/themes/` + `src/theme.ts` |
| 富文本编辑器 | `src/editor/` |
| 自动补全（@用户、:emoji 等） | `src/autocomplete/` |
| 通知 / Push 规则 | `src/notifications/` |
| 斜杠命令 | `src/slash-commands/` |
| E2E 加密相关 | `src/verification.ts` + `src/SecurityManager.ts` |
| 视觉特效 | `src/effects/` |
| 音频 / 语音消息 | `src/audio/` |
| Widget 小部件 | `src/widgets/` |
| 键盘快捷键 | `src/Keyboard.ts` + `src/KeyBindingsManager.ts` |
| 无障碍 | `src/accessibility/` |
| 性能监控 | `src/performance/` |
| 错误追踪 | `src/sentry.ts` + `src/rageshake/` |
| Webpack 配置 | `webpack.config.ts` |
| Nx 任务配置 | `project.json` |
| 运行时配置 | `config.json` |
| E2E 测试 | `playwright/e2e/` |
| 单元测试 | `test/unit-tests/` |

---

## 四、架构分层

```
┌─────────────────────────────────────────────────────┐
│  structures/  页面级容器组件（MatrixChat, RoomView）  │
├─────────────────────────────────────────────────────┤
│  views/       功能视图组件（auth, rooms, settings）   │
├─────────────────────────────────────────────────────┤
│  viewmodels/  ViewModel 层（MVVM 架构）              │
├─────────────────────────────────────────────────────┤
│  stores/      状态管理（RoomListStore, RoomStore）    │
├─────────────────────────────────────────────────────┤
│  models/      数据模型（Call, LocalRoom）             │
├─────────────────────────────────────────────────────┤
│  dispatcher/  事件分发（Action → Payload）            │
├─────────────────────────────────────────────────────┤
│  hooks/       React Hooks（useSettings, useRoomMembers）│
├─────────────────────────────────────────────────────┤
│  utils/       工具函数（105 个文件）                   │
└─────────────────────────────────────────────────────┘
```