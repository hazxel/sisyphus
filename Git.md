# Git Commands

- `git checkout`: updates the HEAD to point to either the specified branch or commit

  > HEAD is Git's way of referring to the current snapshot

- `git checkout -b newb` = `git branch newb` + `git checkout newb`
- `git push -X ours` vs `git pull -X ours`
- `git push --set-upstream origin master`
- `git add -u`
- `git stash` (`git stash save`) & `git stash pop`
- `git reset HEAD XXX/XXX/XXX.java`: undo add op
- `git reset --hard origin/master`: reset branch to master
- `git reset --soft HEAD^`: undo last commit op
- `git clean -xfd`: remove f(untracked files), d(directory), x(file in .gitignore)
- `git show HEAD`: check the status of the Head
- `git show-ref --head`: List references in a local repository, including HEAD reference

- `git update-ref <dst> <src>`: update the dst ref with the src provided

  > e.g. when head is detached, you can update the branch reference `refs/heads/your_branch` with `HEAD` to make head attached

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


- xxx

 

# Multi-Account

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

## 



# HEAD detached

Most of the time, HEAD points to a branch name. 

- When you add a new commit, your branch reference is updated to point to it, but HEAD remains the same. 
- When you change branches, HEAD is updated to point to the branch you’ve switched to. 

All of that means that, in these scenarios, HEAD is synonymous with “the last commit in the current branch.” This is the *normal* state, in which HEAD is *attached* to a branch.

The detached HEAD state is: HEAD pointing directly to a commit instead of a branch.