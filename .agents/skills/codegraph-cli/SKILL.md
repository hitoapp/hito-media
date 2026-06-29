---
name: codegraph
description: CodeGraph 代码智能与知识图谱 CLI 使用指南。用于在任意代码库中初始化索引、搜索符号、分析调用关系（callers/callees）、评估改动影响范围、查找受影响的测试文件。适用于"用 codegraph 搜索"、"分析代码调用链"、"查找受影响的测试"、"初始化 codegraph"等请求。
---

# CodeGraph

## 初始化项目并构建索引，会在项目根目录创建 `.codegraph/`

```sh
codegraph init [path]
```

选项：`-v` / `--verbose` 详细日志。

## 移除 CodeGraph（删除 `.codegraph/` 目录）

```sh
codegraph uninit [path]
```

## 全量索引所有文件

```sh
codegraph index [path]
```

选项：`-f` / `--force` 强制重建；`-q` / `--quiet` 静默；`-v` / `--verbose` 详细日志。

## 增量同步变更文件，日常使用

```sh
codegraph sync [path]
```

选项：`-q` / `--quiet` 静默（适合 git hook）。

## 搜索符号（函数、类、变量等）

```sh
codegraph query <search> [options]
```

| 选项 | 说明 |
|------|------|
| `-p, --path <path>` | 项目路径 |
| `-l, --limit <number>` | 最大结果数（默认 10） |
| `-k, --kind <kind>` | 按类型过滤：`function`、`class`、`method` 等 |
| `-j, --json` | JSON 格式输出 |

```sh
codegraph query "handleSubmit" -k function -l 20
```

## 查看项目文件结构

```sh
codegraph files [options]
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

```sh
codegraph files --filter src --pattern "*.vue" --max-depth 2
```

## 查找谁调用了指定符号

```sh
codegraph callers <symbol> [options]
```

选项：`-p, --path`、`-l, --limit`（默认 20）、`-j, --json`。

```sh
codegraph callers "createChannel" -l 30
```

## 查找指定符号调用了谁

```sh
codegraph callees <symbol> [options]
```

选项：`-p, --path`、`-l, --limit`（默认 20）、`-j, --json`。

```sh
codegraph callees "createChannel"
```

## 分析修改某个符号会影响哪些代码

```sh
codegraph impact <symbol> [options]
```

| 选项 | 说明 |
|------|------|
| `-p, --path <path>` | 项目路径 |
| `-d, --depth <number>` | 遍历深度（默认 2） |
| `-j, --json` | JSON 输出 |

```sh
codegraph impact "ChannelCode" -d 3
```

## 根据变更的源文件，找出受影响的测试文件

```sh
codegraph affected [options] [files...]
```

| 选项 | 说明 |
|------|------|
| `-p, --path <path>` | 项目路径 |
| `--stdin` | 从 stdin 读取文件列表（每行一个） |
| `-d, --depth <number>` | 依赖遍历深度（默认 5） |
| `-f, --filter <glob>` | 测试文件过滤（如 `e2e/*.spec.ts`） |
| `-j, --json` | JSON 输出 |
| `-q, --quiet` | 仅输出路径，无装饰 |

```sh
git diff --name-only HEAD~1 | codegraph affected --stdin -q | xargs npm test
```

## 查看索引状态和统计

```sh
codegraph status [path]
```

选项：`-j` / `--json` JSON 格式输出。

## 移除残留锁文件（索引异常中断时使用）

```sh
codegraph unlock [path]
```
