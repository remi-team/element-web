# Element Web — 完整功能列表与产品设计详情

> 基于 `apps/web` v1.12.21 源码深度分析，覆盖 ~620 个组件、~55 个 ViewModel、~54 个 Hook、78 种 Action、157 个 Setting、72 种 Dialog 的完整清单。

---

## 目录

1. [功能矩阵概览](#1-功能矩阵概览)
2. [完整组件清单](#2-完整组件清单)
3. [状态管理与数据流](#3-状态管理与数据流)
4. [完整设置清单](#4-完整设置清单)
5. [交互设计模式](#5-交互设计模式)
6. [认证与安全设计](#6-认证与安全设计)
7. [消息系统设计](#7-消息系统设计)
8. [通话系统设计](#8-通话系统设计)
9. [扩展与定制设计](#9-扩展与定制设计)

---

## 1. 功能矩阵概览

### 1.1 按产品域分类的功能清单

| 功能域 | 子功能 | 是否核心 | 复杂度 |
|--------|--------|---------|--------|
| **认证** | 密码登录 | 核心 | 中 |
| | SSO 单点登录 | 核心 | 中 |
| | OIDC 原生登录 | 核心 | 高 |
| | QR 码扫码登录 | 扩展 | 高 |
| | 访客模式 | 扩展 | 低 |
| | 软登出（Session 恢复） | 核心 | 中 |
| | 注册（含交互式认证） | 核心 | 高 |
| | 忘记密码（多阶段） | 核心 | 中 |
| | 会话锁（多标签页协调） | 基础设施 | 中 |
| **聊天** | 文本消息（Markdown） | 核心 | 中 |
| | 富文本编辑器（WYSIWYG） | 扩展 | 高 |
| | 图片消息（缩略图+灯箱） | 核心 | 中 |
| | 视频消息（播放器+缩略海报） | 核心 | 中 |
| | 音频消息（播放器+时长） | 核心 | 中 |
| | 文件附件 | 核心 | 中 |
| | 语音消息（录制+波形播放） | 核心 | 高 |
| | 贴纸 | 扩展 | 低 |
| | 投票 | 核心 | 中 |
| | 位置分享（静态+实时） | 扩展 | 高 |
| | 表情回应（Reactions） | 核心 | 中 |
| | 消息编辑 | 核心 | 中 |
| | 消息撤回 | 核心 | 低 |
| | 消息转发 | 核心 | 中 |
| | 消息引用/回复 | 核心 | 中 |
| | 线程（Threads） | 核心 | 高 |
| | 置顶消息 | 核心 | 低 |
| | URL 预览 | 核心 | 中 |
| | @提及（用户/房间/@room） | 核心 | 中 |
| | 消息搜索 | 核心 | 高 |
| | 拖拽上传 | 核心 | 低 |
| | LaTeX 数学公式 | 扩展 | 低 |
| | 代码语法高亮 | 核心 | 低 |
| | 剧透内容 | 核心 | 低 |
| | 彩虹色/特效文本 | 扩展 | 低 |
| | 聊天特效（烟花/雪花等） | 扩展 | 低 |
| **房间** | 创建房间（含多种可见性） | 核心 | 中 |
| | 加入房间（多种方式） | 核心 | 中 |
| | 房间预览（未加入时） | 核心 | 高 |
| | 房间设置（8 个设置标签） | 核心 | 高 |
| | 成员管理（邀请/踢出/封禁/权限） | 核心 | 高 |
| | 房间升级（协议版本迁移） | 核心 | 中 |
| | 离开/遗忘房间 | 核心 | 低 |
| | 房间目录（公开房间浏览） | 核心 | 中 |
| | 房间标签（收藏/低优先级等） | 核心 | 低 |
| | 导出聊天 | 扩展 | 中 |
| | 举报房间/消息 | 核心 | 低 |
| | 房间列表排序/筛选 | 核心 | 高 |
| **空间** | 创建/管理空间 | 核心 | 中 |
| | 空间层级（父子空间） | 核心 | 中 |
| | 元空间（首页/收藏/私聊/未分类） | 核心 | 中 |
| | 空间过滤（选中空间后过滤房间列表） | 核心 | 中 |
| **通话** | Legacy 1:1 WebRTC 通话 | 核心 | 高 |
| | Group 群组通话（Element Call） | 扩展 | 高 |
| | Jitsi 集成 | 扩展 | 高 |
| | 画中画（PiP） | 核心 | 中 |
| | 来电通知 | 核心 | 中 |
| | 通话设备管理 | 核心 | 中 |
| | PSTN 拨号 | 扩展 | 中 |
| | 通话转移 | 扩展 | 中 |
| **通知** | 桌面通知（浏览器/Electron） | 核心 | 中 |
| | 通知声音 | 核心 | 低 |
| | 推送通知（邮箱推送） | 核心 | 中 |
| | 通知偏好（全局+按房间） | 核心 | 高 |
| | 未读徽章 | 核心 | 中 |
| | 关键词提及 | 核心 | 低 |
| | 已读回执 | 核心 | 中 |
| **设置** | 外观（主题/布局/字体/图片大小） | 核心 | 中 |
| | 通知设置 | 核心 | 中 |
| | 偏好（语言/时间/表情/拼写/等等） | 核心 | 高 |
| | 安全与隐私 | 核心 | 高 |
| | 加密管理（密钥备份/交叉签名） | 核心 | 高 |
| | 实验室功能（26 个实验功能） | 扩展 | 中 |
| | 会话管理（设备列表） | 核心 | 中 |
| | 键盘快捷键（50+ 个） | 核心 | 低 |
| **搜索** | 全局搜索（Spotlight/Cmd+K） | 核心 | 高 |
| | 房间内消息搜索 | 核心 | 高 |
| | 用户目录搜索 | 核心 | 中 |
| | 公开房间搜索 | 核心 | 中 |
| | 事件索引（本地全文搜索） | 扩展 | 高 |
| **Widget** | Widget 嵌入（iframe） | 核心 | 中 |
| | 集成管理器（Scalar 兼容） | 扩展 | 高 |
| | Widget 权限管理 | 核心 | 中 |
| | AppsDrawer（顶部 Widget 栏） | 核心 | 中 |
| **桌面** | 系统托盘 | 核心 | 中 |
| | 自启动 | 扩展 | 低 |
| | 自动更新 | 核心 | 中 |
| | 原生通知 | 核心 | 低 |
| | 任务栏徽章 | 核心 | 低 |
| | 拼写检查 | 核心 | 中 |
| | 安全存储（safeStorage） | 核心 | 中 |
| | 深度链接（自定义协议） | 核心 | 中 |
| | 多账户支持 | 扩展 | 中 |
| | 屏幕共享选择器 | 核心 | 中 |
| **其他** | 国际化（40+ 语言） | 基础设施 | 高 |
| | 主题系统（含自定义主题） | 核心 | 中 |
| | 表情选择器 | 核心 | 中 |
| | 斜杠命令（40+ 条） | 核心 | 中 |
| | 无障碍（键盘导航/ARIA） | 核心 | 高 |
| | 模块扩展系统 | 基础设施 | 高 |
| | 定制化点（10 个 hook） | 基础设施 | 中 |

---

## 2. 完整组件清单

### 2.1 结构化组件（structures/）— 65 个

| 组件 | 文件 | 功能描述 |
|------|------|---------|
| MatrixChat | `MatrixChat.tsx` | 根组件，管理认证状态机、路由、全局对话框 |
| LoggedInView | `LoggedInView.tsx` | 登录后主布局三栏容器 |
| RoomView | `RoomView.tsx` | 房间视图控制器（2717 行），管理预览/加入/正常等状态 |
| LeftPanel | `LeftPanel.tsx` | 左侧面板容器 |
| RightPanel | `RightPanel.tsx` | 右侧面板，根据 RightPanelStore phase 渲染不同卡片 |
| MainSplit | `MainSplit.tsx` | 可拖拽水平分割面板 |
| TimelinePanel | `TimelinePanel.tsx` | 消息时间线（1911 行），管理分页、已读回执、阅读标记 |
| MessagePanel | `MessagePanel.tsx` | 消息列表渲染层（1125 行），支持事件连续和日期分隔 |
| ThreadPanel | `ThreadPanel.tsx` | 线程列表面板 |
| ThreadView | `ThreadView.tsx` | 单线程视图（461 行）|
| SpaceRoomView | `SpaceRoomView.tsx` | 空间房间视图 |
| SpaceHierarchy | `SpaceHierarchy.tsx` | 空间层级浏览 |
| HomePage | `HomePage.tsx` | 首页 |
| UserView | `UserView.tsx` | 用户资料视图 |
| SearchBox | `SearchBox.tsx` | 搜索输入框 |
| ScrollPanel | `ScrollPanel.tsx` | 虚拟滚动面板 |
| FileDropTarget | `FileDropTarget.tsx` | 文件拖拽目标区域 |
| UploadBar | `UploadBar.tsx` | 文件上传进度条 |
| ToastContainer | `ToastContainer.tsx` | Toast 通知容器 |
| NonUrgentToastContainer | `NonUrgentToastContainer.tsx` | 非紧急 Toast 容器 |
| PipContainer | `PipContainer.tsx` | 画中画容器 |
| PictureInPictureDragger | `PictureInPictureDragger.tsx` | PiP 拖拽控件 |
| RoomSearch | `RoomSearch.tsx` | 房间搜索触发器（Cmd+K 按钮） |
| RoomSearchView | `RoomSearchView.tsx` | 消息搜索结果视图 |
| FilePanel | `FilePanel.tsx` | 文件列表面板 |
| NotificationPanel | `NotificationPanel.tsx` | 通知列表面板 |
| TabbedView | `TabbedView.tsx` | 多标签视图容器 |
| InteractiveAuth | `InteractiveAuth.tsx` | 交互式认证流程 |
| ViewSource | `ViewSource.tsx` | 事件源码查看器 |
| ErrorMessage | `ErrorMessage.tsx` | 错误消息显示 |
| EmbeddedPage | `EmbeddedPage.tsx` | 嵌入页面视图 |
| ContextMenu | `ContextMenu.tsx` | 通用上下文菜单 |
| GenericDropdownMenu | `GenericDropdownMenu.tsx` | 通用下拉菜单 |
| ReleaseAnnouncement | `ReleaseAnnouncement.tsx` | 发布公告显示 |

**认证子组件（structures/auth/）— 14 个**

| Login | `Login.tsx` | 登录页面 |
| Registration | `Registration.tsx` | 注册页面 |
| ForgotPassword | `ForgotPassword.tsx` | 忘记密码页面（多阶段） |
| SoftLogout | `SoftLogout.tsx` | 软登出页面 |
| E2eSetup | `E2eSetup.tsx` | 首次加密设置 |
| CompleteSecurity | `CompleteSecurity.tsx` | 交叉签名设备验证 |
| SetupEncryptionBody | `SetupEncryptionBody.tsx` | 加密设置主体 |
| LoginSplashView | `LoginSplashView.tsx` | 登录同步加载画面 |
| SessionLockStolenView | `SessionLockStolenView.tsx` | 会话锁被窃取页面 |
| ConfirmSessionLockTheftView | `ConfirmSessionLockTheftView.tsx` | 确认接管会话锁页面 |

**事件分组器（structures/grouper/）— 4 个**

| MainGrouper.tsx | 主事件分组器 |
| CreationGrouper.tsx | 房间创建事件分组 |
| LateEventGrouper.ts | 延迟到达事件分组 |
| BaseGrouper.ts | 基础分组器 |

**认证交互组件（structures/auth/...）— 5 个**

| AuthHeaderProvider / AuthHeaderDisplay / AuthHeaderModifier / AuthHeaderContext | 认证页面头部 Context 系统 |
| CheckEmail / EnterEmail / VerifyEmailModal | 忘记密码邮箱验证子页面 |

### 2.2 视图组件（views/）— 约 450 个

#### 认证视图（views/auth/）— 20 个

| AuthPage | 认证页面容器 |
| AuthHeader / AuthHeaderLogo | 认证页面头部/Logo |
| AuthBody / AuthFooter | 认证页面正文/底部 |
| PasswordLogin | 密码登录表单（支持三种标识符） |
| RegistrationForm | 注册表单（含异步用户名校验） |
| PassphraseField / PassphraseConfirmField | 密码字段/确认密码字段 |
| EmailField / CountryDropdown | 邮箱字段/国家代码下拉 |
| CaptchaForm | reCAPTCHA 表单 |
| InteractiveAuthEntryComponents | 交互式认证入口组件集 |
| LanguageSelector | 语言选择器 |
| Welcome / DefaultWelcome | 欢迎页面 |
| LoginWithQR / LoginWithQRFlow | QR 登录组件/流程 |
| CompleteSecurityBody | 安全设置完成页面容器 |

#### 消息视图（views/messages/）— 20 个

| MessageEvent | 通用消息事件（工厂分发到具体 Body） |
| TextualBodyFactory | 文本消息生成器（m.text/m.notice/m.emote） |
| MAudioBody / MVoiceOrAudioBody / MVoiceMessageBody | 音频/语音消息体 |
| MBeaconBody | 实时位置信标体 |
| MFileBody (via MBodyFactory) | 文件附件体 |
| MImageReplyBody | 图片回复体 |
| MLocationBody | 位置消息体 |
| MPollBody | 投票消息体 |
| MStickerBody | 贴纸消息体 |
| CallEvent / LegacyCallEvent | 通话事件体 |
| CodeBlock | 代码块渲染（highlight.js） |
| SenderProfile | 发送者头像+名称 |
| EditHistoryMessage | 编辑历史消息标记 |
| RoomPredecessorTile | 房间前任标记 |
| IBodyProps / IMediaBody | 消息体接口定义 |

#### 房间视图（views/rooms/）— 约 60 个核心文件 + 子目录

**核心**

| EventTile | 事件卡片（1394 行） |
| MessageComposer | 消息输入框容器（726 行） |
| SendMessageComposer | 纯文本发送框（666 行） |
| EditMessageComposer | 编辑消息框 |
| BasicMessageComposer | 基础消息框 |
| RoomHeader | 房间头部（551 行） |
| RoomTile | 房间列表项 |
| RoomSublist | 房间子列表（可折叠分区） |
| RoomListPanel / RoomListView / RoomListSearch | 新房间列表面板/视图/搜索 |
| LegacyRoomList / LegacyRoomListHeader | 旧版房间列表 |
| RoomPreviewBar | 房间预览栏（766 行，多状态） |
| RoomPreviewCard | 房间预览卡片 |
| NotificationBadge / StatelessNotificationBadge | 通知徽章 |
| ReadReceiptGroup / ReadReceiptMarker | 已读回执组/标记 |
| JumpToBottomButton | 跳到底部按钮（44 行） |
| TopUnreadMessagesBar | 顶部未读消息条（41 行） |
| WhoIsTypingTile | 正在输入提示（218 行） |
| PinnedMessageBanner | 置顶消息横幅（296 行） |
| PinnedEventTile | 置顶消息卡片 |
| AppsDrawer | Widget 应用抽屉 |
| VoiceRecordComposerTile | 语音录制控件 |
| ReplyPreview / ReplyTile | 回复预览/回复卡片 |
| RoomKnocksBar | 敲门请求栏 |
| SearchResultTile | 搜索结果卡片 |
| Autocomplete | 自动补全（@用户/#房间/表情） |
| Stickerpicker | 贴纸选择器 |
| E2EIcon | 加密状态图标 |
| NewRoomIntro | 新房间引导 |
| RoomInfoLine | 房间信息行 |
| RoomBreadcrumbs | 房间面包屑 |
| AuxPanel | 辅助面板 |
| RoomUpgradeWarningBar | 房间升级警告栏 |

**EventTile 子组件（views/rooms/EventTile/）— 14 个**

| ActionBarAdapter | 操作栏适配器（回复/回应/编辑等按钮） |
| E2eStandardPadlockIcon | E2E 标准锁图标 |
| E2eMessageSharedIconAdapter | E2E 共享消息图标 |
| EventPreviewAdapter | 事件预览适配器 |
| EventTileFooter | 卡片脚注区 |
| EventTilePreviewBody | 卡片预览正文 |
| EventTileThreadInfo | 卡片线程信息 |
| EventTileTimestampSlot | 卡片时间戳位置 |
| MessageTimestampAdapter | 消息时间戳适配器 |
| ReactionsRowAdapter | 回应行适配器 |
| ReceiptAdapter | 已读回执适配器 |
| SenderIdentityAdapter | 发送者身份适配器 |
| ThreadListActionBarAdapter | 线程列表操作栏 |
| ThreadMessagePreviewAdapter | 线程消息预览 |

**成员列表子组件（views/rooms/MemberList/）— 10 个**

| MemberListView / MemberListHeaderView | 成员列表视图/头部 |
| RoomMemberTileView / ThreePidInviteTileView | 成员卡片/第三方邀请卡片 |
| E2EIconView / InvitedIconView / PresenceIconView | 加密/邀请/在线状态图标 |
| MemberTileView | 基础成员卡片 |

**房间头部子组件（views/rooms/RoomHeader/）— 5 个**

| RoomHeader | 主头部组件 |
| CallGuestLinkButton | 通话访客链接按钮 |
| VideoRoomChatButton | 视频房间聊天按钮 |
| ToggleableIcon / useToggled | 可切换图标/Hook |

**富文本编辑器（views/rooms/wysiwyg_composer/）— 约 25 个**

| SendWysiwygComposer / EditWysiwygComposer | 发送/编辑富文本编辑器 |
| WysiwygComposer / PlainTextComposer | 富文本/纯文本编辑器 |
| LinkModal | 链接插入弹窗 |
| FormattingButtons / Emoji | 格式按钮/表情按钮 |
| WysiwygAutocomplete | 富文本自动补全 |
| EditionButtons | 编辑模式按钮 |
| 13 个 Hook（useComposerFunctions, useEditing 等） |
| 9 个工具函数（autocomplete, createMessageContent 等） |

**通知徽章子组件（views/rooms/NotificationBadge/）— 2 个**

| StatelessNotificationBadge | 无状态徽章（纯渲染） |
| UnreadNotificationBadge | 未读徽章（含状态） |

#### 设置视图（views/settings/）— 约 80 个文件

**主设置组件（18 个）**

| AvatarSetting | 头像设置 |
| AddRemoveThreepids | 添加/移除 3PID |
| ChangePassword | 修改密码 |
| EmailAddresses | 邮箱地址管理 |
| PhoneNumbers | 手机号管理 |
| DiscoverySettings | 发现设置 |
| IntegrationManager | 集成管理器设置 |
| SetIdServer / SetIntegrationManager | 身份服务器/集成管理器 URL 设置 |
| LayoutSwitcher | 布局切换器（IRC/Group/Bubble） |
| ThemeChoicePanel | 主题选择面板 |
| FontScalingPanel | 字体缩放面板 |
| ImageSizePanel | 图片大小面板 |
| SpellCheckSettings | 拼写检查设置 |
| Notifications | 通知设置 |
| KeyboardShortcut | 键盘快捷键显示 |
| UpdateCheckButton | 检查更新按钮 |
| EventIndexPanel | 事件索引面板 |
| UserProfileSettings / UserPersonalInfoSettings | 用户资料/个人信息 |

**设备管理子组件（settings/devices/）— 17 个**

| CurrentDeviceSection / OtherSessionsSectionHeading | 当前设备区域/其他会话区域 |
| DeviceTile / SelectableDeviceTile | 设备卡片/可选设备卡片 |
| DeviceDetails / DeviceDetailHeading / DeviceExpandDetailsButton | 设备详情/头部/展开按钮 |
| DeviceMetaData / DeviceTypeIcon | 设备元数据/类型图标 |
| DeviceSecurityCard / DeviceVerificationStatusCard / DeviceSecurityLearnMore | 设备安全卡片/验证状态/了解更多 |
| FilteredDeviceList / FilteredDeviceListHeader | 过滤设备列表/头部 |
| LoginWithQRSection | QR 登录区域 |
| SecurityRecommendations | 安全建议 |
| deleteDevices / filter / useOwnDevices / types | 删除/过滤/Hook/类型 |

**加密设置子组件（settings/encryption/）— 9 个**

| EncryptionCard / EncryptionCardButtons / EncryptionCardEmphasisedContent | 加密卡片/按钮/强调内容 |
| KeyStoragePanel | 密钥存储面板 |
| RecoveryPanel / RecoveryPanelOutOfSync | 恢复面板/不同步状态 |
| ChangeRecoveryKey | 修改恢复密钥 |
| DeleteKeyStoragePanel | 删除密钥存储 |
| AdvancedPanel | 高级加密面板 |
| ResetIdentityPanel / ResetIdentityBody | 重置身份面板/主体 |

**通知设置子组件（settings/notifications/）— 2 个**

| NotificationSettings2 | 通知设置 V2 |
| NotificationPusherSettings | 推送通知设置 |

**共享设置组件（settings/shared/）— 4 个**

| SettingsBanner / SettingsSection / SettingsSubsection / SettingsIndent | 横幅/分区/子分区/缩进 |

**设置标签页（settings/tabs/）**

**房间设置标签（tabs/room/）— 9 个**

| GeneralRoomSettingsTab | 常规设置 |
| SecurityRoomSettingsTab | 安全与隐私（587 行） |
| RolesRoomSettingsTab | 角色与权限（476 行） |
| NotificationSettingsTab | 通知设置 |
| PeopleRoomSettingsTab | 人员管理 |
| AdvancedRoomSettingsTab | 高级设置 |
| VoipRoomSettingsTab | 语音视频设置 |
| BridgeSettingsTab | 桥接设置 |
| PollHistoryTab | 投票历史 |

**用户设置标签（tabs/user/）— 15 个**

| AccountUserSettingsTab | 账户设置 |
| AppearanceUserSettingsTab | 外观设置 |
| PreferencesUserSettingsTab | 偏好设置 |
| NotificationUserSettingsTab | 通知设置 |
| SecurityUserSettingsTab | 安全隐私设置 |
| EncryptionUserSettingsTab | 加密设置 |
| VoiceUserSettingsTab | 语音视频设置 |
| KeyboardUserSettingsTab | 键盘快捷键 |
| SidebarUserSettingsTab | 侧边栏设置 |
| LabsUserSettingsTab | 实验室功能 |
| HelpUserSettingsTab | 帮助与关于 |
| SessionManagerTab | 会话管理 |
| MjolnirUserSettingsTab | Mjolnir 管理 |
| InviteRulesAccountSettings | 邀请规则 |
| MediaPreviewAccountSettings | 媒体预览 |

#### 右侧面板（views/right_panel/）— 20 个

| BaseCard | 基础卡片（含返回按钮和历史栈） |
| RoomSummaryCardView | 房间摘要卡片 |
| MemberListView | 成员列表视图 |
| UserInfo | 用户信息（支持 MemberInfo/EncryptionPanel 两个 phase） |
| EncryptionInfo / EncryptionPanel / VerificationPanel | 加密信息/面板/验证面板 |
| ThreadPanel / ThreadView (structures/) | 线程面板/视图 |
| PinnedMessagesCard | 置顶消息卡片 |
| FilePanel (structures/) | 文件面板 |
| NotificationPanel (structures/) | 通知面板 |
| WidgetCard | Widget 卡片 |
| ExtensionsCard | 扩展列表卡片 |
| TimelineCard | 时间线卡片（含输入框） |
| EmptyState | 空状态占位 |
| ThirdPartyMemberInfo | 第三方成员信息 |

#### 对话框（views/dialogs/）— 72 个

| 类别 | 对话框 |
|------|--------|
| **基础** | BaseDialog, ScrollableBaseModal, QuestionDialog, ErrorDialog, InfoDialog, TextInputDialog |
| **认证** | InteractiveAuthDialog, IncomingSasDialog, ServerPickerDialog, ServerOfflineDialog, SetEmailDialog, RegistrationEmailPromptDialog |
| **房间** | CreateRoomDialog, RoomSettingsDialog, RoomUpgradeDialog, RoomUpgradeWarningDialog, LeaveSpaceDialog |
| **成员** | InviteDialog, InviteProgressDialog, ConfirmUserActionDialog, ConfirmSpaceUserActionDialog, DeclineAndBlockInviteDialog, ManageRestrictedJoinRuleDialog |
| **空间** | AddExistingToSpaceDialog, AddExistingSubspaceDialog, CreateSubspaceDialog, CreateSectionDialog, RemoveSectionDialog, SpacePreferencesDialog, SpaceSettingsDialog, ShareDialog |
| **安全/加密** | AccessSecretStorageDialog, InitialCryptoSetupDialog, RestoreKeyBackupDialog, SetupEncryptionDialog, ConfirmKeyStorageOffDialog, ManualDeviceKeyVerificationDialog, VerificationRequestDialog, ResetIdentityDialog |
| **消息** | ForwardDialog, MessageEditHistoryDialog, ReportEventDialog, ReportRoomDialog, BulkRedactDialog, ConfirmRedactDialog, EndPollDialog, UnpinAllDialog |
| **文件** | UploadConfirmDialog, UploadFailureDialog |
| **用户** | UserSettingsDialog, DeactivateAccountDialog, LogoutDialog |
| **Widget** | ModalWidgetDialog, WidgetCapabilitiesPromptDialog, WidgetOpenIDPermissionsDialog, IntegrationsDisabledDialog, IntegrationsImpossibleDialog, ModuleUiDialog |
| **开发** | DevtoolsDialog（含 13 个子面板）, BugReportDialog |
| **搜索** | SpotlightDialog（含子组件 Option, TooltipOption, PublicRoomResultDetails 等） |
| **其他** | ChangelogDialog, ExportDialog, FeedbackDialog, BetaFeedbackDialog, GenericFeatureFeedbackDialog, AnalyticsLearnMoreDialog, StorageEvictedDialog, TermsDialog, SeshatResetDialog, SessionRestoreErrorDialog, ConfirmWipeDeviceDialog, AskInviteAnywayDialog, UntrustedDeviceDialog, PollHistoryDialog, SlashCommandHelpDialog, PollCreateDialog |
| **通话** | DialPadModal |
| **定位** | LocationViewDialog, BeaconViewDialog |

#### 语音消息视图（views/audio_messages/）— 9 个

| PlayPauseButton | 播放/暂停按钮 |
| PlaybackWaveform / Waveform | 播放波形/基础波形 |
| PlaybackClock / LiveRecordingClock | 播放时钟/录制时钟 |
| LiveRecordingWaveform | 实时录制波形 |
| RecordingPlayback | 录制回放（含删除/发送） |
| AudioPlayerBase | 音频播放器基类 |
| LegacySeekBar | 旧版进度条 |

#### 通话视图（views/voip/）— 12 个

| LegacyCallView (LegacyCallViewForRoom) | 旧版 1:1 通话视图 |
| CallView | 新版 Widget 通话视图 |
| VideoFeed / AudioFeed / AudioFeedArrayForLegacyCall | 视频源/音频源/音频源数组 |
| DialPad / DialPadModal | 拨号盘/弹窗 |
| CallDuration / SessionDuration | 通话时长/会话时长 |
| LegacyCallViewButtons / LegacyCallViewHeader / LegacyCallViewSidebar | 按钮/头部/侧边栏 |

#### 空间视图（views/spaces/）— 14 个

| SpacePanel | 空间面板（左侧导航栏） |
| SpaceTreeLevel | 空间层级 |
| SpaceCreateMenu | 空间创建菜单（两步流程） |
| SpaceBasicSettings | 基础空间设置 |
| SpaceSettingsGeneralTab / SpaceSettingsVisibilityTab | 常规/可见性设置标签 |
| SpaceChildrenPicker | 子空间/房间选择器 |
| SpacePublicShare | 公开分享 |
| QuickSettingsButton / QuickThemeSwitcher | 快速设置/主题切换 |
| ThreadsActivityCentre / ThreadsActivityCentreButton | 线程活动中心/按钮 |

#### 其他视图

| 类别 | 组件 |
|------|------|
| **表情** | EmojiPicker, ReactionPicker, Category, Search, Preview, Header, Emoji, QuickReactions (8 个) |
| **位置** | LocationPicker, LocationButton, LocationShareMenu, Map, MapError, MapFallback, Marker, SmartMarker, ZoomButtons 等（16 个） |
| **信标** | BeaconListItem, BeaconMarker, BeaconStatus, BeaconStatusTooltip, BeaconViewDialog, DialogOwnBeaconStatus 等（15 个） |
| **头像** | BaseAvatar, RoomAvatar, MemberAvatar, DecoratedRoomAvatar, WidgetAvatar, SearchResultAvatar, WithPresenceIndicator 等（8 个） |
| **验证** | VerificationShowSas, VerificationComplete, VerificationCancelled (3 个) |
| **元素** | AccessibleButton, Spinner, Dropdown, ToggleSwitch, StyledCheckbox, StyledRadioButton, StyledRadioGroup, ProgressBar, QRCode, FacePile 等（66 个） |
| **上下文菜单** | MessageContextMenu, RoomGeneralContextMenu, SpaceContextMenu, DeviceContextMenu, WidgetContextMenu 等（13 个） |
| **Toast** | GenericToast, GenericExpiringToast, NonUrgentEchoFailureToast, VerificationRequestToast (4 个) |

---

## 3. 状态管理与数据流

### 3.1 架构概述

```
┌─────────────────────────────────────────────────────────────────┐
│                       MatrixDispatcher                          │
│                    (78 种 Action 类型)                           │
└──────┬──────────┬──────────┬──────────┬──────────┬─────────────┘
       │          │          │          │          │
   ┌───▼───┐  ┌──▼───┐  ┌──▼───┐  ┌──▼───┐  ┌──▼──────┐
   │Store A │  │Store B│  │Store C│  │Store D│  │ViewModel│
   │(Async  │  │       │  │       │  │       │  │         │
   │ Store) │  │       │  │       │  │       │  │         │
   └───┬───┘  └──┬───┘  └──┬───┘  └──┬───┘  └──┬──────┘
       │ Emit     │         │         │         │
       │ Events   │         │         │         │
   ┌───▼──────────▼─────────▼─────────▼─────────▼────────┐
   │                   React Components                   │
   │    (通过 useEventEmitter / useDispatcher / useContext) │
   └──────────────────────────────────────────────────────┘
```

### 3.2 核心 Store 清单

| Store | 职责 | 关键状态 |
|-------|------|---------|
| **RoomViewStore** | 当前查看的房间状态 | roomId, threadId, joining, initialEvent, replyingToEvent, promptAskToJoin |
| **RightPanelStore** | 右侧面板卡片历史栈 | isOpen, history[]（卡片栈）, viewedRoomId |
| **SpaceStore** | 空间层级和活动空间 | spaceMap, parentMap, activeSpace, metaSpaces |
| **WidgetStore** | 所有房间的 Widget | widgetMap, roomMap |
| **ActiveWidgetStore** | 活跃 Widget 状态 | persistentWidgetId, dockedWidgetRefs |
| **WidgetLayoutStore** | Widget 布局 | container assignments, heights, widths |
| **RoomNotificationStateStore** | 全局通知状态 | globalState (Summarized) |
| **RoomNotificationState** | 单房间通知状态 | level, count, symbol, muted |
| **RoomListStore / V3** | 房间列表排序 | tagged rooms, algorithms, filters |
| **CallStore** | 通话状态 | calls per room, connectedCalls |
| **TypingStore** | 正在输入状态 | per-room typing users, self-typing |
| **BreadcrumbsStore** | 最近访问房间 | breadcrumb_rooms (max 20) |
| **UIStore** | 窗口尺寸 | windowWidth, elementDimensions |
| **MemberListStore** | 成员列表 | sorted, filtered members |
| **WidgetEchoStore** | Widget 乐观更新 | pending widget adds/removals |
| **VoiceRecordingStore** | 录音状态 | recordings per room/thread |
| **OwnBeaconStore** | 自身实时位置 | own live beacons |
| **SetupEncryptionStore** | 加密设置向导 | phases: Loading → Intro → Busy → Done |
| **ToastStore** | 紧急 Toast | ordered by priority |
| **ReleaseAnnouncementStore** | 发布公告 | viewed features |

### 3.3 React Context 清单

| Context | 提供值 | 使用场景 |
|---------|--------|---------|
| MatrixClientContext | MatrixClient 实例 | 所有需要 API 调用的组件 |
| SDKContext | SdkContextClass（所有 Store 的延迟初始化） | 全局依赖注入 |
| RoomContext | 房间渲染状态（布局/权限/类型等） | RoomView 子树 |
| ScopedRoomContext | 性能优化的 RoomContext（按需订阅） | EventTile 等高频组件 |
| ToastContext | ToastRack 实例 | 设置对话框等作用域 Toast |
| CurrentRightPanelPhaseContext | 当前右侧面板 phase + isOpen | 右侧面板组件 |

### 3.4 数据流示例：查看房间

```
用户点击房间
  → RoomListViewModel dispatch(Action.ViewRoom, { room_id })
  → RoomViewStore.onDispatch()
    → resolveRoomAlias / createRoom
    → 设置 state: { roomId, roomLoading: true }
    → 请求 /sync 或 peekInRoom
    → 设置 state: { roomId, roomAlias, roomLoading: false, room }
    → dispatch(Action.ActiveRoomChanged)
      → RightPanelStore: 加载房间缓存的右侧面板状态
      → RoomListViewModel: 更新 sticky room
    → MatrixChat.showScreen() 渲染 RoomView
      → RoomView.componentDidMount()
        → 设置 timeline, pagination, read receipts
        → 渲染 RoomHeader + TimelinePanel + MessageComposer
```

### 3.5 Action 分类（78 种）

| 分类 | Action | 数量 |
|------|--------|------|
| 导航 | ViewRoom, ViewHomePage, ViewUser, ViewThread, SwitchSpace, ViewRoomDelta | 6 |
| 房间操作 | JoinRoom, JoinRoomReady, JoinRoomError, PromptAskToJoin, SubmitAskToJoin, CancelAskToJoin, AfterLeaveRoom, AfterForgetRoom | 8 |
| 设置 | ViewUserSettings, ViewUserDeviceSettings, SettingUpdated, RecheckTheme | 4 |
| 编辑器 | FocusSendMessageComposer, FocusEditMessageComposer, FocusAComposer, ClearAndFocusSendMessageComposer, ComposerInsert, EditEvent | 6 |
| 上传 | UploadStarted, UploadProgress, UploadFinished, UploadFailed, UploadCanceled | 5 |
| 认证 | OnLoggedIn, OnLoggedOut, TriggerLogout, OverwriteLogin, WillStartClient, ClientStarted, ClientNotViable, ViewQrLogin | 8 |
| 搜索 | OpenSpotlight, FocusMessageSearch, ViewRoomDirectory | 3 |
| 对话框 | OpenForwardDialog, OpenReportEventDialog, OpenSpacePreferences, OpenSpaceSettings, OpenInviteDialog, OpenAddToExistingSpaceDialog, OpenDialPad, CreateChat, CreateRoom | 9 |
| 线程 | ShowThread, FocusThreadsPanel, ViewThread | 3 |
| 面板 | ToggleUserMenu, ToggleSpacePanel, RoomListCollapseAllSections, RoomListExpandAllSections | 4 |
| 消息 | BulkRedactStart, BulkRedactEnd | 2 |
| 其他 | ViewStartChatOrReuse, ActiveRoomChanged, DoAfterSyncPrepared, RoomLoaded, View3pidInvite, ViewRoomError, Share, ShowRoomTopic, DumpDebugLogs, UserActivity, UpdateFontSizeDelta, MigrateBaseFontSize, UpdateSystemFont, CheckUpdates, ViewQrLogin, PseudonymousAnalyticsAccept/Reject | 17 |

### 3.6 ViewModel 清单（55 个）

| 领域 | ViewModel |
|------|----------|
| **房间列表** | RoomListViewModel, RoomListItemViewModel, RoomListHeaderViewModel, RoomListSectionHeaderViewModel, RoomListSearchViewModel |
| **事件卡片** | EventTileViewModel, TextualEventViewModel, CallTileViewModel, RoomAvatarEventViewModel, DisambiguatedProfileViewModel, EncryptionEventViewModel, MJitsiWidgetEventViewModel, MKeyVerificationRequestViewModel, ThreadSummaryViewModel, DateSeparatorViewModel, EventPreviewViewModel, MessageTimestampViewModel |
| **事件卡片状态** | EventTileInteractionState, EventTileHighlightState, EventTileReceiptState, EventTileVisibilityState, EventTileThreadState, EventTileReplyChainState, EventTileE2eState, EventTileDerivedState, EventTileReactionState |
| **事件卡片 E2E** | E2eMessageSharedIconViewModel, EventTileE2eViewModel |
| **事件卡片 Body** | TextualBodyViewModel, HiddenBodyViewModel, MjolnirBodyViewModel, DecryptionFailureBodyViewModel, ViewSourceEventViewModel, AudioPlayerViewModel |
| **回应** | ReactionsRowViewModel, ReactionsRowButtonViewModel, ReactionsRowButtonTooltipViewModel |
| **消息 Body** | ImageBodyViewModel, VideoBodyViewModel, FileBodyViewModel, TileErrorViewModel, UrlPreviewGroupViewModel, RedactedBodyViewModel, EventContentBodyViewModel, EditHistoryActionBarViewModel |
| **操作栏** | EventTileActionBarViewModel, ThreadListActionBarViewModel |
| **右侧面板** | RoomSummaryCardViewModel, RoomSummaryCardTopicViewModel, UserInfoPowerlevelViewModel, UserInfoBasicOptionsViewModel, UserInfoBasicViewModel, UserInfoHeaderVerificationViewModel, UserInfoHeaderViewModel, UserInfoIgnoreButtonViewModel, UserInfoAdminToolsContainerViewModel, UserInfoBanButtonViewModel, UserInfoKickButtonViewModel, UserInfoMuteButtonViewModel, UserInfoRedactButtonViewModel |
| **其他** | RoomUploadViewModel, RoomStatusBar, WidgetPipViewModel, WidgetContextMenuViewModel, UnreadNotificationBadgeViewModel, ResizerViewModel, UserMenuViewModel |

---

## 4. 完整设置清单

### 4.1 按类别统计

| 类别 | 数量 | 说明 |
|------|------|------|
| 功能标志 (isFeature) | 26 | 实验性/Beta 功能开关 |
| 布尔设置 | 81 | 外观、通知、消息、编辑器等开/关 |
| 字符串设置 | 8 | 主题、字体、语言、设备 ID 等 |
| 数字设置 | 9 | 字体大小、自动补全延迟、阅读标记阈值等 |
| 枚举设置 | 10 | 布局、图片大小、自启动模式等 |
| 复杂对象设置 | 23 | 数组/对象类型的设置 |
| **总计** | **157** | |

### 4.2 设置层级（8 级，优先级从高到低）

1. **DEVICE** — localStorage（`mx_local_settings`）
2. **ROOM_DEVICE** — localStorage（`mx_setting_*_<roomId>`）
3. **ROOM_ACCOUNT** — 房间账户数据（Matrix API）
4. **ACCOUNT** — 账户数据（Matrix API, `im.vector.web.settings`）
5. **ROOM** — 房间状态事件（管理员设置）
6. **PLATFORM** — Electron 平台存储
7. **CONFIG** — config.json（只读）
8. **DEFAULT** — 硬编码默认值

### 4.3 功能标志列表（26 个 Feature Flags）

| Flag | LabGroup | 默认值 | 说明 |
|------|----------|--------|------|
| feature_video_rooms | VoiceAndVideo | false | 视频房间 |
| feature_notification_settings2 | Experimental | false | 通知设置 V2 |
| feature_msc3531_hide_messages_pending_moderation | Moderation | false | 等待审核消息隐藏 |
| feature_latex_maths | Messaging | false | LaTeX 数学公式 |
| feature_wysiwyg_composer | Messaging | false | 富文本编辑器 |
| feature_mjolnir | Moderation | false | Mjolnir 反滥用 |
| feature_custom_themes | Themes | false | 自定义主题 |
| feature_exclude_insecure_devices | Encryption | false | 排除不安全设备 |
| feature_html_topic | Rooms | false | HTML 话题 |
| feature_bridge_state | Rooms | false | 桥接状态 |
| feature_jump_to_date | Messaging | false | 跳转到日期 |
| feature_simplified_sliding_sync | Developer | false | 简化滑动同步 |
| feature_element_call_video_rooms | VoiceAndVideo | false | Element Call 视频房间 |
| feature_group_calls | VoiceAndVideo | false | 群组通话 |
| feature_disable_call_per_sender_encryption | VoiceAndVideo | false | 禁用通话逐发送者加密 |
| feature_location_share_live | Messaging | false | 实时位置分享 |
| feature_dynamic_room_predecessors | Rooms | false | 动态房间前任 |
| feature_render_reaction_images | Messaging | false | 渲染回应图片 |
| feature_new_room_list | Ui | **true** | 新房间列表 |
| feature_room_list_sections | Ui | false | 房间列表分区 |
| feature_login_with_qr | Ui | false | QR 登录（仅 config） |
| feature_ask_to_join | Rooms | false | 敲门加入 |
| feature_notifications | Messaging | false | 通知功能 |
| feature_msc4362_encrypted_state_events | Encryption | false | 加密状态事件 |
| feature_user_status | Profile | false | 用户状态 |
| feature_retention | Messaging | false | 消息保留 |

### 4.4 UIFeature 枚举（18 个，config-only）

| UIFeature | 说明 |
|-----------|------|
| AdvancedEncryption | 高级加密设置 |
| URLPreviews | URL 预览 |
| Widgets | Widget 功能 |
| LocationSharing | 位置分享 |
| Voip | 语音/视频通话 |
| Feedback | 反馈功能 |
| Registration | 注册功能 |
| PasswordReset | 密码重置 |
| Deactivate | 账户停用 |
| ShareQRCode | QR 码分享 |
| ShareSocial | 社交分享 |
| IdentityServer | 身份服务器 |
| ThirdPartyID | 第三方 ID |
| AdvancedSettings | 高级设置标签 |
| RoomHistorySettings | 房间历史设置 |
| TimelineEnableRelativeDates | 时间线相对日期 |
| AllowCreatingPublicRooms | 创建公开房间 |
| AllowCreatingPublicSpaces | 创建公开空间 |

### 4.5 设置控制器（20 个）

| 控制器 | 功能 |
|--------|------|
| ReloadOnChangeController | 变更时重新加载应用 |
| ThemeController | 验证主题是否仍被支持 |
| FontSizeController | 字体大小迁移（旧格式→新 delta 格式） |
| SystemFontController | 系统字体/emoji 字体管理 |
| UIFeatureController | 绑定 UI Feature 开关 |
| ReducedMotionController | 尊重 prefers-reduced-motion |
| IncompatibleController | 当另一个设置冲突时禁用/强制值 |
| SlidingSyncController | 滑动同步生命周期管理 |
| ServerSupportUnstableFeatureController | 检查服务器支持的 MSC |
| AnalyticsController | 跟踪设置变更事件 |
| FallbackIceServerController | ICE 服务器回退管理 |
| NotificationsEnabledController | 通知权限管理 |
| NotificationBodyEnabledController | 通知正文启用管理 |
| DeviceIsolationModeController | 设备隔离模式 |
| MediaPreviewConfigController | MSC4278 媒体预览配置 |
| InviteRulesConfigController | MSC4155 邀请规则 |
| BlockInvitesConfigController | MSC4380 阻止邀请 |
| RequiresSettingsController | 依赖其他设置才能启用 |

---

## 5. 交互设计模式

### 5.1 页面布局模式

**三栏布局（默认）**

```
┌─────┬──────────────────────────┬──────────┐
│ 空间  │       主内容区           │ 右侧面板  │
│ 导航  │                          │          │
│ 68px │  min 224px              │ 320px    │
│      │                          │ (可拖拽)  │
└─────┴──────────────────────────┴──────────┘
```

**设计要点**：
- 空间面板可折叠为仅图标模式
- 左侧面板可通过拖拽调整宽度（手柄在右侧）
- 右侧面板可通过拖拽调整宽度（手柄在左侧），最小 320px，最大 50% 窗口
- 不支持的面板可被完全隐藏

### 5.2 房间状态机（RoomView 10 种状态）

```
┌────────┐    ┌──────────┐    ┌────────────┐
│ 正在创建 │ → │ 本地房间  │ → │ 正常聊天视图 │
│ (Local)│    │ (LocalRM) │    │ (Timeline)  │
└────────┘    └──────────┘    └────┬───────┘
                                   │
┌────────┐    ┌──────────┐    ┌───▼────────┐
│ 房间预览 │    │  受邀     │    │ 公开房间预览 │
│ (Link) │    │ (Invite)  │    │ (Viewing)  │
└────────┘    └──────────┘    └────────────┘
                                   │
┌────────┐    ┌──────────┐    ┌───▼────────┐
│ 被踢出  │    │  被禁止   │    │  敲门加入   │
│ (Kicked)│   │ (Banned) │    │   (Knock)  │
└────────┘    └──────────┘    └────────────┘
```

### 5.3 右侧面板状态机（卡片历史栈）

```
[RoomSummary] ← 根卡片
    │
    ├── [MemberList]
    │       └── [MemberInfo]
    │               └── [EncryptionPanel] (if pending verification)
    ├── [ThreadPanel]
    │       └── [ThreadView]
    ├── [PinnedMessages]
    ├── [FilePanel]
    ├── [NotificationPanel]
    └── [Extensions]
            └── [Widget]
```

**行为**：
- `setCard` 替换整个历史栈
- `pushCard` 在栈顶添加，支持返回
- `popCard` 返回上一层
- `togglePanel` 切换面板开/关
- 自动历史生成（如打开 MemberInfo 自动生成 RoomSummary→MemberList→MemberInfo）

### 5.4 认证状态机（App 级）

```
LOADING
  ├── (有 session) → PENDING_CLIENT_START
  │                      ├── COMPLETE_SECURITY (verify device)
  │                      ├── E2E_SETUP (initial setup)
  │                      └── LOGGED_IN → (主界面)
  ├── (无 session) → WELCOME
  │                      ├── LOGIN
  │                      │     └── FORGOT_PASSWORD → LOGIN
  │                      └── REGISTER
  └── (软登出) → SOFT_LOGOUT
                    └── (重认证) → PENDING_CLIENT_START
```

### 5.5 消息时间线设计模式

**分页**：初始 30 条，每次加载 50 条
**连续性**：同作者 5 分钟内消息合并（隐藏头像和名称）
**日期分隔**：跨天时插入日期分隔符
**阅读标记**：垂直线标识最后阅读位置
**已读回执**：每 500ms 发送一次（防抖），显示在消息末尾

### 5.6 拖拽交互模式

| 场景 | 模式 |
|------|------|
| 面板调整 | re-resizable，拖拽手柄母指宽，鼠标进入时显示 |
| 文件上传 | HTML5 Drag and Drop，拖入显示覆盖层 |
| 空间排序 | react-beautiful-dnd，拖拽重新排序 |
| 画中画 | 自定义拖拽+吸附，松开吸附到最近角落 |

### 5.7 确认/警告弹窗模式

| 弹窗类型 | 组件 | 使用场景 |
|----------|------|---------|
| 二选一确认 | QuestionDialog | 大多数确认操作 |
| 文本输入 | TextInputDialog | 需要用户输入时 |
| 错误提示 | ErrorDialog | 操作失败 |
| 信息提示 | InfoDialog | 纯信息展示 |
| 交互式认证 | InteractiveAuthDialog | 需要验证步骤时 |
| 进度显示 | InviteProgressDialog | 批量操作进度 |

### 5.8 键盘导航模式

**地标导航（F6/Shift+F6）**：活动空间按钮 → 房间搜索 → 房间列表 → 消息输入框
**Roving Tab Index**：在表情选择器、工具栏、菜单等组件中使用 WAI-ARIA 模式
**快捷键系统**：50+ 快捷键分 7 类，平台自适应（Mac=⌘, Win=Ctrl）

### 5.9 Toast 通知模式

**分级**：
- **ToastStore**（紧急）：按 priority 排序，来电(100) > 桌面通知提示(30)
- **NonUrgentToastStore**（非紧急）：setup 提醒等

**行为**：addOrReplace（同 key 替换），dismiss（移除），自动过期

### 5.10 乐观更新模式（Local Echo）

**WidgetEchoStore / EchoStore**
- 在操作发起时立即更新本地状态
- 服务端确认后同步
- 失败时回滚到前值
- 使用 LocalEchoWrapper 包装异步 Handler

---

## 6. 认证与安全设计

### 6.1 多种认证方式支持

```
配置发现(.well-known) → 获取登录流程(flows)
  ├── m.login.password → 密码登录表单
  │    标识符: MXID / Email / Phone
  │    验证: 密码 + 可选 CAPTCHA
  ├── m.login.sso / m.login.cas → SSO 跳转
  │    提供商: SAML/OAuth/CAS/自定义
  │    返回: loginToken
  ├── oidcNativeFlow → OIDC Provider
  │    支持: authorization_code grant
  │    注册: prompt=create
  │    Token: 自动刷新
  └── m.login.token → SSO Token 登录
```

### 6.2 加密体系

```
┌──────────────────────┐
│  交叉签名 (Cross-Signing)  │  ← 设备信任链
│  Master / Self / User  Key │
└──────────┬───────────┘
           │
┌──────────▼───────────┐
│  密钥备份 (Key Backup)    │  ← 消息历史恢复
│  4S (Secure Storage)      │
└──────────┬───────────┘
           │
┌──────────▼───────────┐
│  Olm / Megolm            │  ← 消息加密
│  设备间加密 / 房间加密     │
└──────────────────────┘
```

**加密状态指标**：
- 🟢 已验证（交叉签名信任 + 所有设备已验证）
- 🟡 正常（加密但未完全验证）
- 🔴 警告（有不安全设备）
- ⚫ 未加密

### 6.3 会话安全

| 安全机制 | 实现 |
|----------|------|
| Token 加密存储 | localStorage/IndexedDB + pickle key |
| 会话锁 | 多标签页间 IndexedDB 协调 |
| 软登出 | session 失效但保留本地数据 |
| 安全存储（桌面） | Electron safeStorage |
| 脱水设备 | 设备离线时的加密消息解密 |

---

## 7. 消息系统设计

### 7.1 消息发送流程

```
用户输入 → 解析 Markdown/Slash 命令
  → 创建消息内容 { msgtype, body, formatted_body, ... }
  → 附加 @提及 (m.mentions)
  → 附加回复关系 (m.in_reply_to)
  → 附加线程关系 (m.relates_to → m.thread)
  → 加密 (如果房间已加密)
  → MatrixClient.sendMessage()
  → 本地回显 (Local Echo)
  → 服务端确认 → 从 pending 列表中移除
```

### 7.2 消息类型矩阵

| 消息类型 | msgtype | 上传要求 | 渲染组件 | 特殊功能 |
|----------|---------|---------|---------|---------|
| 文本 | m.text | 无 | TextualBodyFactory | Markdown, @提及, 代码块, 剧透 |
| 通知 | m.notice | 无 | TextualBodyFactory | 服务器公告样式 |
| 动作 | m.emote | 无 | TextualBodyFactory | /me 消息 |
| 图片 | m.image | 上传文件 | ImageBodyFactory | 缩略图, 灯箱, 模糊占位 |
| 视频 | m.video | 上传文件 | VideoBodyFactory | 播放器, 缩略海报 |
| 音频 | m.audio | 上传文件 | MAudioBody | 播放器, 时长 |
| 文件 | m.file | 上传文件 | FileBodyFactory | 下载链接, 文件类型图标 |
| 语音 | m.audio | 录制+上传 | MVoiceMessageBody | 波形, 播放队列 |
| 贴纸 | m.sticker | 无 | MStickerBody | 贴纸图片 |
| 投票 | m.poll.start | 无 | MPollBody | 交互式投票 |
| 位置 | m.location | 无 | MLocationBody | MapLibre 地图 |
| 信标 | m.beacon_info | 无 | MBeaconBody | 实时位置共享 |
| 通话 | m.call.* / m.room.rtc_notification | 无 | CallEvent / LegacyCallEvent | 通话状态 |

### 7.3 编辑器键盘快捷键

| 快捷键 | 功能 |
|--------|------|
| Enter | 发送消息 |
| Shift+Enter | 换行 |
| Ctrl/Cmd+B | 粗体 |
| Ctrl/Cmd+I | 斜体 |
| Ctrl/Cmd+Shift+X | 删除线 |
| Ctrl/Cmd+Shift+Q | 引用 |
| Ctrl/Cmd+Shift+L | 链接 |
| Ctrl/Cmd+Shift+C | 代码 |
| Ctrl/Cmd+Z | 撤销 |
| Ctrl/Cmd+Shift+Z | 重做 |
| ↑ (空输入) | 编辑上一条消息 |
| Escape | 取消回复/编辑 |

### 7.4 文件处理管线

```
文件选择/拖拽
  → 大小校验 (服务器上传限制)
  → 上传确认 (可批量确认)
  → MIME 检测 → 分配 msgtype
  → 媒体提取 (图片: 缩略图, 音频: 时长, 视频: 缩略海报)
  → 加密 (加密房间: matrix-encrypt-attachment)
  → 上传 (含进度回调)
  → 发送消息 (含元数据)
```

**缩略图优化**：
- <32KB 跳过缩略图
- 缩略图缩减 <10% 且 >1MB 时跳过
- AVIF/WebP/SVG 始终生成缩略图（兼容性）

---

## 8. 通话系统设计

### 8.1 双轨架构

```
┌─────────────────────────────────────────────┐
│              Call Placement Router          │
│         (src/utils/room/placeCall.ts)       │
└──────────────┬──────────────────────────────┘
               │
     ┌─────────┼─────────┐
     │         │         │
┌────▼────┐ ┌──▼───┐ ┌──▼──────────┐
│ Legacy  │ │Jitsi │ │Element Call │
│ WebRTC  │ │Widget│ │  Widget +   │
│ 1:1     │ │      │ │ MatrixRTC   │
└─────────┘ └──────┘ └─────────────┘
```

### 8.2 通话状态

| 调用阶段 | 系统事件 | UI 显示 |
|----------|---------|---------|
| 拨出 | CallState.Connecting | 回铃音 |
| 来电 | CallEventHandlerEvent.Incoming | IncomingCallToast + 铃声 |
| 接通 | CallState.Connected | VideoFeed/AudioFeed |
| 保持 | RemoteHoldUnhold | 保持画面 |
| 结束 | CallState.Ended | 挂断音效，清除音视频 |

### 8.3 画中画设计

- 触发：离开通话房间
- 尺寸：336×232 px
- 拖拽：自由移动 + 松手吸附最近角落
- 动画：20% lerp 移动，10% lerp 吸附
- 点击：5px 拖拽阈值防止误触

---

## 9. 扩展与定制设计

### 9.1 模块系统（Module API）

**Module 接口**：
```typescript
interface Module { load(): Promise<void>; }
interface ModuleFactory {
    readonly moduleApiVersion: string;
    new (api: Api): Module;
}
```

**API 能力**：
- `config` — 读配置
- `i18n` — 注册翻译
- `navigation` — 导航控制
- `dialog` — 打开对话框
- `profile` — 用户资料（Watchable）
- `customComponents` — 注册自定义组件渲染器
- `customisations` — 注册 UI 组件显隐控制
- `widgetLifecycle` — Widget 权限审批
- `widget` — Widget 操作
- `composer` — 输入框操作
- `overwriteAccountAuth` — 替换凭证
- `createRoot` — 创建 React 18 Root

### 9.2 内建模块

| 模块 | 功能 |
|------|------|
| **widget-lifecycle** | 按 URL 模式自动批准 Widget 权限 |
| **widget-toggles** | 房间头部 Widget 切换按钮 |
| **restricted-guests** | 访客限制（隐藏 UI + 注册引导） |
| **banner** | 品牌导航横幅 |

### 9.3 传统定制化点（10 个）

| 定制化点 | 接口 |
|----------|------|
| ComponentVisibility | 控制 7 个 UI 组件显隐 |
| Media | 覆盖媒体对象创建 |
| Lifecycle | 登出/清除钩子 |
| RoomList | 过滤可见房间 |
| UserIdentifier | 自定义用户 ID 显示 |
| Alias | 别名偏好 |
| Directory | 房间发布额外限制 |
| WidgetPermissions | 预批准 Widget 能力 |
| WidgetVariables | 自定义 Widget URL 变量 |
| ChatExport | 强制导出参数 |

---

## 附录

### A. Hook 快速索引（54 个）

| 类别 | Hook |
|------|------|
| **房间** | useRoomName, useRoomState, useTopic, useIsEncrypted, useRoomMembers, useRoomMemberCount, useMyRoomMembership, useRoomIdName, useGuestAccessInformation |
| **通知** | useUnreadNotifications, useRoomThreadNotifications, useGlobalNotificationState, useRoomNotificationState, useNotificationSettings |
| **通话** | useCall, useConnectionState, useParticipantCount, useParticipatingMembers, useRoomCall |
| **搜索** | usePublicRoomDirectory, useUserDirectory, useSpaceResults, useRecentSearches |
| **资料** | useProfileInfo, useRoomMemberProfile, useUserStatus, useUserTimezone, useAccountData, useThreepids, usePushers |
| **设置** | useSettingValue, useSettingValueAt, useFeatureEnabled, useTheme, useMediaVisible |
| **加密** | useEncryptionStatus, useKeyBackupStatus, useHasOtherVerifiedDevices |
| **基础设施** | useEventEmitter, useDispatcher, useAsyncMemo, useAsyncRefreshMemo, useLatestResult, useLocalStorageState, useLocalEcho, useTimeout, useInterval, useAnimation, useHover, useFocus |
| **Pin** | usePinnedEvents, useReadPinnedEvents, useFetchedPinnedEvents, useSortedFetchedPinnedEvents |
| **Permalink** | usePermalink, usePermalinkTargetRoom, usePermalinkEvent, usePermalinkMember |
| **其他** | useWindowWidth, useDownloadMedia, useIsReleaseAnnouncementOpen, useCurrentPhase, useDebouncedCallback |

### B. 斜杠命令速查（40+ 条）

| 类别 | 命令 |
|------|------|
| **消息** | /spoiler, /shrug, /tableflip, /unflip, /lenny, /plain, /html, /rainbow, /rainbowme, /me |
| **操作** | /join, /goto, /invite, /part, /nick, /myroomnick, /msg, /query, /ignore, /unignore, /status |
| **管理** | /topic, /roomname, /roomavatar, /myroomavatar, /myavatar, /op, /deop, /remove, /ban, /unban, /addwidget, /upgraderoom, /converttodm, /converttoroom |
| **高级** | /devtools, /rageshake, /whois, /jumptodate, /verify, /discardsession, /holdcall, /unholdcall, /help |
| **特效** | /fireworks, /snowfall, /rain, /spaceinvaders, /hearts, /confetti |

### C. URL Hash 路由表

| Hash | 视图 |
|------|------|
| `#/welcome` | 欢迎页 |
| `#/login` | 登录 |
| `#/register` | 注册 |
| `#/forgot_password` | 忘记密码 |
| `#/soft_logout` | 软登出 |
| `#/qr_login` | QR 登录 |
| `#/home` | 首页 |
| `#/room/:roomId` | 房间视图 |
| `#/room/:roomId/:eventId` | 定位到消息 |
| `#/user/:userId` | 用户视图 |
