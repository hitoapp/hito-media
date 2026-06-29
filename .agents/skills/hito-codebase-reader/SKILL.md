---
name: hito-codebase-reader
description: 喜多（HITO）前后端代码库阅读方法参考，适用于任何需要阅读喜多代码库的场景。
---

# hito-codebase-reader

## 目标

提供喜多（HITO）前后端代码库的阅读方法参考。

## 前置条件

- 已安装 `codegraph-cli` skill（用于代码符号搜索）
- 项目根目录存在 `.env` 文件，包含代码库路径变量
- `rg`（ripgrep）可用于补充搜索中文文案和硬编码字符串

## 获取代码库路径

从项目根目录 `.env` 文件中读取代码库路径：

```sh
cat .env
```

必需变量：

| 变量 | 说明 |
|------|------|
| `HITO_MP_DIR` | 前端小程序代码库路径 |
| `HITO_DIR` | 后端 Java/Kotlin 代码库路径 |

如果 `.env` 文件不存在或变量缺失，提示用户配置后再继续。

## 初始化代码库

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

## CodeGraph 使用说明

### 搜索符号（函数、类、变量等）

```sh
codegraph query <search> [options]
# 示例：
codegraph query "handleSubmit" -k function -l 20
```

| 选项 | 说明 |
|------|------|
| `-p, --path <path>` | 项目路径 |
| `-l, --limit <number>` | 最大结果数（默认 10） |
| `-k, --kind <kind>` | 按类型过滤：`function`、`class`、`method` 等 |
| `-j, --json` | JSON 格式输出 |

### 查看项目文件结构

```sh
codegraph files [options]
# 示例：
codegraph files --filter src --pattern "*.vue" --max-depth 2
```

| 选项 | 说明 |
|------|------|
| `-p, --path <path>` | 项目路径 |
| `--filter <dir>` | 限定目录 |
| `--pattern <glob>` | glob 过滤（如 `*.vue`） |
| `--format <format>` | 输出格式：`tree`（默认）、`flat`、`grouped` |
| `--max-depth <number>` | tree 模式最大深度 |
| `--no-metadata` | 隐藏元数据 |
| `-j, --json` | JSON 输出 |

### 查找谁调用了指定符号

```sh
codegraph callers <symbol> [options]
# 示例：
codegraph callers "createChannel" -l 30
```

选项：`-p, --path`、`-l, --limit`（默认 20）、`-j, --json`。

### 查找指定符号调用了谁

```sh
codegraph callees <symbol> [options]
# 示例：
codegraph callees "createChannel"
```

选项：`-p, --path`、`-l, --limit`（默认 20）、`-j, --json`。

### 分析修改某个符号会影响哪些代码

```sh
codegraph impact <symbol> [options]
# 示例：
codegraph impact "ChannelCode" -d 3
```

| 选项 | 说明 |
|------|------|
| `-p, --path <path>` | 项目路径 |
| `-d, --depth <number>` | 遍历深度（默认 2） |
| `-j, --json` | JSON 输出 |

### 根据变更的源文件，找出受影响的测试文件

```sh
codegraph affected [options] [files...]
# 示例：
git diff --name-only HEAD~1 | codegraph affected --stdin -q | xargs npm test
```

| 选项 | 说明 |
|------|------|
| `-p, --path <path>` | 项目路径 |
| `--stdin` | 从 stdin 读取文件列表（每行一个） |
| `-d, --depth <number>` | 依赖遍历深度（默认 5） |
| `-f, --filter <glob>` | 测试文件过滤（如 `e2e/*.spec.ts`） |
| `-j, --json` | JSON 输出 |
| `-q, --quiet` | 仅输出路径，无装饰 |

### 全量索引所有文件

```sh
codegraph index [path]
```

选项：`-f` / `--force` 强制重建；`-q` / `--quiet` 静默；`-v` / `--verbose` 详细日志。

### 增量同步变更文件，日常使用

```sh
codegraph sync [path]
```

选项：`-q` / `--quiet` 静默（适合 git hook）。


### 查看索引状态和统计

```sh
codegraph status [path]
```

选项：`-j` / `--json` JSON 格式输出。

### 移除残留锁文件（索引异常中断时使用）

```sh
codegraph unlock [path]
```
