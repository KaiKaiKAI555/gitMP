欢迎使用gitMP～这是一个辅助项目版本管理和项目上推迭代的skill🤖

触发方式： /git-mp，或提及"存档""回退版本""推送到GitHub""初始化git"等

三步工作流：

步骤	功能	说明
1. 初始化	git init + .gitignore + 首次 commit	自动检测是否已有仓库，无则创建
2. 日常存档	git add -A + commit	用户写备注，AI 自动读取项目文件生成工作总结，合并到 commit message
3. 选择操作	回退 / 推送 / 结束	三选一
Step 3 两条路径：

版本回退 → 展示 commit 列表 → 用户选择目标 → 显示将丢失的改动 → 确认后 git reset --hard
版本迭代 → 输入 GitHub URL → 自动配 remote → git push
核心设计： 回档前强制展示 git diff --stat，用户确认后才执行，防止误操作丢失代码。

注意⚠️：
关于Push to GitHub，需要用户配置 gh CLI，在github-setting-Developer Settings中生成一个token（repo编辑权限打开）。
终端运行gh auth login：选 GitHub.com → HTTPS → GitHub账号 + "Paste an authentication token"，粘贴新 token。即可✅
