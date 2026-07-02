# 喜多（HITO）用户手册生成计划 — 并行执行版

> 基于 `hito-mp` 前端代码库全量分析，按模块拆分为可并行执行的子任务。
> 更新日期：2026-07-02

---

## 执行模型

采用 **Wave 分波 + Subagent 并行** 模式。每个 Wave 内的所有任务可同时启动，不同 Wave 之间有依赖关系。

```
Wave 0 (前置)  ──→  Wave 1 (P0×9 并行)  ──→  Wave 2 (P1×8 并行)  ──→  Wave 3 (P2×3 并行)
```

- **Wave 0**：生成核心概念共享文档，所有后续手册引用此文档。
- **Wave 1**：P0 核心模块，9 个 subagent 并行执行。
- **Wave 2**：P1 重要模块，8 个 subagent 并行执行（可与 Wave 1 尾部重叠）。
- **Wave 3**：P2 辅助模块，3 个 subagent 并行执行。

---

## 公共信息

### 代码仓库路径

| 仓库 | 本地路径 |
|------|----------|
| 前端 `hito-mp` | `/Users/archcst/broot/INNOSYNC/code/hito/hito-mp` |
| 后端 `hito` | 通过 `HITO_DIR` 环境变量获取 |

### 所有 Subagent 统一指令

每个 subagent 执行时必须：

1. 加载 `hito-codebase-reader` skill 阅读前后端代码。
2. 加载 `hito-version-getter` skill 获取版本号。
3. 按 `hito-manual-generator` skill 的文档结构输出。
4. 输出文件保存到 `manual/{功能英文名}.md`。
5. 完成后执行自检清单（见文末）。

---

## Wave 0：核心概念共享文档

**任务名**：`core-concepts`
**输出文件**：`manual/core-concepts.md`
**依赖**：无
**并行度**：1（单任务）

#### 代码依据

- 前端：`pages/index.mpx`、`pages/crowd.mpx`、`pages/event/`、`pages/personal/my-home.mpx`
- 后端：租户/社团/活动核心 Model 和 Service

#### 手册内容

| 章节 | 说明 |
|------|------|
| SaaS 多租户架构 | 社区（Tenant）→ 社团（Crowd）→ 活动（CIE/CE）三层关系 |
| 角色权限体系 | GA / TA / CO / CA 四级角色定义与权限边界 |
| 活动分类 | CIE（打卡活动）与 CE（历史活动）的区别 |
| 术语表 | 所有核心概念的中英文对照和简要说明 |

#### 交付物

- `manual/core-concepts.md`

---

## Wave 1：P0 核心模块（9 个并行任务）

> 所有任务同时启动，每个 subagent 独立完成一个模块的手册。

---

### 任务 1.1：社团管理

| 字段 | 值 |
|------|-----|
| 任务名 | `crowd-management` |
| 输出文件 | `manual/crowd-management.md` |
| 涉及角色 | TA / CO / 普通用户 |
| 前端页面 | `pages/crowd.mpx`、`pages/owner/apply-crowd.mpx`、`pages/owner/my-crowd-apply.mpx`、`pages/hito/components/crowd-menu.mpx`、`pages/hito/components/crowd-info.mpx` |
| API 文件 | `taApi.ts`（`createCrowd`/`updateCrowdInfo`/`deleteCrowd`/`queryCrowds`/`modifyCrowdCertified`/`modifyCrowdActivityCharge`）、`coApi.ts`（`coCrowdModify`/`coCrowdAvatarModify`/`coCrowdBannerModify`/`coCrowdBannerDelete`/`coTransferCrowd`/`coDeleteCrowd`）、`commonApi.ts`（`createCrowdApply`/`toggleCrowdFollow`/`queryMyCrowdApply`）、`mineApi.ts`（`createCrowd`） |
| 操作入口根路径 | `/pages/crowd` |

#### 需覆盖的功能场景

1. 创建社团（TA 创建 / 用户申请创建）
2. 编辑社团资料（名称、简称、口号、成立日期、Logo、Banner）
3. 删除社团
4. 社团认证（认证标识开关）
5. 活动收费设置
6. 转让社团主理人
7. 关注/取关社团
8. 社团申请审核（TA 视角）
9. 社团浏览与发现

---

### 任务 1.2：成员管理

| 字段 | 值 |
|------|-----|
| 任务名 | `member-management` |
| 输出文件 | `manual/member-management.md` |
| 涉及角色 | CA / CO / 普通用户 |
| 前端页面 | `pages/owner/audit-members.mpx`、`pages/owner/create-members.mpx`、`pages/owner/components/batch-create-member.mpx`、`pages/owner/components/apply-create-member.mpx`、`pages/owner/components/member-info.mpx`、`pages/hito/components/hito-member-new.mpx`、`pages/hito/components/hito-member-select.mpx` |
| API 文件 | `coApi.ts`（`coDeleteMember`/`coModifyMemberInfo`/`coMergeVirtualMember`/`coPassApplyCrowdAdmin`）、`caApi.ts`（`caCreateOneMember`/`caCreateMember`/`caModifyMemberInfo`/`caMemberAvatarUpload`/`caUnbindMember`/`caPassApplyCrowd`/`caRejectApplyCrowd`/`caAcceptAndLinkMember`/`caQueryMemberApplyList`）、`queryApi.ts`（`caQueryMembers`）、`mineApi.ts`（`bindCrowdInvitation`/`createApplyCrowd`/`queryMemberInfo`/`memberAvatarUpload`） |
| 操作入口根路径 | `/pages/crowd` |

#### 需覆盖的功能场景

1. 批量导入成员
2. 单个创建成员
3. 编辑成员资料
4. 删除成员
5. 解绑成员
6. 合并虚拟成员
7. 入群申请审核（通过/拒绝/设为管理员/接受并链接）
8. 成员列表查看
9. 用户申请加入社团
10. 用户查看申请状态
11. 管理员邀请用户成为管理员

---

### 任务 1.3：活动管理

| 字段 | 值 |
|------|-----|
| 任务名 | `event-management` |
| 输出文件 | `manual/event-management.md` |
| 涉及角色 | CA / CO / TA |
| 前端页面 | `pages/event/create-event.mpx`、`pages/event/event-member.mpx`、`pages/event/event-draft.mpx`、`pages/event/event-attachment.mpx`、`pages/event/event-file-download.mpx`、`pages/event/components/create-event-base.mpx`、`pages/event/components/create-event-clock.mpx`、`pages/event/components/create-event-member.mpx` |
| API 文件 | `caApi.ts`（`caCreateEvent`/`caModifyEvent`/`caDeleteEvent`/`caCreateHito`/`caEditHito`/`caDeleteCeEvent`/`caManageHitoMembers`/`caQueryEventParticipants`/`caManageEventParticipant`）、`mineApi.ts`（`queryEventDetail`） |
| 操作入口根路径 | `/pages/crowd` |

#### 需覆盖的功能场景

1. 创建活动（统一入口：基本信息 + 选择成员 + 打卡设置）
2. 编辑活动
3. 删除活动
4. 活动草稿箱
5. 活动参与人管理
6. 活动附件管理（上传、可见范围）
7. 活动附件下载
8. 补充历史活动（CE）
9. 编辑历史活动（CE）
10. 删除历史活动（CE）
11. 历史活动详情查看
12. 活动审核流程

---

### 任务 1.4：打卡系统

| 字段 | 值 |
|------|-----|
| 任务名 | `clock-in-system` |
| 输出文件 | `manual/clock-in-system.md` |
| 涉及角色 | CA / CO / 普通用户 |
| 前端页面 | `pages/clockin/clock-in.mpx`、`pages/clockin/clock-in-detail.mpx`、`pages/clockin/clock-in-records.mpx`、`pages/clockin/clock-in-extra-edit.mpx`、`pages/clockin/components/clock-in-detail-info.mpx`、`pages/clockin/components/clock-in-detail-records.mpx`、`pages/clockin/components/onsite-clock-in-dialog.mpx`、`pages/clockin/components/payment-dialog.mpx` |
| API 文件 | `caApi.ts`（`caCreateClockIn`/`caEditClockIn`/`caDeleteClockIn`/`caQueryClockInRecord`/`caExportClockInExcel`/`caGenerateOnsiteClockInQrcode`/`caScanUserCode`/`caToggleClockOut`/`caEditExtraPic`）、`mineApi.ts`（`mineClockIn`/`mineClockOut`/`mineClockInSignUp`/`cancelClockInSignUp`/`queryClockInRecord`） |
| 操作入口根路径 | `/pages/clockin/clock-in-detail` |

#### 需覆盖的功能场景

1. 创建打卡活动（时间、签到方式、地理围栏、报名）
2. 编辑打卡活动设置
3. 用户打卡签到（普通打卡/现场亮码/扫码用户码）
4. 用户签退
5. 活动报名（时段、名额）
6. 取消报名
7. 地理位置打卡（围栏校验）
8. NFC 打卡
9. 现场亮码签到
10. 管理员扫用户码签到
11. 打卡记录查看与管理
12. 导出打卡记录 Excel
13. 打卡成员审核
14. 补充图片编辑
15. 手动启停签退
16. 打卡详情页

---

### 任务 1.5：渠道码系统

| 字段 | 值 |
|------|-----|
| 任务名 | `channel-code` |
| 输出文件 | `manual/channel-code.md` |
| 涉及角色 | CA / CO / 普通用户 |
| 前端页面 | `qrcode-pkg/pages/event-channel-code.mpx`、`qrcode-pkg/pages/event-channel-code-detail.mpx`、`qrcode-pkg/pages/channel-manage.mpx` |
| API 文件 | `caApi.ts`（`createChannel`/`modifyChannel`/`queryChannelList`/`createChannelCode`/`queryChannelCodes`/`queryChannelCodeDetail`/`queryChannelCodeScanRecords`）、`coApi.ts`（`createChannel`/`modifyChannel`/`deleteChannel`/`toggleChannelStatus`/`queryChannelList`）、`mineApi.ts`（`queryChannelCodeInfo`） |
| 操作入口根路径 | `/pages/clockin/clock-in-detail` |

#### 需覆盖的功能场景

1. 创建渠道
2. 编辑渠道
3. 删除渠道
4. 启用/停用渠道
5. 渠道列表查看
6. 生成活动渠道码
7. 渠道码详情（浏览量、扫码去重人数）
8. 渠道码扫码记录
9. 用户端渠道码解析

---

### 任务 1.6：问卷系统

| 字段 | 值 |
|------|-----|
| 任务名 | `survey` |
| 输出文件 | `manual/survey.md` |
| 涉及角色 | CA / CO / 普通用户 |
| 前端页面 | `survey-pkg/pages/survey-admin.mpx`、`survey-pkg/pages/survey-submissions.mpx`、`survey-pkg/pages/survey-user-test.mpx` |
| API 文件 | `caApi.ts`（`createSurvey`/`disableSurvey`/`enableSurvey`/`addSurveyQuestion`/`disableSurveyQuestion`/`sortSurveyQuestions`/`querySurveyDetail`/`querySurveyDetailByEvent`/`querySurveySubmissions`/`exportSurveyExcel`）、`mineApi.ts`（`submitSurvey`/`editSurveySubmission`/`querySurveyDetail`/`queryMySurveySubmission`） |
| 操作入口根路径 | `/pages/clockin/clock-in-detail` |

#### 需覆盖的功能场景

1. 创建问卷（关联活动 + 初始题目）
2. 添加/编辑题目（文本/多行文本/单选/多选）
3. 题目排序
4. 启用/停用问卷
5. 启用/停用题目
6. 用户填写问卷
7. 用户编辑已提交的问卷
8. 管理员查看提交记录
9. 导出问卷结果 Excel

---

### 任务 1.7：评论系统

| 字段 | 值 |
|------|-----|
| 任务名 | `comment` |
| 输出文件 | `manual/comment.md` |
| 涉及角色 | 所有用户 / 评论作者 / 管理员 |
| 前端页面 | `pages/comment/comment-test.mpx`、`pages/comment/components/comment-input.mpx`、`pages/comment/components/comment-item.mpx`、`pages/comment/components/comment-list.mpx`、`pages/comment/components/comment-thread.mpx` |
| API 文件 | `commonApi.ts`（`publishEventComment`/`deleteEventComment`/`queryEventCommentThreads`/`queryEventCommentReplies`/`toggleEventCommentLike`/`commentPhotoUpload`）、`caApi.ts`（`caToggleEventComment`/`caSelectPicFromComment`） |
| 操作入口根路径 | `/pages/hito/ce-detail` 或 `/pages/clockin/clock-in-detail` |

#### 需覆盖的功能场景

1. 发布评论（文字 + 图片）
2. 回复评论（评论树结构）
3. 查看评论（分页加载）
4. 删除评论
5. 点赞评论
6. 管理员开关评论区
7. 管理员精选评论图片

---

### 任务 1.8：个人中心

| 字段 | 值 |
|------|-----|
| 任务名 | `my-home` |
| 输出文件 | `manual/my-home.md` |
| 涉及角色 | 普通用户 / CA / CO |
| 前端页面 | `pages/personal/my-home.mpx`、`pages/personal/member-profile.mpx`、`pages/personal/joined-crowds.mpx`、`pages/personal/my-event.mpx`、`pages/personal/persona-detail.mpx`、`pages/personal/components/personal-profile.mpx`、`pages/personal/components/my-managing-crowds.mpx`、`pages/personal/components/my-managing-events.mpx`、`pages/personal/components/my-participation-timeline.mpx`、`pages/personal/components/user-qrcode-dialog.mpx`、`pages/clockin/components/create-persona.mpx`、`pages/clockin/components/modify-persona.mpx`、`pages/clockin/components/persona-card.mpx` |
| API 文件 | `mineApi.ts`（`queryMineSummary`/`queryMineTimeline`/`queryMineParticipation`/`queryMineCreatedEvents`/`generateUserCode`/`avatarRenewUpload`/`updateNickname`/`createPersona`/`modifyPersona`/`queryPersona`/`uploadPersonaAvatar`） |
| 操作入口根路径 | `/pages/personal/my-home` |

#### 需覆盖的功能场景

1. 个人首页（数据概览 + 管理的社团/活动 + 参与时间线）
2. 编辑个人资料（昵称、头像、手机号）
3. 我的参与时间线
4. 我管理的社团
5. 我管理的活动
6. 我已加入的社团
7. 我的活动（审核状态）
8. 我的成员资料查看/编辑
9. 个人二维码（管理员扫码签到用）

---

### 任务 1.9：首页与发现

| 字段 | 值 |
|------|-----|
| 任务名 | `home-discover` |
| 输出文件 | `manual/home-discover.md` |
| 涉及角色 | 所有用户 |
| 前端页面 | `pages/index.mpx`、`pages/tab/home-tab.mpx`、`pages/tab/mine-tab.mpx`、`pages/hito/components/my-subscribe.mpx`、`pages/hito/components/hot-crowds.mpx`、`pages/hito/components/my-hito-timeline.mpx`、`pages/hito/components/lg-activity-heatmap.mpx`、`pages/hito/components/index-advertisement.mpx` |
| API 文件 | `mineApi.ts`（`queryMineHeatmap`/`queryTenantHeatmap`/`queryMineFollowCrowdEvents`/`queryTenantTimeline`） |
| 操作入口根路径 | `/pages/index` |

#### 需覆盖的功能场景

1. 社区首页（动态时间线 + 活动热力图 + 关注社团 + 热门社团）
2. 个人活动热力图（年度参与日历）
3. 关注社团的最新活动
4. 热门社团推荐
5. 个人首页时间线
6. 社团首页时间线
7. 认证社团浏览

---

## Wave 2：P1 重要模块（8 个并行任务）

> 可在 Wave 1 完成大部分后启动，或与 Wave 1 尾部重叠执行。

---

### 任务 2.1：抽奖系统

| 字段 | 值 |
|------|-----|
| 任务名 | `lottery` |
| 输出文件 | `manual/lottery.md` |
| 涉及角色 | CA / CO / 抽奖管理员 / 普通用户 |
| 前端页面 | `lottery-pkg/pages/lottery-manage.mpx`、`lottery-pkg/pages/lottery-captcha.mpx`、`lottery-pkg/pages/lottery-history.mpx`、`lottery-pkg/pages/lottery-draw.mpx` |
| API 文件 | `lottery.ts`（`createLottery`/`createLotteryRound`/`modifyLotteryRound`/`deleteLotteryRound`/`prepareLotteryRound`/`startLotteryRound`/`drawLotteryRound`/`joinLotteryTopic`/`leaveLotteryTopic`/`lotteryHistory`/`verifyLotteryCode`/`changeLotteryAdmin`/`screenScanAuthorize`） |
| 操作入口根路径 | `/pages/hito/ce-detail` 或 `/pages/clockin/clock-in-detail` |

#### 需覆盖的功能场景

1. 创建抽奖（关联活动）
2. 创建/编辑/删除抽奖轮次
3. 开始抽奖（准备 → 开始 → 出结果）
4. 用户参与抽奖
5. 抽奖历史查看
6. 大屏授权码扫码
7. 验证抽奖码
8. 转移抽奖管理员

---

### 任务 2.2：支付与收费

| 字段 | 值 |
|------|-----|
| 任务名 | `payment` |
| 输出文件 | `manual/payment.md` |
| 涉及角色 | 普通用户 / CO / CA |
| 前端页面 | `pages/payment/payment-test.mpx`、`pages/clockin/components/payment-dialog.mpx` |
| API 文件 | `userApi.ts`（`payCieOrder`/`queryPayCieOrder`）、`coApi.ts`（`coSetVip`/`coSetSingleOrderFree`）、`taApi.ts`（`modifyCrowdActivityCharge`） |
| 操作入口根路径 | `/pages/clockin/clock-in-detail` |

#### 需覆盖的功能场景

1. 活动报名付费（微信支付）
2. 支付订单查询
3. 设置会员（VIP）
4. 设置单笔订单免费
5. VIP 免费参与活动

---

### 任务 2.3：分享传播

| 字段 | 值 |
|------|-----|
| 任务名 | `share` |
| 输出文件 | `manual/share.md` |
| 涉及角色 | 所有用户 / CA / CO / TA |
| 前端页面 | `pages/owner/share-crowd.mpx`、`pages/owner/components/share-crowd-canvas.mpx`、`pages/owner/components/share-crowd-canvas2.mpx`、`pages/clockin/components/share-event.mpx` |
| API 文件 | `userApi.ts`（`generateShareCode`/`queryShareCode`/`getShareQrcode`）、`caApi.ts`（`caQueryShareStats`/`caQueryShareStatsDetail`/`caQueryInvitedRecords`） |
| 操作入口根路径 | `/pages/clockin/clock-in-detail` |

#### 需覆盖的功能场景

1. 生成活动分享码
2. 分享活动到微信
3. 分享统计（分享者维度 + 受邀记录）
4. 管理员分享统计页面
5. 邀请管理员页面

---

### 任务 2.4：AI 辅助功能

| 字段 | 值 |
|------|-----|
| 任务名 | `ai-features` |
| 输出文件 | `manual/ai-features.md` |
| 涉及角色 | CA / CO / TA |
| 前端页面 | `pages/admin/prompt-manage.mpx` |
| API 文件 | `caApi.ts`（`caGenCeAiDescription`/`caGenCieAiDescription`）、`taApi.ts`（`savePrompt`/`queryPrompts`/`activePrompt`/`testPromptCie`/`testPromptCe`） |
| 操作入口根路径 | `/pages/crowd` |

#### 需覆盖的功能场景

1. AI 生成活动描述（CE 历史活动）
2. AI 生成活动描述（CIE 打卡活动，支持海报识别）
3. 提示词管理（TA 视角：创建/保存/查询/激活/测试）

---

### 任务 2.5：聊天系统

| 字段 | 值 |
|------|-----|
| 任务名 | `chat` |
| 输出文件 | `manual/chat.md` |
| 涉及角色 | 所有用户 |
| 前端页面 | `chat-pkg/pages/chat.mpx`、`chat-pkg/pages/conversation.mpx` |
| API 文件 | `chat.ts`（`chatConversation`/`chatConversations`/`chatHistory`/`publishChat`/`readChat`/`getConversationId`）、`userApi.ts`（`uploadChatImage`/`queryChatImage`） |
| 操作入口根路径 | `/pages/index` |

#### 需覆盖的功能场景

1. 会话列表
2. 一对一聊天（文字 + 图片）
3. 消息已读/未读

---

### 任务 2.6：TA 管理后台

| 字段 | 值 |
|------|-----|
| 任务名 | `admin-ta` |
| 输出文件 | `manual/admin-ta.md` |
| 涉及角色 | TA |
| 前端页面 | `pages/admin/admin-index.mpx`、`pages/admin/create-crowd.mpx`、`pages/admin/query-crowd-new.mpx`、`pages/admin/crowd-info-edit.mpx`、`pages/admin/crowd-delete.mpx`、`pages/admin/crowd-manage.mpx`、`pages/admin/crowd-apply-list.mpx`、`pages/admin/crowd-apply-detail.mpx`、`pages/admin/event-manage.mpx`、`pages/admin/audit-event.mpx`、`pages/admin/audit-event-detail.mpx`、`pages/admin/tenant-data-analysis.mpx`、`pages/admin/tenant-config.mpx`、`pages/admin/event-insight-analysis.mpx`、`pages/admin/share-statistics.mpx`、`pages/admin/share-statistics-detail.mpx`、`pages/admin/prompt-manage.mpx` |
| API 文件 | `taApi.ts` 全量 |
| 操作入口根路径 | `/pages/admin/admin-index` |

#### 需覆盖的功能场景

1. 创建社团
2. 社团查询
3. 社团管理（编辑/删除）
4. 社团申请审核
5. 事件管理
6. 活动审核
7. 租户配置
8. 租户数据分析
9. 活动数据分析
10. 分享统计
11. 提示词管理

---

### 任务 2.7：反馈系统

| 字段 | 值 |
|------|-----|
| 任务名 | `feedback` |
| 输出文件 | `manual/feedback.md` |
| 涉及角色 | 普通用户 |
| 前端页面 | `pages/personal/feedback.mpx` |
| API 文件 | `commonApi.ts`（`publishFeedback`/`uploadFeedbackPic`） |
| 操作入口根路径 | `/pages/personal/my-home` |

#### 需覆盖的功能场景

1. 用户提交反馈（标题+描述+图片）

---

### 任务 2.8：活动审核流程（独立成册）

| 字段 | 值 |
|------|-----|
| 任务名 | `event-audit` |
| 输出文件 | `manual/event-audit.md` |
| 涉及角色 | TA / CO / CA |
| 前端页面 | `pages/admin/audit-event.mpx`、`pages/admin/audit-event-detail.mpx` |
| API 文件 | `taApi.ts`（`queryAuditEventList`/`queryEventAuditDetail`/`auditEvent`）、`caApi.ts`（`queryAuditEventList`/`queryEventAuditDetail`）、`mineApi.ts`（`queryAuditEventList`） |
| 操作入口根路径 | `/pages/admin/admin-index` |

#### 需覆盖的功能场景

1. TA 审核活动（列表 → 详情 → 通过/拒绝）
2. CA/CO 查看自己提交的审核记录
3. 审核状态流转说明
4. 审核开关配置（`eventNeedAudit`）

---

## Wave 3：P2 辅助模块（3 个并行任务）

---

### 任务 3.1：NFC 功能

| 字段 | 值 |
|------|-----|
| 任务名 | `nfc` |
| 输出文件 | `manual/nfc.md` |
| 涉及角色 | TA / CA / CO / 普通用户 |
| 前端页面 | 无独立页面，嵌入其他页面 |
| API 文件 | `taApi.ts`（`createNfc`/`queryNfcScheme`/`queryNfcQrcode`）、`queryApi.ts`（`queryNfcRedirect`）、`caApi.ts`（`caToggleNfcEvent`） |
| 操作入口根路径 | `/pages/admin/admin-index`（TA）/ `/pages/clockin/clock-in-detail`（CA） |

#### 需覆盖的功能场景

1. TA 创建 NFC Scheme
2. NFC 跳转（用户触碰 → 跳转社团页）
3. 活动绑定/解绑 NFC

---

### 任务 3.2：邀请流程

| 字段 | 值 |
|------|-----|
| 任务名 | `invite-flow` |
| 输出文件 | `manual/invite-flow.md` |
| 涉及角色 | CA / 被邀请用户 |
| 前端页面 | `pages/invite/apply-invite.mpx`、`pages/invite/apply-invite-admin.mpx`、`pages/invite/apply-invite-one.mpx`、`pages/invite/applying.mpx`、`pages/invite/clock-in-applying.mpx`、`pages/invite/hito-fwh.mpx` |
| API 文件 | `caApi.ts`（`caCreateSingleInvite`/`caConsumeCrowdAdminInvitation`）、`queryApi.ts`（`consumeSingleInvite`/`querySingleInvite`）、`commonApi.ts`（`getFwhQrcode`） |
| 操作入口根路径 | `/pages/crowd` |

#### 需覆盖的功能场景

1. 单发邀请（创建邀请 → 扫码/链接消费）
2. 管理员邀请码
3. 服务号引导关注

---

### 任务 3.3：租户配置

| 字段 | 值 |
|------|-----|
| 任务名 | `tenant-config` |
| 输出文件 | `manual/tenant-config.md` |
| 涉及角色 | TA |
| 前端页面 | `pages/admin/tenant-config.mpx` |
| API 文件 | `taApi.ts`（`modifyMiniConfig`/`queryMiniConfig`/`uploadMiniConfigPic`）、`commonApi.ts`（`queryMiniConfig`） |
| 操作入口根路径 | `/pages/admin/admin-index` |

#### 需覆盖的功能场景

1. 小程序主题配置（主色/副色/品牌Logo）
2. 社团类型名称自定义
3. 活动是否需审核开关

---

## 执行调度伪代码

```
# Wave 0：必须先完成
run_subagent("core-concepts")

# Wave 1：9 个任务并行
parallel:
  run_subagent("crowd-management")
  run_subagent("member-management")
  run_subagent("event-management")
  run_subagent("clock-in-system")
  run_subagent("channel-code")
  run_subagent("survey")
  run_subagent("comment")
  run_subagent("my-home")
  run_subagent("home-discover")

# Wave 2：8 个任务并行（可与 Wave 1 尾部重叠）
parallel:
  run_subagent("lottery")
  run_subagent("payment")
  run_subagent("share")
  run_subagent("ai-features")
  run_subagent("chat")
  run_subagent("admin-ta")
  run_subagent("feedback")
  run_subagent("event-audit")

# Wave 3：3 个任务并行
parallel:
  run_subagent("nfc")
  run_subagent("invite-flow")
  run_subagent("tenant-config")
```

---

## Subagent Prompt 模板

每个 subagent 启动时注入以下 prompt：

```
你是喜多（HITO）用户手册编写专家。请为以下模块生成用户手册：

## 任务信息
- 模块名：{task_name}
- 输出文件：manual/{task_name}.md
- 涉及角色：{roles}
- 操作入口根路径：{root_path}

## 前端代码路径
{frontend_files}

## API 文件
{api_files}

## 需覆盖的功能场景
{scenarios}

## 执行步骤
1. 加载 hito-codebase-reader skill，阅读上述前端页面和 API 文件
2. 加载 hito-version-getter skill，获取当前前后端版本号
3. 按 hito-manual-generator skill 的文档结构编写手册
4. 保存到 manual/{task_name}.md
5. 执行自检清单

## 自检清单
- [ ] 文档面向终端用户，无技术语言
- [ ] 每个操作步骤有代码依据
- [ ] 角色权限表与后端鉴权逻辑一致
- [ ] 覆盖所有角色视角和边界状态
- [ ] Front Matter 字段完整（title/updated_at/version_fe/version_be）
- [ ] 每个 ### 场景包含：角色权限、操作入口、操作流程、限制与约束、常见问题
```

---

## 交付物清单

| Wave | 文件 | 状态 |
|------|------|------|
| 0 | `manual/core-concepts.md` | ⬜ |
| 1 | `manual/crowd-management.md` | ⬜ |
| 1 | `manual/member-management.md` | ⬜ |
| 1 | `manual/event-management.md` | ⬜ |
| 1 | `manual/clock-in-system.md` | ⬜ |
| 1 | `manual/channel-code.md` | ⬜ |
| 1 | `manual/survey.md` | ⬜ |
| 1 | `manual/comment.md` | ⬜ |
| 1 | `manual/my-home.md` | ⬜ |
| 1 | `manual/home-discover.md` | ⬜ |
| 2 | `manual/lottery.md` | ⬜ |
| 2 | `manual/payment.md` | ⬜ |
| 2 | `manual/share.md` | ⬜ |
| 2 | `manual/ai-features.md` | ⬜ |
| 2 | `manual/chat.md` | ⬜ |
| 2 | `manual/admin-ta.md` | ⬜ |
| 2 | `manual/feedback.md` | ⬜ |
| 2 | `manual/event-audit.md` | ⬜ |
| 3 | `manual/nfc.md` | ⬜ |
| 3 | `manual/invite-flow.md` | ⬜ |
| 3 | `manual/tenant-config.md` | ⬜ |

**共计 21 个手册文件**。
