---
title: 命令备忘录：Git
date: 2021-04-02 09:50:12
categories:
- 安装配置
- 应用软件
tags: 
- Linux
---

## 删除远程分支

`git push origin --del rzluan-orderchange/ndc_xml_api_v3`

## 克隆指定分支到指定文件夹

`git clone -b [branch_name] [git_url] [dir_path]`

## 变更统计

```
git log --author=rzluan --after="2019-11-01 00:00:00" --before="2019-11-30 23:59:59" --name-only | grep '^src/*' | sort |  uniq 
git log --author=rzluan --after="2019-11-01 00:00:00" --before="2019-11-30 23:59:59" --numstat | grep "src" | awk '{a[$3]+=$1;b[$3]+=$2}END{for(j in a) print j,"代码变更"(a[j]+b[j])"行"}'
git log --author=rzluan --after="2019-11-01 00:00:00" --before="2019-11-30 23:59:59" --numstat | grep "src" | awk '{a[$3]+=($1+$2)}END{for(j in a) b+=a[j]} END{print "总行数",b}'
```

## 修改commit记录

1. 修改提交时间：`git commit --amend --date="May 8 12:00:30 2020 +0800"`

2. 修改提交人信息：`git commit --amend --author "luanrzh luanrzh@qq.com"`

3. 强制推送至远程仓库：`git push <remote> <branch> -f`

## 撤销commit记录

1. `git reset HEAD` : 撤销commit，撤销暂存区，工作空间代码改动不变
2. `git reset --mixed HEAD^` : 同上
3. `git reset --soft HEAD^` : 撤销commit，暂存区不变，工作空间代码改动不变
4. `git reset --hard HEAD^` : 撤销commit，清空暂存区，清空工作空间代码改变
5. `git reset HEAD~n` : 撤销n次commit
6. `git commit --amend` : 修改commit注释

## 清除所有commit记录

> 参见[Git – Remove All Commits – Clear Git History (Local & Remote)](https://www.shellhacks.com/git-remove-all-commits-clear-git-history-local-remote/)

```shell
git checkout --orphan temp_branch # 切换到一个临时分支，该分支的状态和`git init`很像
# 在一步可以做一些特殊操作，如删除当前文件夹下所有文件、新增文件等等，后续新分支将会以此处的文件状态为准
git add -A # 提交工作区中所有文件变动，到临时分支暂存区
git commit -am "The first commit" # 提交暂存区中所有文件变动，到临时分支本地仓库
git branch -D main # 删除需要清除所有commit记录的分支
git branch -m main # 将当前临时分支修改成刚刚删掉的分支名
$ git push -f origin main # 推送到远程仓库（！！！危险操作！！！）
```