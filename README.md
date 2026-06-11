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
└── .tmp/
```

- `publish/xiaohongshu/`：存放小红书发布内容。
- `publish/official-account/`：存放微信公众号发布内容。
- `docs/hito-mp-versions.md`：记录关联 `hito-mp` 仓库的有效版本索引。
- `skills/`：存放本仓库维护的 Codex skill。
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

正式写入文件前，应先在对话中确认标题、正文、配图方向和是否发布。发布到对应平台后，需要把确认后的内容和素材提交并推送到本 Git 仓库。

## 关联代码仓库

生成文案前需要分析喜多小程序代码仓库：

```text
git@github.com:hitoapp/hito-mp.git
```

本仓库约定将它克隆到：

```text
.tmp/hito-mp
```

`.tmp/` 已加入 `.gitignore`，不会提交到本仓库。每次生成小红书或公众号内容前，都应先检查 `.tmp/hito-mp` 是否存在，并执行：

```sh
git -C .tmp/hito-mp pull --ff-only
```

然后通过 `git tag -n` 和版本间 diff 分析最近产品改动，把技术提交转译成适合宣传喜多产品价值的内容。

每次拉取关联仓库后，应同步更新 `docs/hito-mp-versions.md`，方便后续查阅版本历史。

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

如果需要实际发布到小红书，skill 会使用用户本机 Chrome 登录态和 Computer Use 操作小红书创作服务平台。发布前应先确认文案和图片，明确要求发布后才点击发布。

公众号 skill 当前先用于文章生成、保存和 Git 归档。公众号后台自动发布流程后续再按实际后台登录态和发布方式补充。
