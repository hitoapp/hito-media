# hito-media Agent Guide

本仓库用于维护喜多（HITO）对外发布内容。平台 skill 只负责小红书、微信公众号等平台的内容表达和素材组织；关联代码仓库获取、版本索引和产品改动分析属于项目级工作，统一放在这里和 `README.md` 中维护。

## 工作边界

- `skills/xiaohongshu-publish/`：专注小红书文案、手机端可读性、配图建议、话题建议、手动发布说明。
- `skills/wechat-official-publish/`：专注喜多产品更新类微信公众号文章、操作说明、配图图注和素材整理。
- `AGENTS.md` / `README.md` / `docs/hito-mp-versions.md`：维护 `hito-mp` 关联仓库、版本索引和产品改动分析流程。

当用户只提供截图、现成产品说明或明确主题时，可以直接进入对应平台 skill。只有当用户要求“基于最近改动”“基于某个版本”“看看 hito-mp 最近更新”等任务时，才执行下面的版本分析流程。

## 关联代码仓库

喜多小程序代码仓库：

```text
git@github.com:hitoapp/hito-mp.git
```

本仓库约定克隆到：

```text
.tmp/hito-mp
```

`.tmp/` 不提交到 Git。

## 版本分析流程

1. 确认 `.gitignore` 包含 `.tmp/`。
2. 如果 `.tmp/hito-mp` 不存在，提示用户需要先克隆，或执行：

   ```sh
   mkdir -p .tmp
   git clone git@github.com:hitoapp/hito-mp.git .tmp/hito-mp
   ```

3. 如果 `.tmp/hito-mp` 已存在，先更新：

   ```sh
   git -C .tmp/hito-mp pull --ff-only
   ```

4. 拉取失败时停止版本分析，说明失败原因，不要继续基于过期代码推断。
5. 拉取成功后列出 tag：

   ```sh
   git -C .tmp/hito-mp tag -n
   git -C .tmp/hito-mp for-each-ref --sort=-creatordate --format='%(refname:short)|%(creatordate:short)|%(subject)|%(objectname:short)' refs/tags
   ```

6. 忽略没有 `v` 前缀、但语义上与 `v` 版本重复的 tag。例如 `1.9.4` 与 `v1.9.4` 同时存在时，只保留 `v1.9.4`。
7. 同步更新 `docs/hito-mp-versions.md`，至少记录有效版本、tag 日期、说明、commit 摘要、最新 HEAD 和更新时间。
8. 让用户选择要宣传的版本范围。如果用户不想选，可以建议默认分析“最新有效 tag 与上一个有效 tag 之间的改动”，但仍需要用户确认。
9. 如果 HEAD 超过最新 tag，额外列出“尚未打 tag 的最新改动”，由用户决定是否纳入宣传。

## 改动分析方法

分析版本改动时，不要只看 commit message。至少结合：

```sh
git -C .tmp/hito-mp log --oneline <上个版本>..<目标版本>
git -C .tmp/hito-mp diff --stat <上个版本>..<目标版本>
git -C .tmp/hito-mp diff --name-only <上个版本>..<目标版本>
```

必要时继续查看关键文件 diff，判断哪些是真正可宣传的产品改动，哪些只是技术优化、配置调整或重构。

版本分析输出先给用户“可宣传改动候选清单”，不要直接进入正式文案。候选清单包含：

- 版本范围
- 技术改动摘要
- 可宣传的产品特性
- 对用户或运营者的价值
- 适合的平台与内容风格
- 建议配图方向
- 需要用户确认的问题

用户确认主题、重点和平台后，再进入对应平台 skill。

## 归档要求

- 内容产物按平台保存到 `publish/xiaohongshu/` 或 `publish/official-account/`。
- 版本索引只更新 `docs/hito-mp-versions.md`，不要把完整代码仓库或 `.tmp/` 内容提交。
- 提交发布素材时，只包含本次确认后的文案、素材、发布记录和必要文档变更。
