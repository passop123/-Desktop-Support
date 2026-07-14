# 修改 GitHub 提交时间（PowerShell 操作指南)

本文档整理自一次交互式支持会话，记录了在本地通过 git amend 修改最近一次提交的 author/committer 时间并推送到 GitHub 的完整流程、出现的问题与对应解决办法，适用于在 Windows PowerShell 下操作的情形。

---

## 概要
目标：将最近一次提交的 AuthorDate 与 CommitDate 修改为 `2025-07-13T12:00:00+08:00`，并把改写后的提交强制推送到远端仓库 `origin/main`。
仓库（示例）：https://github.com/passop123/-Desktop-Support.git

用户最终成功执行了修改并 push，git show 显示：

```
AuthorDate: Sun Jul 13 12:00:00 2025 +0800
CommitDate: Sun Jul 13 12:00:00 2025 +0800
```

---

## 一：准备工作（先决条件）
- 确保你位于仓库根目录（含 `.git`）。
- 建议先创建远端备份分支以便回滚（强烈推荐）。
- 使用 PowerShell（每条命令单独回车执行，切勿把多条命令合并为一行，除非用 `;` 分隔）。

---

## 二：安全备份（强烈建议）
在仓库目录逐行运行：
```powershell
git fetch origin
git checkout main                     # 若分支不是 main，请换成当前分支名
git pull origin main
git branch backup-main
git push origin backup-main
```

备份分支 `backup-main` 可在出问题时用于恢复。

---

## 三：修改最近一次提交的时间（PowerShell，逐行执行）
1) 临时设置 committer 时间（环境变量，仅当前会话有效）：
```powershell
$env:GIT_COMMITTER_DATE="2025-07-13T12:00:00+08:00"
```

2) amend 最近一次提交（只改时间，不改提交信息）：
```powershell
git commit --amend --no-edit --date="2025-07-13T12:00:00+08:00"
```

3) 删除临时环境变量：
```powershell
Remove-Item Env:GIT_COMMITTER_DATE
```

4) 强制推送（更安全的强推）：
```powershell
git push --force-with-lease origin main
```

5) 验证提交时间：
```powershell
git show --pretty=fuller -s HEAD
```

---

## 四：如果要修改历史中某一条早期提交（交互式 rebase）
- 找到目标提交 SHA：
```bash
git log --oneline --decorate --graph -n 50
```
- 交互 rebase（把 `<TARGET_SHA>` 替换成目标提交的 SHA）：
```bash
git rebase -i <TARGET_SHA>^
```
- 把目标行 `pick` 改为 `edit`，保存退出。rebase 停在该提交时：
```powershell
$env:GIT_COMMITTER_DATE="2025-07-13T12:00:00+08:00"
git commit --amend --no-edit --date="2025-07-13T12:00:00+08:00"
Remove-Item Env:GIT_COMMITTER_DATE
git rebase --continue
```
- 完成后强推：
```bash
git push --force-with-lease origin main
```

注：若出现冲突，按提示修改、git add <file> 后 git rebase --continue。要中止：git rebase --abort。

---

## 五：常见错误与解决办法（对话中遇到的问题）
1) 把多条命令放在一行，导致 git 解析错误（示例错误：`error: unknown switch 'D'` 或 `error: no action specified`）
   - 解决：每条命令单独回车执行，或用 `;` 分隔。
   - 示例错误来源：`git clone https://... cd -Desktop-Support`（把 cd 也给 git 了）。

2) 目标文件夹以 `-` 开头（例如 `-Desktop-Support`），在 PowerShell 中需用 `.\` 前缀或引号：
   - 进入目录：
```powershell
Set-Location -LiteralPath ".\-Desktop-Support"
# 或
cd ".\-Desktop-Support"
```
   - 可使用 Tab 补全避免输入错误。

3) Set-Location 错误（路径不存在）
   - 原因：缺少 `\` 前缀或路径拼写错误。先用 `Get-ChildItem -Directory` 查看目录名，再 cd 进入。

4) commit amend 时出现 “Please tell me who you are” / fatal: unable to auto-detect email
   - 解决：在当前仓库设置 user.name/user.email（不加 --global 仅影响当前仓库）：
```powershell
git config user.name "pangxiegong"
git config user.email "2871300742@qq.com"
```

5) 将 PowerShell 命令误当作 git 参数（示例错误：`error: pathspec 'Remove-Item' did not match any file(s) known to git`）
   - 原因：把三条命令合成了一行。请逐行执行 `$env:...`、`git commit --amend ...`、`Remove-Item ...`。

6) 认证/连接问题（info: please complete authentication in your browser... / Could not connect to server）
   - 诊断命令：
```powershell
ping github.com
Test-NetConnection github.com -Port 443
git remote -v
```
   - 若网络连通但出现浏览器认证提示，按提示在浏览器完成授权（Git Credential Manager 会弹出浏览器）。
   - 可选择 PAT 或 SSH 作为替代认证方式（见下节）。

---

## 六：认证方式（A/B/C 比较与操作）
A) 浏览器认证（Git Credential Manager）
- 优点：交互方便，默认行为。
- 缺点：需要能访问 github.com:443 并能在本地打开浏览器。

B) Personal Access Token（PAT）通过 HTTPS（适用于受限环境）
- 在 GitHub → Settings → Developer settings → Personal access tokens 创建 PAT（至少勾选 `repo` 权限）。
- 推送时以用户名 + PAT 作为密码输入，或临时将 remote 修改为含 token 的 URL（不推荐长期保存）。
```bash
git remote set-url origin https://<username>:<PAT>@github.com/owner/repo.git
git push --force-with-lease origin main
# 恢复 remote（完成后）
git remote set-url origin https://github.com/owner/repo.git
```

C) SSH（长期推荐）
- 生成 key 并添加到 GitHub → Settings → SSH and GPG keys：
```powershell
ssh-keygen -t ed25519 -C "2871300742@qq.com"
# Windows: start ssh-agent then add key
Start-Service ssh-agent
ssh-add $env:USERPROFILE\.ssh\id_ed25519
# 复制公钥并粘贴到 GitHub
Get-Content $env:USERPROFILE\.ssh\id_ed25519.pub | clip
# 修改 remote 为 SSH 并推送
git remote set-url origin git@github.com:passop123/-Desktop-Support.git
git push --force-with-lease origin main
```

---

## 七：会话中实际诊断与结果摘录
用户执行网络/远端诊断得到的结果（摘要）：
- ping github.com 成功，平均 118 ms，0% 丢包
- Test-NetConnection github.com -Port 443 返回 `TcpTestSucceeded : True`
- git remote -v 指向 `https://github.com/passop123/-Desktop-Support.git`

push 过程中的关键信息：
```
info: please complete authentication in your browser...
Enumerating objects: 5, done.
...
To https://github.com/passop123/-Desktop-Support.git
 + f45d4cb...54e667e main -> main (forced update)
```
最终 `git show --pretty=fuller -s HEAD` 输出：
```
commit 54e667ed4379e5e718623eecd856d4ff104ac2d4 (HEAD -> main, origin/main, origin/HEAD)
Author:     pangxiegong <142774308+passop123@users.noreply.github.com>
AuthorDate: Sun Jul 13 12:00:00 2025 +0800
Commit:     pangxiegong <2871300742@qq.com>
CommitDate: Sun Jul 13 12:00:00 2025 +0800
```
说明：Author 使用了 GitHub 的 `noreply` 地址（142774308+passop123@users.noreply.github.com），Commit 使用本地设置的邮箱。

---

## 八：恢复与清理（可选）
- 将远端恢复到备份分支（若要回滚）：
```bash
git push --force origin backup-main:main
```
- 删除远端备份分支（若确认不再需要）：
```bash
git push origin --delete backup-main
```

---

## 九：后续建议
- 若你想要 Author 也使用你的邮箱，可单独 amend 提交作者信息：
```bash
git commit --amend --author="pangxiegong <2871300742@qq.com>" --no-edit
git push --force-with-lease origin main
```
- 若需在仓库历史中批量替换邮箱/作者，建议使用 `git filter-repo`（慎用，需知会协作者并备份远端）。
- 推荐长期配置 SSH key，以避免每次认证时弹出浏览器或输入 PAT。

---

## 十：常见注意事项
- 修改已推送的提交会改写历史（force push），请确认没有其他合作者在该分支上并知会团队。  
- 使用 `--force-with-lease` 比直接 `--force` 更安全（避免覆盖别人新推的提交）。  
- 在 PowerShell 中注意路径以 `-` 开头的目录需用 `.\` 前缀或引号。

---

## 附录：关键命令（速查）
```powershell
# 备份远端分支
git fetch origin
git checkout main
git pull origin main
git branch backup-main
git push origin backup-main

# 修改最近一次提交时间（PowerShell）
$env:GIT_COMMITTER_DATE="2025-07-13T12:00:00+08:00"
git commit --amend --no-edit --date="2025-07-13T12:00:00+08:00"
Remove-Item Env:GIT_COMMITTER_DATE
git push --force-with-lease origin main
git show --pretty=fuller -s HEAD

# 修改当前提交的 author（仅一条）
git commit --amend --author="pangxiegong <2871300742@qq.com>" --no-edit
git push --force-with-lease origin main

# 恢复/删除备份
git push --force origin backup-main:main
git push origin --delete backup-main
```

---

如果你需要，我可以：
- 把这份文档保存成文件并提供给你（例如 README.md），或  
- 根据你想要的风格（更精简 / 更详细 / 加上步骤截图）调整文档，或  
- 帮你把 Author email 一并改回并演练一次（在当前仓库只改这一条提交）。
