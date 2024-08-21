# Shell syntax

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
   
   - if 比较字符串
   
     `if [ “x${var}" == “x” ]` 是为了判断 `${var}` 是否为空。如果不写 x ，只用 if `[ “${var}" == “0” ]` 来判断，当 `${var}` 为空或未设置时，语句被解释为 `if [ == "0" ]`，出现语法错误。
   
     > 补充提醒shell的奇葩语法： = 与 == 在 [ ] 中表示判断(字符串比较)时是等价的； 在 (( )) 中 = 表示赋值， == 表示判断(整数比较)，它们不等价； 而()、(())、[]、[[]]又是有区别的。。。 shell真是最值得被替代的一个老古董！
   
   - xxx
   
7. xx



### Redirect

- `2>&1` means redirecting `stderr` to `stdout` because:
  - File descriptor 1 is the standard output (`stdout`)
  - File descriptor 2 is the standard error (`stderr`).
- `/dev/null`: a null device file in Linux, discards anything written to it, and return EOF on reading.





# Useful

- get parent dir of script path:

  ```sh
  BASE_DIR=$(cd -P "$(dirname "${BASH_SOURCE-$0}")/../"; pwd -P)
  ```

- xxx



# bash vs sh

a `#!/bin/bash` script  is a bash script, and a `#!/bin/sh` script is a sh script, on some systems they are different.