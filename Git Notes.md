## Git Commands

- `git checkout -b newb` = `git branch newb` + `git checkout newb`

- `git push -X ours` vs `git pull -X ours`

- `git push --set-upstream origin master`

- `git add -u`

- `git stash` (`git stash save`) & `git stash pop`

- `git reset HEAD XXX/XXX/XXX.java`: undo add op

- `git reset --hard origin/master`: reset branch to master

- `git reset --soft HEAD^`: undo last commit op

- `git clean -xfd`: remove f(untracked files), d(directory), x(file in .gitignore)

- For problem regarding **"Encountered 1 file(s) that should have been pointers, but weren't:"** link here: https://stackoverflow.com/questions/46704572/git-error-encountered-7-files-that-should-have-been-pointers-but-werent

  1. Push any changes you don't want to lose

  2. Stop Everything

  3. Open a Command Window (or Terminal)

  4. Uninstall LFS

     `git lfs uninstall`

     Then it'll print something like:

     `Hooks for this repository have been removed.  Global Git LFS configuration has been removed.`

  5. Reset

     `git reset --hard`

     It'll go through a lot of output...

  6. Reinstall LFS

     `git lfs install`

     This may again say it found files that should have been pointers but weren't. *That's OK, keep going!*

  7. Pull with LFS

     `git lfs pull`

  > Alternative：
  >
  > 1. git rm .gitattributes
  > 2. git reset --hard HEAD




## Interactive rebase

`git rebase` re-applies commits, one by one, in order, from your current branch onto another. It accepts several options and parameters, so that’s a tip of the iceberg explanation, enough to bridge the gap in between StackOverflow or GitHub comments and the git man pages.

An interesting option it accepts is --interactive (-i for short), which will open an editor with a list of the commits which are about to be changed. This list accepts commands, allowing the user to edit the list before initiating the rebase action.

1. run `git rebase -i -p <newest-good-commit>`
2. mark every commits you want to amend with `edit` or `e` for short
3. save and exit editor, you will be guided to the first commit you pick
4. run commands like `git commit --amend --reset-author`
5. to fix the next one, run `git rebase --continue`




## Multi-Account

1. Remove setting for global account

   `git config --global --unset user.name`

   `git config --global --unset user.email` 

2. Edit `~/.ssh/config` file as the following:

   ```
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

   `ssh -T git@github.com`

4. Lastly, configure **user.email** and **user.name** in every repository freshly cloned or existing before.


   `     git config user.email "my_office_email@gmail.com"`    

   `git config user.name "Rahul Pandey"`



## Git Submodules

### Commands

- `git submodule init`
- `git submodule update`
- `git submodule update --init --recursive`: equivalent to two previous commands

