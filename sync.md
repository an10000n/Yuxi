### 获取开源分支新增的提交哈希并按时间正序保存
```
git log --pretty=format:"%H"  v0.6.0..v0.7.0 --reverse > cherry-picks.txt
```
### 批量应用这些提交到工作区
```
git cherry-pick -n $(cat cherry-picks.txt)

# 如果有冲突，解决后执行 git add . 再 git cherry-pick --continue
```

当 cherry-pick 遇到代码冲突时，Git 会暂停操作并标记冲突文件。处理步骤如下：

| 序号 | 步骤   | 命令/操作                      |作用|注意事项|
|-----|------|----------------------------|--|---|
| 1️⃣ | 检测冲突 | git status                 |查看冲突文件列表|红色标记的文件需处理|
| 2️⃣ | 定位冲突 | git diff 或 IDE 查看          |识别具体冲突位置|文件中会显示 <<<<<<< HEAD 等标记|
| 3️⃣ | 解决冲突 | 手动编辑文件                     |保留/合并所需代码|删除冲突标记（<<<<<<<, =======, >>>>>>>）|
| 4️⃣ | 标记解决 | git add <文件>               |将解决后的文件暂存|必须执行，否则 Git 认为冲突未解决|
| 5️⃣ | 继续操作 | git cherry-pick --continue |完成 cherry-pick 流程|若使用 -n 参数，此步后仍不会自动提交|