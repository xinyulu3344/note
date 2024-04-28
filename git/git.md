# Git

## 常用命令

```bash
git config --global user.name xxx
git config --global user.email xxx

git init

git status

# 将文件从工作区添加到暂存区
git add .
git add FILE

# 提交暂存区所有修改到本地库
git commit -m ""

# 提交暂存区指定文件的修改到本地库
git commit -m "" FILE

git log
git log --pretty=oneline
git reflog

git reset --hard COMMIT_ID
git reset --hard HEAD^
git reset --hard HEAD^^
git reset --hard HEAD~100

git restore --staged FILE
git checkout -- FILE

git diff HEAD -- FILE
```

## 分支

```bash
# 查看分支列表
git branch

# 创建分支
git branch <分支名>

# 切换分支
git switch <分支名>
git checkout <分支名>

# 删除分支
git branch -d <分支名>

# 合并分支
git merge <分支名>

```

## 远程仓库

```bash
git remote add origin git@github.com-me:xinyulu3344/note.git
git remote add gitee git@gitee.com:xinyulu3344/note.git
git push -u origin 分支
git push gitee 分支
```

### 配置不同github账号使用不同密钥

编辑`~/.ssh/config`配置文件

```bash
Host github.com
    HostName github.com
    user jdcloud-me
    IdentityFile "~/.ssh/id_rsa_jdcloud-me_luxinyu"
    IdentitiesOnly yes
Host github.com-me
    HostName github.com
    user xinyulu3344
    IdentityFile "~/.ssh/id_rsa"
    IdentitiesOnly yes
```

## 配置使用beyond compare

```bash
git config --local merge.tool bc3
git config --local mergetool.path ''
git config --local mergetool.keepBackup false
```

解决冲突

```bash
git mergetool
```

