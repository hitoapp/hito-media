# hito-media

这个仓库用于管理喜多（HITO）对外发布的媒体内容，包括小红书图文、微信公众号文章，以及配套图片、文案草稿和发布记录。

## 目录结构

```text
hito-media/
├── publish/
│   ├── xiaohongshu/
│   └── official-account/
├── docs/
│   └── hito-mp-versions.md
├── skills/
│   ├── xiaohongshu-publish/
│   │   └── SKILL.md
│   └── wechat-official-publish/
│       └── SKILL.md
├── AGENTS.md
└── .tmp/
```

- `publish/xiaohongshu/`：存放小红书发布内容。
- `publish/official-account/`：存放微信公众号发布内容。
- `docs/hito-mp-versions.md`：记录关联 `hito-mp` 仓库的有效版本索引。
- `skills/`：存放本仓库维护的 Codex skill。
- `AGENTS.md`：项目级执行规范，包含关联代码仓库、版本分析和候选清单流程。
- `.tmp/`：临时工作目录，用于拉取关联代码仓库，不提交到 Git。

平台目录使用英文命名，主要是为了减少网页上传素材时 macOS 文件选择器对中文路径的兼容问题。

## 发布内容存放规则

每次生成小红书或公众号内容时，在对应平台目录下新建一个文件夹：

```text
publish/xiaohongshu/YYYY-MM-DD-主题/
publish/official-account/YYYY-MM-DD-主题/
```

示例：

```text
publish/xiaohongshu/2026-06-11-接入方管理升级/
publish/official-account/2026-06-11-活动分享统计优化/
```

建议每个发布文件夹内至少包含：

- `文案.md`：最终发布文案或候选文案。
- `素材说明.md`：图片生成需求、截图说明或素材来源。
- `生成配图/`：用于发布的封面图、正文图、产品截图或拼图。
- `发布记录.md`：发布平台、发布时间、链接和后续数据复盘。

正式写入文件前，应先在对话中确认标题、正文、配图方向和是否发布。发布后如需归档，记录最终链接、发布时间和后续复盘数据。

## 关联代码仓库

当用户要求“基于最近改动”“基于某个版本”“看看 hito-mp 最近更新”来生成发布内容时，需要先按项目级流程分析喜多小程序代码仓库：

```text
git@github.com:hitoapp/hito-mp.git
```

本仓库约定将它克隆到：

```text
.tmp/hito-mp
```

`.tmp/` 已加入 `.gitignore`，不会提交到本仓库。

如果 `.tmp/hito-mp` 不存在，先克隆：

```sh
mkdir -p .tmp
git clone git@github.com:hitoapp/hito-mp.git .tmp/hito-mp
```

如果 `.tmp/hito-mp` 已存在，先更新：

```sh
git -C .tmp/hito-mp pull --ff-only
```

然后列出有效 tag，并通过版本间 diff 分析最近产品改动：

```sh
git -C .tmp/hito-mp tag -n
git -C .tmp/hito-mp for-each-ref --sort=-creatordate --format='%(refname:short)|%(creatordate:short)|%(subject)|%(objectname:short)' refs/tags
git -C .tmp/hito-mp log --oneline <上个版本>..<目标版本>
git -C .tmp/hito-mp diff --stat <上个版本>..<目标版本>
git -C .tmp/hito-mp diff --name-only <上个版本>..<目标版本>
```

忽略没有 `v` 前缀、但语义上与 `v` 版本重复的 tag。例如 `1.9.4` 与 `v1.9.4` 同时存在时，只保留 `v1.9.4`。

每次拉取关联仓库后，应同步更新 `docs/hito-mp-versions.md`，方便后续查阅版本历史。版本分析完成后，先输出“可宣传改动候选清单”，让用户确认主题、平台和内容重点，再进入小红书或公众号 skill。

更完整的执行规范见 `AGENTS.md`。

## 安装 skill

本仓库内置的 skill 位于：

```text
skills/xiaohongshu-publish
skills/wechat-official-publish
```

安装到 Codex 本机 skill 目录：

```sh
mkdir -p ~/.codex/skills
cp -R skills/xiaohongshu-publish ~/.codex/skills/
cp -R skills/wechat-official-publish ~/.codex/skills/
```

安装后，重启 Codex 或刷新 skill 列表，再使用类似指令触发：

```text
基于 hito-mp 最近版本改动，帮我生成一篇宣传喜多的小红书图文内容
基于 hito-mp 最近版本改动，帮我生成一篇喜多公众号产品公告
```

小红书 skill 只生成或整理手动发布所需素材，不自动操作小红书后台。

公众号 skill 当前用于喜多产品更新类文章生成、摘要整理、配图建议和素材保存；平台发布由用户后续手动完成。
