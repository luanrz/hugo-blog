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

## 撤销commit

1. `1git reset HEAD` : 撤销commit，撤销暂存区，工作空间代码改动不变
2. `git reset --mixed HEAD^` : 同上
3. `git reset --soft HEAD^` : 撤销commit，暂存区不变，工作空间代码改动不变
4. `git reset --hard HEAD^` : 撤销commit，清空暂存区，清空工作空间代码改变
5. `git reset HEAD~n` : 撤销n次commit
6. `git commit --amend` : 修改commit注释
