# 安装并试用专家技能

把获选技能放在独立 Codex 工作目录的 `.agents/skills/`。这里的安装步骤适用于
macOS 或 Linux，使用 Git、tar 和已配置登录的 Codex CLI。示例从已核对的
上游 commit 安装一个产品经理技能，同一个 repo 里的其他技能可以按需挑选。

## 在独立工作目录加载一个技能

下面以 PM Skills 的 `prioritize-assumptions` 为例，复制该技能目录和上游许可。
工作目录与下载源使用两个不同的临时目录。

```sh
(
  set -eu
  buddy_revision=9af7bfd0bc47fdad43c2eaa7a43cd13918e7aaef
  buddy_skill=pm-product-discovery/skills/prioritize-assumptions
  buddy_worker=$(mktemp -d)
  buddy_source=$(mktemp -d)
  trap 'rm -rf -- "$buddy_source"' EXIT

  git -C "$buddy_source" init -q
  git -C "$buddy_source" fetch -q --depth=1 \
    https://github.com/phuryn/pm-skills.git "$buddy_revision"
  git -C "$buddy_source" archive --output="$buddy_source/skill.tar" \
    "$buddy_revision" "$buddy_skill" LICENSE
  tar -xf "$buddy_source/skill.tar" -C "$buddy_source"

  git -C "$buddy_worker" init -q
  mkdir -p "$buddy_worker/.agents/skills"
  cp -R "$buddy_source/$buddy_skill" "$buddy_worker/.agents/skills/"
  cp "$buddy_source/LICENSE" \
    "$buddy_worker/.agents/skills/prioritize-assumptions/LICENSE.upstream"
  printf 'Worker directory: %s\n' "$buddy_worker"
)
```

需要评审的文档可以复制到输出的 worker 目录。用实际路径启动一个只读 Codex
会话，然后在输入框里显式选择 `$prioritize-assumptions`。

```sh
codex --cd '<worker-directory>' --sandbox read-only
```

```text
$prioritize-assumptions 评估 CSV 导入的三个假设：用户能定位问题行、
能修正后重传、能理解之前的列映射是否保留。请在对话中给出影响、风险和验证方式。
```

用 `/skills` 确认 `prioritize-assumptions` 出现在该目录的会话中。这个步骤
把第三方技能写入本次 worker 的 Git 工作目录。你的常用 Codex 会话继续使用自己
目录和用户配置里的技能。[Codex 技能说明](https://developers.openai.com/codex/skills)

## 换成其他职业技能

从链接选定来源和 commit，按相同方式取出包含 `SKILL.md` 的目录。把依赖的
模板、引用和适用许可证一起保留。技能自带许可证时，复制整个目录即可保留它。

| 场景 | 已固定版本的技能目录 | 额外准备 |
| --- | --- | --- |
| 产品经理评审假设 | [prioritize-assumptions](https://github.com/phuryn/pm-skills/tree/9af7bfd0bc47fdad43c2eaa7a43cd13918e7aaef/pm-product-discovery/skills/prioritize-assumptions) | 保留上游 MIT `LICENSE`。基础矩阵可单独使用。 |
| UX 研究访谈 | [interview-script](https://github.com/phuryn/pm-skills/tree/9af7bfd0bc47fdad43c2eaa7a43cd13918e7aaef/pm-product-discovery/skills/interview-script) | 保留同一份 MIT `LICENSE`，提供研究目标与受访者背景。 |
| Web UI 与可访问性 | [web-design-guidelines](https://github.com/vercel-labs/agent-skills/tree/1a2c79d4e6c4a2283dc2c614b6538cbe1ed5ca62/skills/web-design-guidelines) | 保留上游 MIT 许可。入口会读外部指南，做版本对照时也固定指南的版本。 |
| 浏览器 QA | [webapp-testing](https://github.com/anthropics/skills/tree/e4bb5d960d87126ff3b3a8af1a104b7db67560ba/skills/webapp-testing) | 保留目录内 Apache-2.0 许可和脚本。执行测试时为该 worker 配置 Python Playwright、本地测试站点和对应的工具权限。 |

角色名册与多阶段工作流通常有自己的载入方式。Agency Agents 提供 Codex
角色适配，gstack 的部分技能依赖其工具链。把所需角色或依赖配置在对应的专家
环境中。[Agency Agents 安装](https://github.com/msitarzewski/agency-agents) · [gstack 技能说明](https://github.com/garrytan/gstack/blob/main/docs/skills.md)

## 接入 Code Buddy 的方式

上面是可手动试用的安装路径。设计中的 harness 会从固定版本的缓存为每次派单
准备同样的技能目录，在该 worker 的 app-server 中用 `skills/list` 确认可用，
再通过 `turn/start` 的 skill 输入显式选中。自动检查上游、验证候选和切换版本
的流程见[运行设计](./code-buddy-dynamic-skills.md)。
