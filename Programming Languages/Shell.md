Useful commands

1. To figure out the default shell:

   ```shell
   echo $SHELL
   ```


2. To figure out all available shells:

   ```shell
   cat /etc/shells
   ```


3. Switch between shells:

   ```shell
   chsh -s /bin/bash
   chsh -s /bin/zsh
   ```


4. Alias: defined in your shell configuration file, and act as a shortcut to reference a frequently used command, for example:

   ```shell
   alias v="vim"
   ```


5. Xxx



### About Environment Variables

1. Priority of loading environment variable coifigure files:

   ```shell
   /etc/profile
   /etc/bashrc & /etc/zshrc
   /etc/paths 
   ~/.bash_profile # macOS
   ~/.bash_login 
   ~/.profile 
   ~/.bashrc & ~/.zshrc # linux
   ```


2. When you need to add a environment variable, better not to add directly to **/etc/paths** , but to create a new file in directory **/etc/paths.d** , and include your desired path, for example:

   ```shell
   /opt/homebrew/bin
   ```


3. Similarly, modification to **/etc/profile** , **/etc/bashrc** and **/etc/zshrc** are not recommended.

4. This is how to add paths to environment variable **$PATH**:

   ```shell
   export PATH="$PATH:<PATH 1>:<PATH 2>:<PATH 3>:...:<PATH N>"
   ```


5. And how to show the current **$PATH**:

   ```shell
   echo $PATH
   ```


6. PROMPT:

   Under oh-my-zsh, if using a theme, $PROMPT can be changed in the corresponding theme configuration file. (e.g. `~/.oh-my-zsh/themes/ys.zsh-theme`)

7. If put many export in a script:

   If you are executing your files like `sh 1.sh` or `./1.sh`, you are executing it in a sub-shell. If you want the changes take effect in current shell, you could do: `. 1.sh` or `source 1.sh`

8. xxx



### Shell syntax

1. String:
2. Variables:
   - `$#`: the number of parameters the script was called with
   - `$@`: all parameters `$1 .. whatever`
   - `${@:2}`: all arguments start from 2nd one (included) (index start from 1) 
   - `$*`: similar to `$@`, but does not preserve any whitespace quoting
   - `$0`: the basename of the program as it was called.
   - `$1 .. $9`: the first 9 additional parameters the script was called with.
3. List
   - `[@]`: all elements
   - 
4. Function:`funname() {}` or `function funname() {}`
5. Backtick (`) symbol: Everything you type between backticks is evaluated (executed) by the shell before the main command and the *output* of that execution is used by that command, just as if you'd type that output at that place in the command line.
6. If & for & ...
   - for: `for i in $(seq 1 $#); do xxx done`
   - if
   - Xxx
7. xx



### Redirect

- `2>&1` means redirecting `stderr` to `stdout` because:
  - File descriptor 1 is the standard output (`stdout`)
  - File descriptor 2 is the standard error (`stderr`).
- `/dev/null`: a null device file in Linux, discards anything written to it, and return EOF on reading.
- 