# vibe-coding-skills

一套中文 agent skill，实现「人定目标、AI 执行、多轮检查纠偏」的开发流程。适用于 Claude Code，也适用于任何支持同类技能格式（`SKILL.md` + YAML frontmatter）的 agent。

核心分工：**AI 负责「做」，人负责「定」。**

## 包含什么

| 目录 | 作用 |
|---|---|
| `vibe-coding/` | 总指挥。五阶段闭环流程，串起下面两个 skill，每个阶段设人工闸门 |
| `grilling/` | 需求追问。分轮次拷问计划，直到没有未决问题 |
| `review-and-fix-cycle/` | 复核循环。独立子代理查错 → 主代理逐条判断并修复 → 同一子代理再复核确认 |
| `.claude/skills/install-skills/` | **仅 Claude Code 用的安装器**，见下方「安装」。自己不参与安装 |

前三个目录各带一份 `agents/openai.yaml`（显示名、简介、是否允许隐式调用），供非 Claude 平台的技能注册使用。

`vibe-coding` 设计为**只手动触发**，两个平台各用各的开关：Claude Code 读 frontmatter 的 `disable-model-invocation: true`，其他平台读 `agents/openai.yaml` 的 `allow_implicit_invocation: false`。另两个 skill 既能被自动调起，也能手动调。

## 安装

把这三个目录**复制**到你的 agent 的技能目录：

```bash
REPO=/path/to/vibe-coding-skills
DST=~/.claude/skills          # 换成你那边 agent 的技能目录
for s in grilling review-and-fix-cycle vibe-coding; do
  rm -rf "$DST/$s"
  cp -r "$REPO/$s" "$DST/$s"
done
```

装完让 agent 重新加载技能清单（Claude Code 是 `/reload-skills`，不必重启会话）。

**Claude Code 上还有更省事的一条路**：在仓库里输入 `/install-skills`，或直接说「装一下 skill」——它会自己定位仓库根、复制、再逐个 `diff -r` 校验。

## 用法

能打斜杠命令就直接打：

```
/vibe-coding 给配置页加一个 CSV 批量导入
```

不能打的话用自然语言触发——三个 skill 的 description 里都写了触发语，例如「拷问我这个方案」「用 vibe coding 做个 ×××」。

单独用其中一个也行：

- `grilling` —— 只想被追问、还没打算动手
- `review-and-fix-cycle` —— 已经改完一批文件，只想复核

## 流程

| 阶段 | 做什么 | 闸门 |
|---|---|---|
| 一 · 需求构思与深度规划 | AI 产出 11 项开发计划；`grilling` 分轮追问 | **G1** 人确认计划 |
| 二 · AI 执行开发 | 一次性写完本轮全部改动 + 同步写单测 | **G2** 人确认开发完成 |
| 三 · AI 自动复核 | 独立子代理查错，循环至收敛 | **G3** |
| 四 · 最终验证（可选） | **先问用户要测什么**，再跑全量单测 / 端到端 / 发布测试（可多选） | **G4** |
| 五 · 提交代码 | 自查暂存范围、起草提交信息 | **G5** 人授权提交 |

阶段四**可以跳过**，但只有人能提——AI 不许建议跳过。跳过了什么，必须写进提交信息（`未做最终验证`）。

## 硬规则

| 规则 | 说明 |
|---|---|
| **四件事逐次获批** | `git commit` / `git push` / 跑**全量**单元测试 / 删除数据库。每次单独问，**上次同意不算这次**（跑单个测试文件不受限） |
| **闸门只能由人打开** | 静默 ≠ 同意。模糊回复（「嗯」「哦」「再看看」）视为未确认 |
| **不改变目标** | 实现细节可自主决定，需求/范围/技术选型/对外接口必须停下来问人 |
| **交付版本由子代理确认** | 最后一轮必须是子代理的确认回执，不是主代理自述「已修复」。确认之后再改一个字符，确认作废 |
| **每次回复都带阶段脚注** | 首行播报阶段横幅，**末尾**用固定格式写明「当前阶段 / 此刻不做 / 建议」——建议只能取六种固定取值之一 |
| **验证方式由人选** | 阶段四用户没明确说要测什么，AI **必须先问**，不得自己选定，也不得把「跳过」摆进选项 |

## 平台差异

| 事项 | Claude Code | 其他平台 |
|---|---|---|
| 触发 | `/名称` 或自然语言 | 自然语言（description 里的触发语） |
| 禁止自动调用 | `disable-model-invocation: true` | `allow_implicit_invocation: false` |
| 重新加载技能 | `/reload-skills` | 按各平台机制，通常重开会话 |
| 一键安装 | `/install-skills` | 用上面的 `cp` 循环 |
| 派独立子代理复核 | Agent 工具 | 平台各自的子任务能力 |

最后一行值得留意：**阶段三「独立子代理复核」是整条流程最核心的一环**，它要求平台能起一个不带主 agent 上下文的独立执行体。平台没有这个能力的话，`vibe-coding` 仍能跑，但复核会退化成自我检查——那正是这套流程想避免的东西。

## 改完记得重装

改了仓库里的任何 skill 文件，都要**重跑安装**并让 agent 重新加载技能清单。否则改的是仓库，生效的还是旧副本。

## 来源与许可

| 目录 | 来源 |
|---|---|
| `grilling/` | 移植自 [mattpocock/skills](https://github.com/mattpocock/skills) 的 `skills/productivity/grilling`。MIT License, Copyright (c) 2026 Matt Pocock |
| `vibe-coding/`、`review-and-fix-cycle/` | 本仓库原创 |

设计树、可问集（frontier）、分轮次推进、问题格式均沿用上游。**本仓库对上游 grilling 的改动**：中文化；改写了探查规则——上游允许「先问不受探查结果影响的问题，不必等子代理回报」，本仓库改成**等本轮探查全部收齐、重算可问集之后再统一发问**（探查结果会改写问题本身，分两次问等于让用户把同一场拷问答两遍）；并补了探查失败的兜底——报错/超时/含糊结论都算「未查明」，转交用户裁决，不自己猜答案，也不因此卡住。

许可：本仓库以 **MIT** 发布，全文见 [LICENSE](LICENSE)。上游 grilling 的完整许可文本见 [THIRD-PARTY.md](THIRD-PARTY.md)——MIT 要求保留版权声明与许可声明，所以那份文本随仓库一并分发。
