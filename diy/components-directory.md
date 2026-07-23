# components 目录结构

## 概述

`apps/web/src/components/` 是 Element Web **核心 UI 组件目录**，遵循 **MVVM 架构模式**。整个目录分为三大模块：

| 模块 | 职责 |
|------|------|
| `structures/` | **页面级结构组件** — 应用骨架、路由、布局、页面级容器 |
| `viewmodels/` | **ViewModel 层** — UI 逻辑，直接操作 Model（matrix-js-sdk / Store） |
| `views/` | **View 层** — 哑组件，从 ViewModel 获取数据并渲染 |

```
components/
├── structures/       ← 页面骨架与顶层布局
├── viewmodels/       ← MVVM ViewModel 层
└── views/            ← MVVM View 层
```

---

## structures/（页面结构组件）

**路径：** `apps/web/src/components/structures/`

负责应用的**顶层框架**：页面路由、左右面板布局、认证流程、空间层级、消息展示面板等。

```
structures/
├── auth/                        ← 认证相关页面
│   ├── forgot-password/         ← 忘记密码流程组件
│   ├── header/                  ← 认证页头部组件
│   ├── CompleteSecurity.tsx     ← 完成安全设置
│   ├── ConfirmSessionLockTheftView.tsx
│   ├── E2eSetup.tsx             ← E2E 加密设置
│   ├── ForgotPassword.tsx       ← 忘记密码
│   ├── Login.tsx                ← 登录页面
│   ├── LoginSplashView.tsx      ← 登录启动页
│   ├── Registration.tsx         ← 注册页面
│   ├── SessionLockStolenView.tsx
│   ├── SetupEncryptionBody.tsx
│   └── SoftLogout.tsx           ← 软登出页面
├── grouper/                     ← 事件分组器
│   ├── BaseGrouper.ts           ← 基础分组逻辑
│   ├── CreationGrouper.tsx      ← 创建事件分组
│   ├── LateEventGrouper.ts      ← 延迟事件分组
│   └── MainGrouper.tsx          ← 主分组器
├── MatrixChat.tsx               ← **根组件**，路由控制中心
├── LoggedInView.tsx             ← 登录后的根视图
├── MainSplit.tsx                ← 主分栏布局(左/中/右)
├── LeftPanel.tsx                ← 左侧面板(房间列表)
├── RightPanel.tsx               ← 右侧面板(详情/设置)
├── RoomView.tsx                 ← 房间主视图
├── MessagePanel.tsx             ← 消息列表面板
├── TimelinePanel.tsx            ← 时间线面板
├── ThreadPanel.tsx              ← 线程面板
├── ThreadView.tsx               ← 线程视图
├── HomePage.tsx                 ← 首页
├── UserView.tsx                 ← 用户视图
├── SpaceRoomView.tsx            ← 空间房间视图
├── SpaceHierarchy.tsx           ← 空间层级结构
├── SpacePillButton.tsx          ← 空间药丸按钮
├── SearchBox.tsx                ← 搜索输入框
├── RoomSearch.tsx               ← 房间搜索
├── RoomSearchView.tsx           ← 房间搜索视图
├── NotificationPanel.tsx        ← 通知面板
├── FilePanel.tsx                ← 文件面板
├── SplashPage.tsx               ← 启动屏
├── UploadBar.tsx                ← 上传进度条
├── ScrollPanel.tsx              ← 可滚动面板
├── IndicatorScrollbar.tsx       ← 自定义滚动条
├── ToastContainer.tsx           ← Toast 通知容器
├── NonUrgentToastContainer.tsx  ← 非紧急 Toast 容器
├── BackdropPanel.tsx            ← 背景遮罩面板
├── ContextMenu.tsx              ← 右键菜单
├── AutocompleteInput.tsx        ← 自动补全输入
├── EmbeddedPage.tsx             ← 嵌入页面容器
├── ErrorMessage.tsx             ← 错误信息展示
├── FileDropTarget.tsx           ← 文件拖拽目标
├── GenericDropdownMenu.tsx      ← 通用下拉菜单
├── InteractiveAuth.tsx          ← 交互式认证
├── LargeLoader.tsx              ← 大尺寸加载器
├── LegacyCallEventGrouper.ts    ← 旧版通话事件分组
├── MatrixClientContextProvider.tsx
├── PictureInPictureDragger.tsx  ← 画中画拖拽器
├── PipContainer.tsx             ← 画中画容器
├── ReleaseAnnouncement.tsx      ← 版本发布公告
├── TabbedView.tsx               ← 标签页视图
├── ViewSource.tsx               ← 查看消息源码
├── WaitingForThirdPartyRoomView.tsx
└── static-page-vars.ts          ← 静态页面变量
```

> **重构提示：** `structures/auth/` 子目录集中了所有认证相关页面（登录、注册、密码重置等），新的认证流程应优先放在此目录下。

---

## viewmodels/（视图模型层）

**路径：** `apps/web/src/components/viewmodels/`

MVVM 模式中的 **ViewModel** 层。负责从 Model（matrix-js-sdk、Store、Customisations）获取/操作数据，提供给 View 层消费。遵循 `useViewModel` hook 机制，通过 `useSyncExternalStore` 驱动视图更新。

```
viewmodels/
├── avatars/
│   └── RoomAvatarViewModel.tsx       ← 房间头像 ViewModel
├── memberlist/
│   ├── tiles/
│   │   ├── MemberTileViewModel.tsx   ← 成员卡片 ViewModel
│   │   └── ThreePidTileViewModel.tsx  ← 第三方 ID 卡片 ViewModel
│   └── MemberListViewModel.tsx       ← 成员列表 ViewModel
├── right_panel/
│   ├── user_info/
│   │   ├── admin/
│   │   │   ├── UserInfoAdminToolsContainerViewModel.tsx
│   │   │   ├── UserInfoBanButtonViewModel.tsx
│   │   │   ├── UserInfoKickButtonViewModel.tsx
│   │   │   ├── UserInfoMuteButtonViewModel.tsx
│   │   │   └── UserInfoRedactButtonViewModel.tsx
│   │   ├── UserInfoBasicOptionsViewModel.tsx
│   │   ├── UserInfoBasicViewModel.tsx
│   │   ├── UserInfoHeaderVerificationViewModel.tsx
│   │   ├── UserInfoHeaderViewModel.tsx
│   │   └── UserInfoIgnoreButtonViewModel.tsx
│   ├── RoomSummaryCardViewModel.tsx
│   ├── RoomSummaryCardTopicViewModel.tsx
│   └── UserInfoPowerlevelViewModel.tsx
├── rooms/
│   └── UserIdentityWarningViewModel.tsx
└── settings/encryption/
    └── KeyStoragePanelViewModel.ts   ← 密钥存储面板 ViewModel
```

> **命名约定：** `{Feature}ViewModel` 文件名即 ViewModel 类名。View 层通过 `FooView` 组件引用 `FooViewModel` 类型。

> **重构提示：** viewmodels 当前只覆盖了部分功能（avatar、memberlist、right_panel），其余功能仍使用传统模式。逐步向 MVVM 迁移时，新增 ViewModel 应放在对应的功能子目录中。

---

## views/（视图层）

**路径：** `apps/web/src/components/views/`

MVVM 模式中的 **View 层**。哑组件，不直接操作业务逻辑，从 ViewModel 获取快照（Snapshot）并渲染。同时也包含大量尚未迁移到 MVVM 的传统组件。

按功能领域划分的 19 个子目录：

### auth/（认证视图组件）
认证流程中的 UI 片段，与 `structures/auth/` 配合使用。
```
views/auth/
├── AuthBody.tsx                ← 认证主体容器
├── AuthFooter.tsx              ← 认证页脚
├── AuthHeader.tsx              ← 认证页头部
├── AuthHeaderLogo.tsx          ← 认证页 Logo
├── AuthPage.tsx                ← 认证页面框架
├── CaptchaForm.tsx             ← CAPTCHA 表单
├── CompleteSecurityBody.tsx    ← 安全设置完成主体
├── CountryDropdown.tsx         ← 国家下拉选择
├── DefaultWelcome.tsx          ← 默认欢迎页
├── EmailField.tsx              ← 邮箱输入字段
├── InteractiveAuthEntryComponents.tsx
├── LanguageSelector.tsx        ← 语言选择器
├── LoginWithQR.tsx             ← QR 码登录
├── LoginWithQR-types.ts
├── LoginWithQRFlow.tsx         ← QR 码登录流程
├── PassphraseConfirmField.tsx  ← 密码短语确认
├── PassphraseField.tsx         ← 密码短语输入
├── PasswordLogin.tsx           ← 密码登录表单
├── RegistrationForm.tsx        ← 注册表单
└── Welcome.tsx                 ← 欢迎页面
```

### audio_messages/（音频消息）
音频播放与录制 UI。
```
views/audio_messages/
├── AudioPlayerBase.tsx         ← 音频播放器基类
├── LegacySeekBar.tsx           ← 旧版进度条
├── LiveRecordingClock.tsx      ← 实时录制时钟
├── LiveRecordingWaveform.tsx   ← 实时录制波形
├── PlayPauseButton.tsx         ← 播放/暂停按钮
├── PlaybackClock.tsx           ← 回放时钟
├── PlaybackWaveform.tsx        ← 回放波形
├── RecordingPlayback.tsx       ← 录制回放
└── Waveform.tsx                ← 波形图基础组件
```

### avatars/（头像组件）
```
views/avatars/
├── BaseAvatar.tsx              ← 基础头像
├── DecoratedRoomAvatar.tsx     ← 装饰房间头像
├── MemberAvatar.tsx            ← 成员头像
├── RoomAvatar.tsx              ← 房间头像
├── RoomAvatarView.tsx          ← 房间头像视图(MVVM)
├── SearchResultAvatar.tsx      ← 搜索结果头像
├── WidgetAvatar.tsx            ← Widget 头像
└── WithPresenceIndicator.tsx   ← 在线状态指示器
```

### beacon/（位置共享）
实时位置共享（Beacon）相关组件。
```
views/beacon/
├── BeaconListItem.tsx
├── BeaconMarker.tsx
├── BeaconStatus.tsx
├── BeaconStatusTooltip.tsx
├── BeaconViewDialog.tsx
├── DialogOwnBeaconStatus.tsx
├── DialogSidebar.tsx
├── LeftPanelLiveShareWarning.tsx
├── LiveTimeRemaining.tsx
├── OwnBeaconStatus.tsx
├── RoomCallBanner.tsx
├── ShareLatestLocation.tsx
├── StyledLiveBeaconIcon.tsx
├── displayStatus.ts
└── index.tsx
```

### beta/（Beta 功能）
```
views/beta/
└── BetaCard.tsx                ← Beta 功能提示卡
```

### context_menus/（上下文菜单）
```
views/context_menus/
├── DeveloperToolsOption.tsx
├── DeviceContextMenu.tsx
├── DialpadContextMenu.tsx
├── GenericElementContextMenu.tsx
├── IconizedContextMenu.tsx
├── KebabContextMenu.tsx        ← 肉串菜单(三点菜单)
├── LegacyCallContextMenu.tsx
├── MessageContextMenu.tsx      ← 消息右键菜单
├── RoomGeneralContextMenu.tsx
├── RoomNotificationContextMenu.tsx
├── SpaceContextMenu.tsx
├── ThreadListContextMenu.tsx
└── WidgetContextMenu.tsx
```

### dialogs/（弹窗/对话框）

最大的子目录，涵盖所有弹窗组件。进一步按功能细分：

```
views/dialogs/
├── devtools/                   ← 开发者工具弹窗(13 文件)
├── invite/                     ← 邀请相关弹窗
├── security/                   ← 安全设置弹窗
├── spotlight/                  ← 搜索弹出框(6 文件)
├── BaseDialog.tsx              ← 弹窗基类
├── BugReportDialog.tsx         ← 错误报告
├── ChangelogDialog.tsx         ← 更新日志
├── CreateRoomDialog.tsx        ← 创建房间
├── CreateSectionDialog.tsx
├── CreateSubspaceDialog.tsx
├── DevtoolsDialog.tsx
├── ErrorDialog.tsx
├── FeedbackDialog.tsx
├── ForwardDialog.tsx           ← 转发消息
├── InviteDialog.tsx
├── LogoutDialog.tsx
├── ModuleUiDialog.tsx          ← 模块 UI
├── RoomSettingsDialog.tsx      ← 房间设置
├── ShareDialog.tsx             ← 分享
├── SpaceSettingsDialog.tsx     ← 空间设置
├── UserSettingsDialog.tsx      ← 用户设置
├── VerificationRequestDialog.tsx
├── ...(更多弹窗见上方完整列表)
```

### directory/（目录服务）
```
views/directory/
└── NetworkDropdown.tsx         ← 网络下拉选择器
```

### elements/（通用 UI 元素）

核心 UI 组件库，类似组件库的原子组件。
```
views/elements/
├── crypto/
│   └── ...                     ← 加密相关 UI
├── AccessibleButton.tsx        ← 无障碍按钮
├── AppTile.tsx                 ← 应用磁贴(Widget)
├── AppWarning.tsx              ← 应用警告
├── CopyableText.tsx            ← 可复制文本
├── DesktopCapturerSourcePicker.tsx
├── DialPadBackspaceButton.tsx  ← 拨号盘退格键
├── DialogButtons.tsx           ← 弹窗按钮组
├── Draggable.tsx               ← 可拖拽组件
├── Dropdown.tsx                ← 下拉框
├── EditableItemList.tsx        ← 可编辑列表
├── EditableText.tsx            ← 可编辑文本
├── EffectsOverlay.tsx          ← 特效叠加层
├── ErrorBoundary.tsx           ← 错误边界
├── ...(共 52+ 文件)
```

> 包含 `Tooltip`、`ToggleSwitch`、`Spinner`、`ProgressBar`、`Pill` 等大量基础 UI 组件。

### emojipicker/（表情选择器）
```
views/emojipicker/
└── ...(8 文件)                 ← 表情选择器及分类
```

### location/（位置共享）
```
views/location/
├── Map.tsx
├── MapError.tsx
├── MapFallback.tsx
├── Marker.tsx
├── SmartMarker.tsx
├── ZoomButtons.tsx
├── ShareDialogButtons.tsx
├── ShareType.tsx
├── index.tsx
└── shareLocation.ts
```

### messages/（消息渲染）
消息内容的渲染组件，按消息类型分发。
```
views/messages/
├── shared/                     ← 共享消息组件
├── CallEvent.tsx               ← 通话事件
├── CodeBlock.tsx               ← 代码块
├── EditHistoryMessage.tsx      ← 编辑历史
├── IBodyProps.ts               ← Body 属性接口
├── IMediaBody.ts               ← 媒体 Body 接口
├── LegacyCallEvent.tsx
├── MAudioBody.tsx              ← 音频消息
├── MBeaconBody.tsx             ← 位置分享消息
├── MBodyFactory.tsx            ← **消息 Body 工厂**(路由)
├── MImageReplyBody.tsx         ← 图片回复
├── MLocationBody.tsx           ← 位置消息
├── MPollBody.tsx               ← 投票消息
├── MStickerBody.tsx            ← 贴纸消息
├── MVoiceMessageBody.tsx       ← 语音消息
├── MVoiceOrAudioBody.tsx       ← 语音/音频消息
├── MessageEvent.tsx            ← 消息事件
├── RoomPredecessorTile.tsx     ← 前身房间提示
├── SenderProfile.tsx           ← 发送者信息
└── TextualBodyFactory.tsx      ← 文本 Body 工厂
```

### polls/（投票）
```
views/polls/
├── pollHistory/                ← 投票历史
└── PollOption.tsx
```

### right_panel/（右侧面板）
```
views/right_panel/
├── user_info/                  ← 用户信息视图
├── BaseCard.tsx                ← 卡片基类
├── EmptyState.tsx              ← 空状态
├── EncryptionInfo.tsx          ← 加密信息
├── EncryptionPanel.tsx         ← 加密面板
├── ExtensionsCard.tsx          ← 扩展(Widge)卡片
├── PinnedMessagesCard.tsx      ← 置顶消息卡片
├── RoomSummaryCardView.tsx     ← 房间摘要
├── TimelineCard.tsx            ← 时间线卡片
├── UserInfo.tsx                ← 用户信息
├── VerificationPanel.tsx       ← 验证面板
├── WidgetCard.tsx              ← Widget 卡片
├── context.ts
└── types.ts
```

### room_settings/（房间设置）
```
views/room_settings/
├── AliasSettings.tsx           ← 别名设置
├── RoomProfileSettings.tsx     ← 房间资料设置
└── RoomPublishSetting.tsx      ← 房间发布设置
```

### rooms/（核心房间 UI）

最大最复杂的视图子目录，包含消息输入、房间列表、事件卡片等核心功能。
```
views/rooms/
├── EventTile/                  ← 事件卡片
├── MemberList/                 ← 成员列表
│   ├── tiles/
│   │   └── common/             ← 通用成员卡片
├── NotificationBadge/          ← 通知徽章
├── RoomHeader/                 ← 房间头部
│   └── toggle/                 ← 切换按钮
├── RoomListPanel/              ← 房间列表面板
├── wysiwyg_composer/           ← 富文本编辑器
│   ├── components/             ← 编辑器子组件
│   ├── hooks/                  ← 编辑器 Hooks
│   └── utils/                  ← 编辑器工具函数
├── AppsDrawer.tsx              ← 应用抽屉(Widge)
├── Autocomplete.tsx            ← 自动补全
├── BasicMessageComposer.tsx    ← 基础消息编辑器
├── EditMessageComposer.tsx     ← 编辑消息编辑器
├── MessageComposer.tsx         ← 消息编辑器(组合)
├── SendMessageComposer.tsx     ← 发送消息编辑器
├── VoiceRecordComposerTile.tsx ← 语音录制
├── EventTile.tsx               ← 事件卡片
├── RoomTile.tsx                ← 房间卡片(Space 面板)
├── RoomSublist.tsx             ← 房间子列表
├── ReadReceiptGroup.tsx        ← 已读回执
├── ReadReceiptMarker.tsx       ← 已读标记
├── SearchResultTile.tsx
├── PinnedEventTile.tsx         ← 置顶事件
├── PinnedMessageBanner.tsx     ← 置顶消息横幅
├── NotificationBadge.tsx       ← 通知徽章
├── NotificationDecoration.tsx  ← 通知装饰
├── JumpToBottomButton.tsx      ← 跳到底部
├── NewRoomIntro.tsx            ← 新房间介绍
├── RoomPreviewBar.tsx          ← 房间预览栏
├── RoomPreviewCard.tsx         ← 房间预览卡片
├── RoomKnocksBar.tsx           ← 敲门(申请加入)栏
├── RoomUpgradeWarningBar.tsx   ← 房间升级警告
├── TopUnreadMessagesBar.tsx    ← 未读消息提示
├── PresenceLabel.tsx           ← 在线状态标签
├── WhoIsTypingTile.tsx         ← 正在输入提示
├── Stickerpicker.tsx           ← 贴纸选择器
├── EmojiButton.tsx             ← 表情按钮
├── ...(共约 49 个文件)
```

### settings/（设置面板）
```
views/settings/
├── devices/                    ← 设备管理(16文件)
├── discovery/                  ← 发现设置
├── encryption/                 ← 加密设置(10文件)
├── notifications/              ← 通知设置
├── shared/                     ← 共享设置组件
├── tabs/
│   ├── room/                   ← 房间设置标签页
│   └── user/                   ← 用户设置标签页
├── SettingsTab.tsx
├── ThemeChoicePanel.tsx        ← 主题选择
├── LayoutSwitcher.tsx          ← 布局切换
├── FontScalingPanel.tsx        ← 字体缩放
├── ImageSizePanel.tsx          ← 图片大小
├── ChangePassword.tsx          ← 修改密码
├── KeyboardShortcut.tsx        ← 快捷键设置
├── Notifications.tsx           ← 通知设置
├── UserProfileSettings.tsx     ← 用户资料
├── UserPersonalInfoSettings.tsx
├── ...(共 24+ 文件)
```

### spaces/（空间管理）
```
views/spaces/
├── threads-activity-centre/   ← 线程活动中心
├── SpacePanel.tsx             ← 空间面板(侧边栏)
├── SpaceTreeLevel.tsx         ← 空间树层级
├── SpaceCreateMenu.tsx        ← 创建空间菜单
├── SpaceBasicSettings.tsx     ← 空间基本设置
├── SpaceSettingsGeneralTab.tsx
├── SpaceSettingsVisibilityTab.tsx
├── SpaceChildrenPicker.tsx    ← 空间子项选择
├── SpacePublicShare.tsx       ← 空间公开分享
├── QuickSettingsButton.tsx    ← 快速设置按钮
└── QuickThemeSwitcher.tsx     ← 快速主题切换
```

### 其他视图子目录

| 子目录 | 说明 |
|--------|------|
| `terms/` | 服务条款同意组件 |
| `toasts/` | Toast 通知(4 文件) |
| `typography/` | 排版组件(Caption, Heading) |
| `verification/` | 设备验证 UI(取消/完成/SAS) |
| `voip/` | 音视频通话 UI(10+ 文件) |

---

## MVVM 架构对应关系

| MVVM 层 | components/ 中的位置 |
|---------|---------------------|
| **Model** | matrix-js-sdk、Stores (`stores/`)、Customisations (`customisations/`) |
| **ViewModel** | `viewmodels/` 目录 |
| **View** | `views/` 目录 + `structures/` 目录 |

**View 与 ViewModel 的通信机制：**
- View 调用 `useViewModel` hook 获取 ViewModel 实例
- ViewModel 通过 `useSyncExternalStore` 驱动视图重渲染
- View 定义 `{Feature}ViewState` 快照接口和 `{Feature}ViewActions` 动作接口

---

## 重构指引

### 添加新功能建议
1. **页面级新功能** → `structures/` 下新建文件或子目录
2. **新的 ViewModel** → `viewmodels/` 下按功能子目录存放
3. **纯 UI 组件** → `views/` 下选择合适的子目录
4. **新弹窗** → `views/dialogs/`
5. **新消息类型** → `views/messages/`
6. **新房间 UI** → `views/rooms/`

### 迁移建议
- `structures/` 中的许多大型组件（如 `RoomView.tsx`、`MessagePanel.tsx`）可逐步拆分为 ViewModel + View 模式
- 认证相关视图 (`views/auth/`) 可与 `structures/auth/` 中的页面组件合并或统一目录
- `rooms/` 子目录文件过多(约 49 个)，可进一步按功能拆分（如 `composer/`、`room-list/`、`event-tile/`）