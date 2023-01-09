# Knowledges

Git中的每一次提交都产生git commit object，对应全局唯一的commit ID(SHA-1 算法生成)，叫做revesion hash。Commit对象内部包含了author + commit message的信息。Git存储的不是增量数据，对于每次改动存储的都是全部状态的数据。

由于commit ID全局唯一，非merge commit都有唯一的祖先commit。因此根据commit ID就可以决定git库的文件状态。

Branches是一种引用(ref)，指向确定的commit号。HEAD是一种特殊的ref，指向我们正在操作的分支。

符号^表示前一次提交，多个^表示前多次提交，如HEAD^。
符号\~后面可接数字表示前几次提交，如HEAD~3。


## Merge

当我们需要将feature分支上的代码merge到master分支上时，可以采用如下代码：

    git checkout master
    git merge feature

Merge remote branch:

    git fetch origin master:temp
    # Preview differences
    git diff temp
    git merge temp
    git branch -d temp
    
    or
    
    git merge origin/master

当两个分支进行merge时，会找两个分支的Lowest Common Ancestor，在此commit的基础上进行merge。

-   如果其中一个分支是另一个分支的祖先，merge方式为Fast-forward，仅移动该分支的指向，不会创建merge commit。
-   另一种情况下采用的是three-way merge(using the two snapshots pointed to by the branch tips and the common ancestor of the two)。此时会创建merge commit，同时所形成的一个圈内被merge分支的commit都会加入到该分支下方。被merge分支只需要再merge该分支，进行branch(ref)指针的移动即可。

git pull相当于git fetch + git merge。

[Git-Branching-Basic-Branching-and-Merging](https://git-scm.com/book/en/v2/Git-Branching-Basic-Branching-and-Merging#_basic_merging)


## Rebase

Rebase命令可以帮助我们将本分支的修改(commit)重新应用到另一分支上，这些commit都会以新的形式重新提交，而且使另一分支重新成为本分支的ancestor。

    git checkout feature
    git rebase master

如果想要对当前分支的近一次commit进行修改，也可以采用rebase命令：

    git checkout feature
    git rebase -i HEAD~3

在使用rebase命令时，最好避免对公开分支的commit进行重新修改，因为可能有其他人基于这些commit进行工作。

对远程分支进行rebase时，需要事先fetch，否则只会针对本地存在的origin分支。

[merging-vs-rebasing](https://www.atlassian.com/git/tutorials/merging-vs-rebasing)
[Git-Branching-Rebasing](https://git-scm.com/book/en/v2/Git-Branching-Rebasing)


## Undo things

`git commit --amend`

This command takes your staging area and uses it for the commit. You can stage more files, then use this command. So the previous commit will be replaced.

`git reset HEAD <file>`

Unstage staged files. `git reset --hard` will reset HEAD, index and working tree.

`git checkout -- <file>`

Discard changes in working directory.


## Stashing and Cleaning

`git stash -u --keep-index`

Stash the untracked files(-u) as well as the tracked ones. &#x2013;keep-index helps keep things in the index. -a will include the ignored files.

`git stash list`

Show git stash list.

`git stash apply --index`

Apply stash to current branch, keeping the staged changes.

`git stash drop stash@{0}`

Drop stash.

`git stash branch <new branch>`

Apply stash to indicated branch and generate a new branch.

`git clean -d -n`

Check which files will be removed. This command can only remove untracked files.

`git clean -d -f`

Actually remove files.


# Commands

`git push --set-upstream origin <branch>`

Push local branch to remote the first time.

`git remote add origin <URL>`

Add remote git.

`git config --global user.name <name>`

Configure user.name, user.email. Name and email need to be quoted with "".

`git cherry-pick <commit id>`

Pick up the modifications with a new commit ID. -x will keep the original author's information.

`git add -p`

支持交互式地选择chunk进行stage。

`git diff --staged`

检查暂存区文件的修改，不带参数&#x2013;staged为检查未暂存内容的修改。

`git checkout -b <new branch>`

在当前分支的基础上checkout新的分支。在命令后加上origin/master可以在远程分支的基础上checkout。

`git checkout <branch name>`

切换分支，注意在切换分支时尽量保证working directory是clean的状态。否则可能保留修改内容，也可能丢失修改内容。

`git log --pretty=oneline filename`

Show recent modifications on the specific file.

`git blame -L n,m filename`

Check who has done modifications on specified line ranges of the file.

`git show commitID filename`

Check the modifications of specified commitID on the file.
