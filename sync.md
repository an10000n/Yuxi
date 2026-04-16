### 获取开源分支新增的提交哈希并按时间正序保存
```
git log v0.6.0..v0.6.0.xh415 --reverse --pretty=format:"%H %P" | ForEach-Object {
    $parts = $_ -split ' '
    $commit = $parts[0]
    $isMerge = ($parts.Count -gt 2)  # 父提交 >1 即为合并提交
    if ($isMerge) { "$commit -m 1" } else { $commit }
} > cherry-picks.txt
```
### 批量应用这些提交到工作区
```
# 1. 中止当前失败的 cherry-pick
git cherry-pick --abort
 
# 2. 重置工作区（保留已解决的冲突文件！）
git reset --mixed HEAD  # 保留工作区修改，清空暂存区
 
# 3. 逐个应用补丁（关键：-n + 手动处理冲突）
Get-Content cherry-picks.txt | ForEach-Object {
    $cmd = "git cherry-pick -n $_"
    Write-Host "🚀 执行: $cmd"
    iex $cmd
    if ($LASTEXITCODE -ne 0) {
        Write-Host "⚠️ 冲突检测! 请解决后执行: git add ." -ForegroundColor Yellow 
        pause
    }
}

# 4. 最终一次性提交 
git commit -m "整合多次 cherry-pick: OIDC 认证修复与文档更新"
```

当 cherry-pick 遇到代码冲突时，Git 会暂停操作并标记冲突文件。处理步骤如下：

| 序号 | 步骤   | 命令/操作                      |作用|注意事项|
|-----|------|----------------------------|--|---|
| 1️⃣ | 检测冲突 | git status                 |查看冲突文件列表|红色标记的文件需处理|
| 2️⃣ | 定位冲突 | git diff 或 IDE 查看          |识别具体冲突位置|文件中会显示 <<<<<<< HEAD 等标记|
| 3️⃣ | 解决冲突 | 手动编辑文件                     |保留/合并所需代码|删除冲突标记（<<<<<<<, =======, >>>>>>>）|
| 4️⃣ | 标记解决 | git add <文件>               |将解决后的文件暂存|必须执行，否则 Git 认为冲突未解决|
