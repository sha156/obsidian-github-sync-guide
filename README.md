# obsidian-github-sync-guide
同步obsidian笔记到github仓库
# Obsidian → GitHub 同步指南

将 Obsidian 笔记库备份到 GitHub 的完整流程，包含常见坑的处理方法。

---

## 前置准备

- 安装 [Git for Windows](https://git-scm.com/)
- 已有 GitHub 账号
- 配置好 SSH key（`ssh-ed25519`），并添加到 GitHub

验证 SSH 连接：
```bash
ssh -T git@github.com
```

---

## 初始化流程

```bash
cd 你的Obsidian库路径

git init
git branch -M main
git remote add origin git@github.com:你的用户名/仓库名.git
git pull origin main --allow-unrelated-histories
```

---

## .gitignore 配置

在库根目录创建 `.gitignore`，内容如下：

```
# 二进制可执行文件
*.exe

# MCP 插件运行环境
.obsidian/plugins/mcp-tools/bin/

# Obsidian 工作区状态（频繁自动变更）
.obsidian/workspace.json
.obsidian/workspace-mobile.json

# Obsidian 缓存
.obsidian/cache

# Smart-env 缓存
.smart-env/

# 粘贴的大图片
Pasted image*.png
Pasted image*.jpg
```

---

## 日常同步

```bash
git add .
git commit -m "sync: 更新笔记"
git push origin main
```

---

## 坑：文件超过 GitHub 100MB 限制

push 时报错 `GH001: Large files detected`，需要从历史中彻底删除该文件。

### 步骤一：从所有历史 commit 中移除大文件

```bash
git filter-branch --force --index-filter \
  "git rm --cached --ignore-unmatch 文件的相对路径" \
  --prune-empty --tag-name-filter cat -- --all
```

示例（删除 mcp 插件的 exe）：
```bash
git filter-branch --force --index-filter \
  "git rm --cached --ignore-unmatch .obsidian/plugins/mcp-tools/bin/mcp-server-windows.exe" \
  --prune-empty --tag-name-filter cat -- --all
```

### 步骤二：清理旧引用和对象

```bash
git for-each-ref --format="delete %(refname)" refs/original | git update-ref --stdin
git reflog expire --expire=now --all
git gc --prune=now
```

> Windows 上 `git gc` 可能提示 `Deletion of directory failed`，全部输 `n` 跳过，不影响结果。建议关闭 Obsidian 再执行。

### 步骤三：强制推送

```bash
git push origin main --force
```

---

## 注意事项

- `.gitignore` 只对**未追踪**的文件生效，已 commit 的文件必须用 `filter-branch` 清除
- Obsidian 运行时会锁定部分文件，执行 git 操作前建议关闭 Obsidian
- `workspace.json` 建议加入 `.gitignore`，否则每次打开 Obsidian 都会产生无意义的 commit
