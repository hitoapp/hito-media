---
name: user-manual-generator
description: 根据前后端代码库，为某个功能自动生成用户手册。使用 codegraph CLI 分析前端页面和数据模型，追踪后端 API 和业务逻辑，最终输出结构化的 .md 文档，支持 LangChain RAG chunking 后向量化落库。适用于"生成用户手册""写功能说明文档""为 XX 功能写操作指南"等请求。
---

# user-manual-generator

## 目标

基于前后端代码库，为指定功能自动生成结构化的用户手册（.md 格式），用于支持 LangChain RAG chunking 后向量化落库。

## 强制约束

- 必须完整分析前后端相关代码后，才能开始编写用户手册。
- 生成的文档必须遵循固定的六章节结构（见下方"文档结构"）。
- 文档保存到项目根目录的 `manual/` 目录下。
- 不要编造代码中不存在的功能。所有描述必须有代码依据。
- 如果代码中没有发现与用户描述匹配的功能，明确告知用户并停止。

## 工作流程

### 第一步：加载环境配置

从项目根目录 `.env` 文件中读取代码库路径：

```sh
cat .env
```

必须包含以下两个变量：

| 变量 | 说明 |
|------|------|
| `HITO_MP_DIR` | 前端小程序代码库路径 |
| `HITO_DIR` | 后端 Java/Kotlin 代码库路径 |

如果 `.env` 文件不存在或变量缺失，提示用户配置后再继续。

### 第二步：确认目标功能

询问用户要生成哪个功能的用户手册。用户可能使用以下方式描述：

- 功能中文名（如"渠道码"）
- 功能英文名（如"channel code"）
- 代码中的关键词（如"channel"、"qrcode"）
- 页面路径（如"qrcode-pkg"）

如果用户描述不够明确，先搜索可能的候选功能列表让用户确认。

### 第三步：初始化 CodeGraph

检查前后端代码库中是否已存在 `.codegraph` 目录：

```sh
ls -d $HITO_MP_DIR/.codegraph 2>/dev/null && echo "前端: OK" || echo "前端: 未初始化"
ls -d $HITO_DIR/.codegraph 2>/dev/null && echo "后端: OK" || echo "后端: 未初始化"
```

如果未初始化，执行：

```sh
codegraph init $HITO_MP_DIR
codegraph init $HITO_DIR
```

如果已初始化但索引可能过期，先 sync：

```sh
codegraph sync $HITO_MP_DIR
codegraph sync $HITO_DIR
```

### 第四步：前端代码分析

#### 4.1 搜索前端页面

使用多种方式定位前端相关代码：

```sh
# 通过 codegraph 搜索符号
codegraph query "<关键词>" --path $HITO_MP_DIR -l 20 -j

# 通过 grep 搜索中文/英文关键词
rg -il "<中文关键词>" $HITO_MP_DIR/src --include '*.vue' --include '*.ts' --include '*.mpx' --include '*.js'

# 列出前端文件结构
codegraph files --path $HITO_MP_DIR
```

#### 4.2 读取前端页面文件

找到相关页面后，完整读取每个页面的 `.mpx` / `.vue` 文件。关注：

- 页面标题和导航栏文案 → 用于确定功能在 UI 中的名称
- 数据加载逻辑（onLoad / fetchData 等方法） → 用于了解功能的输入条件和加载流程
- 用户交互方法（tap、submit 等） → 用于编写操作步骤
- 条件分支（if/else、disabled、角色判断） → 用于编写权限和约束说明
- 提示文案（toast、modal、empty state） → 用于了解边界情况和错误提示

#### 4.3 读取数据模型

读取 `types.ts` 或类似的类型定义文件，提取功能的 ViewModel / DTO 结构。关注：

- 字段名称和类型
- 枚举值
- 接口之间的关系

#### 4.4 提取 API 调用

从前端 API 文件（如 `caApi.ts`、`coApi.ts`）中提取与功能相关的 API 调用，记录：

- API 路径（如 `/ca/cmd/channel/create`）
- HTTP 方法（GET/POST/DELETE）
- 请求参数和响应类型
- API 所属角色命名空间（CA / CO / Mine 等）

### 第五步：后端代码分析

#### 5.1 定位后端 Controller

根据前端 API 路径，在后端代码中搜索对应的 Controller：

```sh
# 搜索包含 API 路径的 Controller 文件
rg -l "/ca/cmd/channel\|/co/cmd/channel" $HITO_DIR --include '*.kt' -g '!*test*'

# 通过 codegraph 搜索相关符号
codegraph query "<关键词>" --path $HITO_DIR -l 30 -j
```

#### 5.2 读取 Controller 层

读取 Controller 文件，确认每个 API 端点的：

- 完整路径和 HTTP 方法
- 请求体 / 查询参数
- 鉴权注解（`@ServiceResource`）
- 调用的 CommandHandler 或 Querier

#### 5.3 读取 Command / Query 处理层

读取 CommandHandler 文件，分析业务逻辑：

- **权限校验**：`horizonAuth.assertCrowdOwner` / `assertCrowdAdmin` → 确定哪些角色可以执行操作
- **业务校验**：名称唯一性检查、数量上限检查、状态校验 → 确定业务约束
- **领域对象操作**：创建、修改、删除的实体字段 → 确定功能的数据流

读取 Querier 文件，分析查询逻辑：

- 数据聚合方式
- 分页机制
- 过滤条件

#### 5.4 读取领域模型

读取 Domain 实体文件（如 `Channel.kt`、`ChannelCode.kt`），了解：

- 实体的核心属性
- 业务常量（如 `MAX_CHANNELS_PER_CROWD = 100`）
- 领域方法（如 `assertEnabled()`、`toggleEnabled()`）

### 第六步：编写用户手册

在完整分析前后端代码后，按以下固定结构编写文档。

## 文档结构

用户手册必须遵循以下六章结构。每个 `###` 标题下的内容应自包含，便于 RAG chunking：

```markdown
# {功能中文名}

## 概述

<!-- 2-3 段文字说明：功能是什么、解决什么问题、目标用户是谁 -->

## 核心概念

<!-- 关键术语表，用子弹列出概念名称和说明 -->

- 概念1：说明
- 概念2：说明
……

## 操作指南

### {场景1：创建/配置}

<!-- 逐步操作说明，每步包含：操作入口 → 操作动作 → 预期结果 -->
<!-- 如果前端代码中有条件分支（如角色判断），说明在不同角色下的差异 -->

### {场景2：查看/使用}

<!-- 同上 -->

### {场景3：管理/维护}

<!-- 同上 -->

<!-- 场景数量根据实际功能决定，每个场景一个 ### -->

## 权限说明

<!-- 角色权限矩阵表格 -->

| 操作 | 群主 (CO) | 管理员 (CA) | 说明 |
|------|----------|------------|------|
| ... | ✓ | ✓ / ✗ / 仅自己创建的 | ... |

## 限制与约束

<!-- 列出所有在代码中发现的硬性限制和业务规则 -->

- 数量上限：...
- 命名规则：...
- 删除限制：...
- 状态约束：...

## 常见问题

<!-- 基于代码中的错误提示、边界条件和空状态文案整理 FAQ -->

### 为什么无法删除某个渠道？
<!-- 从代码逻辑中找到原因并解释 -->

### 为什么渠道码没有显示扫码用户？
<!-- 同上 -->
```

### 编写原则

1. **面向终端用户**：用产品语言而非技术语言。例如说"群主"而非"CO"，说"管理员"而非"CA"。
2. **所有描述必须有代码依据**：操作步骤来自前端方法的执行流程，业务规则来自后端 CommandHandler 的校验逻辑。
3. **场景覆盖完整**：根据前端页面的条件分支，覆盖所有角色视角下的操作流程。
4. **FAQ 从错误中来**：所有常见问题的答案都能在代码的 `throw Ex.of(...)`、`showToast`、`wx.showModal`、空状态文案中找到依据。
5. **不编造截图表**：不生成图片描述，但可以在操作步骤中用 `[截图：XX页面]` 标注需要补充截图的位置。

## 输出产物

最终文档保存到 `manual/{功能英文名}.md`，例如：

```
manual/channel-code.md
```

文件名使用功能英文名（小写，连字符分隔），与代码中的模块名保持一致。

生成完成后，输出文档路径和章节摘要，告知用户文档已生成。
