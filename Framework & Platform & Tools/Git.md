# Git Commands

- `git checkout -b newb` = `git branch newb` + `git checkout newb`

- `git push -X ours` vs `git pull -X ours`

- `git push --set-upstream origin master`

- `git add -u`

- `git stash`

  - `git stash (save)`: stash all modifications

  - `git stash -p`: go through all modifications to deside which to stash

  - `git stash pop`: pop a modification

- `git clean -xfd`: remove f(untracked files), d(directory), x(file in .gitignore)

- `git show <commit-id/HEAD/...>`: check a commit

- `git log -p -- file.name`: check history of a file

- `git remote -v`: list of remotes

- `git fetch --all`: retrieve latest branches and commits from all remotes. (otherwise only origin)

  > **Fetch Rules**: control how to fetches branches and tags from a remote repository and stores locally
  >
  > - default: `fetch = +refs/heads/*:refs/remotes/origin/*`
  > - explanation: source references starts with `refs/heads/xxx` are stored as `refs/remotes/origin/xxx` locally
  > - modify through config file: `.git/config`
  > - modify through command: `git config remote.<remote>.fetch`

- `git patch`: 打补丁
  - `git diff > commit-patch`: generate patch
  -  `git patch <commit-patch>`: apply patch
    - `--check`: check if the patch can be successfully applied
    - `--stat`: list the modified files  


- `git fetch --prune origin`: fetch and prune

- `git remote prune origin`: prune only

  

 

# HEAD & commit tree

Head is git's term referring to current snapshot

- `git checkout`: updates the HEAD to point to either the specified branch or commit

  - `git checkout HEAD -- my-file.txt`: reset single file

- `git reset HEAD XXX/XXX/XXX.java`: undo add op

- `git reset --hard origin/master`: reset branch to master

- `git reset --soft HEAD^`: undo last commit op

- `git show-ref --head`: List references in a local repository, including HEAD reference

- `git update-ref <dst> <src>`: update the dst ref with the src provided

  > e.g. when head is detached, you can update the branch reference `refs/heads/your_branch` with `HEAD` to make head attached

- `git reflog`: 查看命令提交历史记录

- `get reset HEAD@{xxx}`: 根据编号回溯至某个commit



# Rebase

rebase可以**提取我们在另一分支上的改动，然后应用在当前分支的代码上**，完成类似于补丁的功能

- `git rebase master`: 将 master 分支的一些新改动应用到当前分支
- `git rebase --continue`: continue rebase
- `git rebase --abort`: stop rebase
- `git rebase --quit`: if had corrupted rebase with sth like  `'.git/rebase-apply/head-name': No such file or directory`



# Login & Account

### 对当前仓库

###  都记录在`.git/config` 

- 记住用户名邮箱：`git config user.name xxx` , `git config user.email xxx`
- 记住密码：`git config credential.helper cache` or `git config credential.helper store`
- xxx

### Multi-Account

1. Remove setting for global account

   `git config --global --unset user.name`

   `git config --global --unset user.email` 

2. Edit `~/.ssh/config` file as the following:

   ```config
   #github  
   Host github.com   
   HostName github.com   
   IdentityFile ~/.ssh/id_ed25519   
   User hazxel
   
   #helio  
   Host ssh.git.helio.dev   
   HostName ssh.git.helio.dev   
   PreferredAuthentications publickey   
   IdentityFile ~/.ssh/id_rsa_helio   
   User boyan
   ```

3. To check connectivity: 

   `ssh -T git@ssh.git.helio.dev`

   ``ssh -T git@github.com`



# Git Submodules

### Commands

- `git submodule init`
- `git submodule update`
- `git submodule update --init --recursive`: equivalent to two previous commands
- `git submodule update --remote`: pull latest commits



# HEAD detached

Most of the time, HEAD points to a branch name. 

- When you add a new commit, your branch reference is updated to point to it, but HEAD remains the same. 
- When you change branches, HEAD is updated to point to the branch you’ve switched to. 

All of that means that, in these scenarios, HEAD is synonymous with “the last commit in the current branch.” This is the *normal* state, in which HEAD is *attached* to a branch.

The detached HEAD state is: HEAD pointing directly to a commit instead of a branch.
