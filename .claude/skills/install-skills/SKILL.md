---
name: install-skills
description: 把本仓库里的三个 skill 复制安装到全局 skill 目录 ~/.claude/skills/。当用户说「装一下 skill」「同步 skill」「安装 skill」「让 skill 改动生效」「重装」，或刚改完本仓库的 skill 文件需要落地时使用。
---

# 安装 skill 到全局目录

**本仓库是「源」，`~/.claude/skills/` 是「副本」。** 改仓库里的文件不会自动生效——必须复制过去。

本技能只在**本仓库**里用得上，所以放在 `.claude/skills/` 下（项目级），不随其余三个 skill 一起安装。

## 要装的是哪三个

`grilling`、`review-and-fix-cycle`、`vibe-coding`——仓库根目录下的三个顶层目录。**本技能自己不安装。**

## 步骤

**1. 定位仓库根**

本文件在 `<仓库根>/.claude/skills/install-skills/SKILL.md`，往上**三级**（`install-skills` → `skills` → `.claude`）就是仓库根。确认该目录下能看到那三个 skill 目录。

**2. 逐个复制：先删后拷**

```bash
REPO="<上一步得到的仓库根绝对路径>"
for s in grilling review-and-fix-cycle vibe-coding; do
  rm -rf "$HOME/.claude/skills/$s"
  cp -r "$REPO/$s" "$HOME/.claude/skills/$s"
done
```

**必须是先 `rm -rf` 再 `cp -r`，不能直接 `cp -r` 覆盖。** 直接覆盖会把上一版残留的、这次已删除的文件留在副本里，于是「删了的文件还在生效」。

**3. 逐个校验，不信「应该拷过去了」**

```bash
for s in grilling review-and-fix-cycle vibe-coding; do
  diff -r "$REPO/$s" "$HOME/.claude/skills/$s" && echo "OK: $s 一致"
done
```

`diff -r` 无输出即一致。有差异就重跑第 2 步，别跳过。

**4. 提醒用户执行 `/reload-skills`**

不必重启会话，但**不 reload 改动就不生效**。这一步不能省，也不要假设用户会自己想起来。

## 两个已知坑

| 坑 | 说明 |
|---|---|
| **别改回符号链接** | Node 扫描目录若用 `dirent.isDirectory()` 判断，符号链接目录会返回 `false` 而被**静默跳过**——不报错，skill 直接消失，排查成本极高。所以用复制。代价是仓库与安装位置分离，改完要重跑本流程。 |
| **`~/.claude/skills/` 下多出的旧目录** | 若某次改了 skill 的**名字**（而不只是内容），旧名字的目录不会被本流程删掉，会作为幽灵 skill 一直挂着。改名字时记得手工删旧目录。 |
