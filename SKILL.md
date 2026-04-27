---
name: git-mp
description: Multi-step git project management — initialize repos, daily commits with AI-generated work summaries, safe rollback with diff preview, and GitHub push. Use /git-mp or trigger this skill whenever the user wants to save/archive/backup their project, commit work with a summary, rollback or rewind to a previous version, push code to a remote repository, or set up version control for the first time. Even if the user just says "存档", "保存进度", "回退版本", "推到GitHub", or "帮我初始化git", consult this skill.
---

# GitMP — Project Git Management

A step-by-step git workflow for solo project management. Follow each step in order; do not skip or reorder.

## Step 1: Ensure git repo exists

Check if the current project directory is already a git repository:

```bash
git rev-parse --git-dir 2>/dev/null && echo "IS_REPO" || echo "NOT_REPO"
```

### If NOT a git repo (first time):
1. Run `git init`
2. Create a `.gitignore` file if one doesn't exist. Read the project structure to determine sensible defaults (e.g., `node_modules/`, `.env`, `.DS_Store`, `dist/`, etc.). If unsure, use a minimal set: `.DS_Store`, `.env`, and any obvious build/output directories.
3. Stage all files: `git add -A`
4. Create the initial commit with message: `"chore: initial commit - project setup"`
5. Report to user: "Git repo initialized and first commit created." Then proceed to Step 2.

### If already a git repo:
Report to user: "Git repo already exists (N previous commits)." Then proceed to Step 2.

## Step 2: Daily archive + work summary

### 2a: Check for changes
```bash
git status --short
```

If there are no changes (clean working tree), tell the user "Working tree is clean — nothing to commit." and skip to Step 3.

### 2b: Generate work summary
Read key project files to understand what has been built or changed. Focus on:
- Any PRD, README, or task lists in the project
- Recently modified source files (use `git diff --stat HEAD` to see what changed, or `git status` for new files)
- The overall project structure

Then generate a concise summary covering:
- **What the project is** (one line)
- **What changed in this phase** (bullet points of key additions/modifications)
- **Current project status** (what's done, what's in progress, what's next — if discernible)

### 2c: Commit
1. Stage all changes: `git add -A`
2. Ask the user: "请写一段commit备注（直接输入文字即可）："
3. Combine the user's note with the AI-generated summary into a commit message using this format:
   ```
   <user's note>

   ## 本阶段工作总结
   <AI-generated work summary>
   ```
4. Run `git commit -m "<combined message>"`
5. Output the AI-generated summary for the user to review.

## Step 3: Choose next action

Ask the user (using AskUserQuestion): "代码已存档。接下来想做什么？"

Options:
1. **版本回退** — 回到之前的某个存档点
2. **版本迭代** — 推送代码到 GitHub
3. **结束** — 什么都不做，就此结束

### Step 3-1: Rollback (版本回退)

1. Show recent commit history (last 15 entries):
   ```bash
   git log --oneline -15
   ```
2. Ask the user: "请选择要回退到的commit hash（输入前7位即可）："
3. **Important safety step**: Once the user selects a commit, show what will be discarded:
   ```bash
   git diff <chosen-commit>..HEAD --stat
   ```
4. Confirm with the user: "以下改动将被永久丢弃（以上 diff stat），确认回退吗？"
5. If confirmed, run:
   ```bash
   git reset --hard <chosen-commit>
   ```
6. Report: "已回退到 commit <hash>。当前状态："
   ```bash
   git log --oneline -3
   ```

**Safety note**: `git reset --hard` permanently discards uncommitted changes and rewinds history. The confirmation step (showing what will be lost) is mandatory — never skip it.

### Step 3-2: Push to GitHub (版本迭代)

1. Ask the user: "请输入 GitHub 仓库 URL（如 https://github.com/user/repo.git）："
2. Check if a remote already exists:
   ```bash
   git remote -v
   ```
3. If remote exists and differs from the provided URL, ask whether to replace it.
4. If no remote or user confirms replacement:
   ```bash
   git remote add origin <url>
   ```
   (Use `git remote set-url origin <url>` if replacing.)
5. Determine current branch:
   ```bash
   git branch --show-current
   ```
6. Push:
   ```bash
   git push -u origin <branch>
   ```
7. Report: "代码已推送至 <url>，分支：<branch>。"

### Step 3-3: Done
Simply say "存档完成，随时可以继续开发。" and exit.
