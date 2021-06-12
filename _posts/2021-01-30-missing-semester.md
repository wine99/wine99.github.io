---
title: 计算机教育中缺失的一课 / MIT 6.NULL
categories: [CS]
comment: false
---

> https://missing.csail.mit.edu/
>
> https://missing-semester-cn.github.io/
>
> https://www.bilibili.com/video/BV14E411J7n2
>

# L1 - 课程概览与 shell

## 笔记

### 关于重定向和 cat

``` bash
$ echo hello > hello.txt
$ cat hello.txt
hello
$ cat < hello.txt
hello
$ cat < hello.txt > hello2.txt
$ cat hello2.txt
hello
```

本以为 `cat < hello.txt` 会报错 `cat: hello: No such file or directory`。猜想正确工作的原因是“参数”和“输入”的区别（未经验证或查找资料）：cat 程序将输入打印在屏幕上，`cat hello.txt` 中的 `hello.txt` 是参数，将该文件的内容作为输入；而 `cat < hello.txt` 是输入重定向，意思也是将文件中的内容作为程序的输入，而不是将文件的内容作为参数，因此二者效果相同。

### tee 的小用处

```bash
$ cd /sys/class/backlight/thinkpad_screen
$ sudo echo 3 > brightness
An error occurred while redirecting file 'brightness'
open: Permission denied
```

出乎意料的是，我们还是得到了一个错误信息。毕竟，我们已经使用了 `sudo` 命令！关于 shell，有件事我们必须要知道。`|`、`>`、和 `<` 是通过 shell 执行的，而不是被各个程序单独执行。 `echo` 等程序并不知道 `|` 的存在，它们只知道从自己的输入输出流中进行读写。 对于上面这种情况， _shell_ (权限为您的当前用户) 在设置 `sudo echo` 前尝试打开 brightness 文件并写入，但是系统拒绝了 shell 的操作因为此时 shell 不是根用户。

明白这一点后，我们可以这样操作：

```bash
$ echo 3 | sudo tee brightness 
```

因为打开 `/sys` 文件的是 `tee` 这个程序，并且该程序以 `root` 权限在运行，因此操作可以进行。

## 课后练习

[![szNEBq.png](https://s3.ax1x.com/2021/01/27/szNEBq.png)](https://imgchr.com/i/szNEBq)

[![szNAun.png](https://s3.ax1x.com/2021/01/27/szNAun.png)](https://imgchr.com/i/szNAun)

# L2 - Shell 工具和脚本

## 笔记

### Shell 脚本

#### 特殊变量

*  `$0` - 脚本名
*  `$1` 到 `$9` - 脚本的参数。 `$1` 是第一个参数，依此类推。
*  `$@` - 所有参数
*  `$#` - 参数个数
*  `$?` - 前一个命令的返回值
*  `$$` - 当前脚本的进程识别码
*  `!!` - 完整的上一条命令，包括参数。常见应用：当你因为权限不足执行命令失败时，可以使用 `sudo !!`再尝试一次。
*  `$_` - 上一条命令的最后一个参数。如果你正在使用的是交互式shell，你可以通过按下 `Esc` 之后键入 `.` 来获取这个值。

### 进程替换

一个冷门的类似特性是 _进程替换_ ( _process substitution_ )， `<( CMD )` 会执行 `CMD` 并将结果输出到一个临时文件中，并将 `<( CMD )` 替换成临时文件名。这在我们希望返回值通过文件而不是STDIN传递时很有用。例如， `diff <(ls foo) <(ls bar)` 会显示文件夹 `foo` 和 `bar` 中文件的区别。

### 通配（globbing）

```bash
convert image.{png,jpg}
# 会展开为
convert image.png image.jpg

cp /path/to/project/{foo,bar,baz}.sh /newpath
# 会展开为
cp /path/to/project/foo.sh /path/to/project/bar.sh /path/to/project/baz.sh /newpath

# 也可以结合通配使用
mv *{.py,.sh} folder
# 会移动所有 *.py 和 *.sh 文件

mkdir foo bar

# 下面命令会创建foo/a, foo/b, ... foo/h, bar/a, bar/b, ... bar/h这些文件
touch {foo,bar}/{a..h}
touch foo/x bar/y
# 显示foo和bar文件的不同
diff <(ls foo) <(ls bar)
# 输出
# < x
# ---
# > y
```

### shebang

注意，脚本并不一定只有用bash写才能在终端里调用。比如说，这是一段Python脚本，作用是将输入的参数倒序输出：

```python
#!/usr/local/bin/python import sys
for arg in reversed(sys.argv[1:]):
    print(arg) 
```

shell知道去用python解释器而不是shell命令来运行这段脚本，是因为脚本的开头第一行的 [shebang](https://en.wikipedia.org/wiki/Shebang_(Unix))。

在 `shebang` 行中使用 [`env`](http://man7.org/linux/man-pages/man1/env.1.html) 命令是一种好的实践，它会利用环境变量中的程序来解析该脚本，这样就提高来您的脚本的可移植性。`env` 会利用我们第一节讲座中介绍过的`PATH` 环境变量来进行定位。 例如，使用了`env`的shebang看上去时这样的`#!/usr/bin/env python`。

### shellcheck

编写 `bash` 脚本有时候会很别扭和反直觉。例如 [shellcheck](https://github.com/koalaman/shellcheck) 这样的工具可以帮助你定位sh/bash脚本中的错误。例如：

[![y9OItH.png](https://s3.ax1x.com/2021/01/28/y9OItH.png)](https://imgchr.com/i/y9OItH)

[![y9xQvn.png](https://s3.ax1x.com/2021/01/28/y9xQvn.png)](https://imgchr.com/i/y9xQvn)

## Shell 工具

### 查看命令如何使用

- [tldr](https://tldr.sh/)
- [cheat](https://github.com/cheat/cheat)

### 查找文件

- [find](http://man7.org/linux/man-pages/man1/find.1.html)
- [locate](http://man7.org/linux/man-pages/man1/locate.1.html)
- [fd](https://github.com/sharkdp/fd)

[locate 和 find 的对比](https://unix.stackexchange.com/questions/60205/locate-vs-find-usage-pros-and-cons-of-each-other)。

### 查找代码

- [grep](http://man7.org/linux/man-pages/man1/grep.1.html)
- [rg](https://github.com/BurntSushi/ripgrep)
- [ack](https://beyondgrep.com)
- [ag](https://github.com/ggreer/the_silver_searcher)

grep 的例子：

[![y9zXFO.png](https://s3.ax1x.com/2021/01/28/y9zXFO.png)](https://imgchr.com/i/y9zXFO)

rg 的例子：

```bash
# 查找所有使用了 requests 库的文件
rg -t py 'import requests'
# 查找所有没有写 shebang 的文件（包含隐藏文件）
rg -u --files-without-match "^#!"
# 查找所有的foo字符串，并打印其之后的5行
rg foo -A 5
# 打印匹配的统计信息（匹配的行和文件的数量）
rg --stats PATTERN
```

### 查找 shell 命令

- history | grep
- history | [fzf](https://github.com/junegunn/fzf)
- 快捷键 Ctrl + R
- 自动补全：[fish shell](https://fishshell.com/) 或者 [zsh-autosuggestions](https://github.com/zsh-users/zsh-autosuggestions)

有一点值得注意，输入命令时，如果您在命令的开头加上一个空格，它就不会被加进shell记录中。当你输入包含密码或是其他敏感信息的命令时会用到这一特性。如果你不小心忘了在前面加空格，可以通过编辑。`bash_history`或 `.zhistory` 来手动地从历史记录中移除那一项。

### 文件夹导航

- [fasd](https://github.com/clvv/fasd)
- autojump: [ohmyzsh](https://github.com/ohmyzsh/ohmyzsh)
- [tree](https://linux.die.net/man/1/tree)
- [ranger](https://github.com/ranger/ranger)

> **Oh-my-zsh? 新手上路看这篇：[Setting up Windows Terminal, WSL and Oh-my-Zsh](https://www.ivaylopavlov.com/setting-up-windows-terminal-wsl-and-oh-my-zsh/#install_windows_terminal)**

## 课后练习

### 习题 1

[![yCCQ0I.png](https://s3.ax1x.com/2021/01/28/yCCQ0I.png)](https://imgchr.com/i/yCCQ0I)

### 习题 2

macro.sh:

```bash
macro() {
    macro_dir=$(pwd)
    echo "I am in $macro_dir" | tee /mnt/f/code/learn/missing-semester/l2-shell-tools/macro.txt
}
```

polo.sh:

```bash
polo() {
    cd "$macro_dir" || exit
    macro
}
```

[![yC0XE4.png](https://s3.ax1x.com/2021/01/29/yC0XE4.png)](https://imgchr.com/i/yC0XE4)

### 习题 3

ex3_solution.sh:

```bash
#!/usr/bin/env bash

./ex3_problem.sh > ex3_result.txt 2> ex3_result.txt
state=$?
count=0

while [[ state -eq 0 ]]; do
    ./ex3_problem.sh >> ex3_result.txt 2>> ex3_result.txt
    state=$?
    count=$((count + 1))
done

cat ex3_result.txt
echo "ex3_problem ran $count times before failure"
```

[![yCyUkF.png](https://s3.ax1x.com/2021/01/29/yCyUkF.png)](https://imgchr.com/i/yCyUkF)

### 习题 4

```bash
$ tree ex4_html_folder
ex4_html_folder
├── 1.html
├── 1.txt
├── a
│   ├── a 1.html
│   ├── a 1.txt
│   ├── a 2.txt
│   └── a 3.txt
└── b
    ├── b 1.html
    ├── b 2.html
    └── b 3.html

2 directories, 9 files
```

参考 `tldr xargs` 给出的用法示例：

```
 - Delete all files with a .backup extension (-print0 uses a null character to split file names, and -0 uses it as delimiter):
   find . -name {{'*.backup'}} -print0 | xargs -0 rm -v
```

`tldr tar` 给出了 tar 命令的用法示例：

```
 - [c]reate an archive from [f]iles:
   tar cf {{target.tar}} {{file1}} {{file2}} {{file3}}

 - E[x]tract a (compressed) archive [f]ile into the target directory:
   tar xf {{source.tar[.gz|.bz2|.xz]}} --directory={{directory}}

 - Lis[t] the contents of a tar [f]ile [v]erbosely:
   tar tvf {{source.tar}}
```

因此本题解答如下：

```bash
find . -name "*.html" -print0 | xargs -0 tar cf html.tar
```

验证一下：

```bash
$ tar tvf html.tar
-rwxrwxrwx yzj/yzj           0 2021-01-29 15:00 ./ex4_html_folder/1.html
-rwxrwxrwx yzj/yzj           0 2021-01-29 15:25 ./ex4_html_folder/a/a 1.html
-rwxrwxrwx yzj/yzj           0 2021-01-29 15:25 ./ex4_html_folder/b/b 1.html
-rwxrwxrwx yzj/yzj           0 2021-01-29 15:25 ./ex4_html_folder/b/b 2.html
-rwxrwxrwx yzj/yzj           0 2021-01-29 15:25 ./ex4_html_folder/b/b 3.html

$ mkdir ex4_html_folder_extracted
$ tar xf html.tar --directory=ex4_html_folder_extracted
$ tree ex4_html_folder_extracted
ex4_html_folder_extracted
└── ex4_html_folder
    ├── 1.html
    ├── a
    │   └── a 1.html
    └── b
        ├── b 1.html
        ├── b 2.html
        └── b 3.html

3 directories, 5 files
```

上面的解法是把 find 命令的输出的分隔符，由原本的换行符变成了 null，然后让 xargs 也用 null 作为分隔符。也可以用 -d 选项指定换行符作为分隔符，因此另解如下：

```bash
find . -name "*.html" | xargs -d "\n" tar cf html.tar
```

### 习题 5

```bash
# 按最近修改顺序列出文件
$ find . -type f -print0 | xargs -0 ls -lt --color
-rwxrwxrwx 1 yzj yzj 10240 Jan 29 15:27  ./html.tar
-rwxrwxrwx 1 yzj yzj     0 Jan 29 15:25 './ex4_html_folder/a/a 1.html'
-rwxrwxrwx 1 yzj yzj     0 Jan 29 15:25 './ex4_html_folder_extracted/ex4_html_folder/a/a 1.html'
-rwxrwxrwx 1 yzj yzj     0 Jan 29 15:25 './ex4_html_folder/a/a 3.txt'
-rwxrwxrwx 1 yzj yzj     0 Jan 29 15:25 './ex4_html_folder/a/a 2.txt'
-rwxrwxrwx 1 yzj yzj     0 Jan 29 15:25 './ex4_html_folder/a/a 1.txt'
-rwxrwxrwx 1 yzj yzj     0 Jan 29 15:25 './ex4_html_folder/b/b 1.html'
-rwxrwxrwx 1 yzj yzj     0 Jan 29 15:25 './ex4_html_folder/b/b 2.html'
-rwxrwxrwx 1 yzj yzj     0 Jan 29 15:25 './ex4_html_folder/b/b 3.html'
-rwxrwxrwx 1 yzj yzj     0 Jan 29 15:25 './ex4_html_folder_extracted/ex4_html_folder/b/b 1.html'
-rwxrwxrwx 1 yzj yzj     0 Jan 29 15:25 './ex4_html_folder_extracted/ex4_html_folder/b/b 2.html'
-rwxrwxrwx 1 yzj yzj     0 Jan 29 15:25 './ex4_html_folder_extracted/ex4_html_folder/b/b 3.html'
-rwxrwxrwx 1 yzj yzj     0 Jan 29 15:01  ./ex4_html_folder/1.txt
-rwxrwxrwx 1 yzj yzj     0 Jan 29 15:00  ./ex4_html_folder/1.html
-rwxrwxrwx 1 yzj yzj     0 Jan 29 15:00  ./ex4_html_folder_extracted/ex4_html_folder/1.html
-rwxrwxrwx 1 yzj yzj   837 Jan 29 10:14  ./ex3_result.txt
-rwxrwxrwx 1 yzj yzj   291 Jan 29 10:11  ./ex3_solution.sh
-rwxrwxrwx 1 yzj yzj   205 Jan 29 09:58  ./ex3_problem.sh
-rwxrwxrwx 1 yzj yzj    58 Jan 29 09:52  ./macro.txt
-rwxrwxrwx 1 yzj yzj    49 Jan 29 09:48  ./polo.sh
-rwxrwxrwx 1 yzj yzj   129 Jan 29 09:44  ./macro.sh
-rwxrwxrwx 1 yzj yzj    50 Jan 28 21:41  ./mcd.sh
-rwxrwxrwx 1 yzj yzj   509 Jan 28 21:10  ./example.sh

# 找到最近修改的文件
$ find . -type f -print0 | xargs -0 ls -lt --color | head -n1
-rwxrwxrwx 1 yzj yzj 10240 Jan 29 15:27 ./html.tar
```

# L3 - 编辑器 (Vim)

## 笔记

个人建议的 vim 入门方法：

1. 先玩 [Vim Adventures](https://vim-adventures.com/)
2. 再做 vimtutor
3. 下一步？网上大佬太多了，随便找博客或者视频看，比如：[上古神器Vim：从恶言相向到爱不释手 - 终极Vim教程01 - 带你配置属于你自己的最强IDE](https://www.bilibili.com/video/BV164411P7tw)
4. [Vim Cheat Sheet](https://vim.rtorr.com/lang/zh_cn)

小贴士：在 linux 中，直接使用

`$ vimtutor`

命令就可以打开 vim tutor。或者使用

`$ vimtutor zh_ch`

命令打开中文版本。

# L4 - 数据整理

## 笔记

### REGEX

1. [入门交互式教程](https://regexone.com/)
2. [进阶文字教程](https://deerchao.cn/tutorials/regex/regex.htm)

[regex debugger](https://regex101.com/r/qqbZqh/2)

### A taste of data wrangling

``` bash
ssh myserver journalctl
 | grep sshd
 | grep "Disconnected from"
 | sed -E 's/.*Disconnected from (invalid |authenticating )?user (.*) [0-9.]+ port [0-9]+( [preauth])?$/2/'
 | sort | uniq -c
 | sort -nk1,1 | tail -n10
 | awk '{print $2}' | paste -sd, 
```

`sort -n` 会按照数字顺序对输入进行排序（默认情况下是按照字典序排序 `-k1,1` 则表示“仅基于以空格分割的第一列进行排序”。`,n` 部分表示“仅排序到第n个部分”，默认情况是到行尾。就本例来说，针对整个行进行排序也没有任何问题，我们这里主要是为了学习这一用法！

如果我们希望得到登陆次数最少的用户，我们可以使用 `head` 来代替`tail`。或者使用`sort -r`来进行倒序排序。

我们可以利用 `paste`命令来合并行(`-s`)，并指定一个分隔符进行分割 (`-d`)。

### AWK

`awk` 其实是一种编程语言，只不过它碰巧非常善于处理文本。

`awk` 程序接受一个模式串（可选），以及一个代码块，指定当模式匹配时应该做何种操作。默认当模式串即匹配所有行（上面命令中当用法）。 在代码块中，`$0` 表示整行的内容，`$1` 到 `$n` 为一行中的 n 个区域，区域的分割基于 `awk` 的域分隔符（默认是空格，可以通过`-F`来修改）。在这个例子中，我们的代码意思是：对于每一行文本，打印其第二个部分，也就是用户名。

再举个例子，让我们统计一下所有以`c` 开头，以 `e` 结尾，并且仅尝试过一次登陆的用户。

```bash
 | awk '$1 == 1 && $2 ~ /^c[^ ]*e$/ { print $2 }' | wc -l 
```

其中 `wc -l` 统计输出结果的行数。

既然 `awk` 是一种编程语言，那么则可以这样：

```bash
BEGIN { rows = 0 }
$1 == 1 && $2 ~ /^c[^ ]*e$/ { rows += $1 }
END { print rows } 
```

`BEGIN` 也是一种模式，它会匹配输入的开头（ `END` 则匹配结尾）。然后，对每一行第一个部分进行累加，最后将结果输出。

### bc

bc (Berkeley Calculator) 是一个命令行计算器。例如这样，可以将每行的数字加起来：

```bash
 | paste -sd+ | bc -l 
```

下面这种更加复杂的表达式也可以：

```bash
echo "2*($(data | paste -sd+))" | bc -l 
```

### Shell 命令中的 `-`

虽然到目前为止我们的讨论都是基于文本数据，但对于二进制文件其实同样有用。例如我们可以用 ffmpeg 从相机中捕获一张图片，将其转换成灰度图后通过SSH将压缩后的文件发送到远端服务器，并在那里解压、存档并显示。

```bash
ffmpeg -loglevel panic -i /dev/video0 -frames 1 -f image2 -
 | convert - -colorspace gray -
 | gzip
 | ssh mymachine 'gzip -d | tee copy.jpg | env DISPLAY=:0 feh -' 
```

其中 `-frames 1` 为第一帧画面，`-f image2` 将结果保存为图片而不是视频格式。

命令中 `-` 代表标准输入输出流，例如 `convert - -colorspace gray -` 的意思是把标准输入流的内容作为程序的输入，灰度处理后的结果再放到标准输出流中。

## 课后练习

### 习题 2

words 文件可以在这里下载：[/usr/share/dict/words](https://gist.github.com/wchargin/8927565)

```bash
$ grep -E "^.*[aA].*[aA].*[aA].*$" /usr/share/dict/words \
| grep -vE "'s$" \
| sed -E "s/^.*(\w{2})$/\1/" \
| sort \
| uniq -ic \
| sort -r \
| head -n3

    101 an
     63 ns
     51 ia
```

共存在多少种词尾两字母组合？显然

```bash
$ echo "26*26" | bc -l
676
```

我们把刚才的词尾保存下来，把所有的字母组合也保存为文件。

```bash
$ grep -E "^.*[aA].*[aA].*[aA].*$" /usr/share/dict/words \
| grep -vE "'s$" \
| sed -E "s/^.*(\w{2})$/\1/" \
| sort \
| uniq -i > words.txt 2> words.txt

$ cat words.txt | head -n5
aa
ac
ad
ae
ag

$ echo {a..z}{a..z} | sed -E 's/ /\n/g' > full_words.txt

$ cat full_words.txt | head -n5
aa
ab
ac
ad
ae
```

分别统计统计一下组合数：

```bash
$ wc -w full_words.txt
676 full_words.txt
$ wc -w words.txt
110 words.txt
```

然后我们找没有出现过的组合，具体做法是把 words.txt 中的每一行作为查找串，在 full_words.txt 中不匹配的行。

```bash
$ grep -F -v -f words.txt full_words.txt | head -n 5
ab
af
ai
aj
ao
```

结果应该共有 `676 - 110 = 566` 个，验证一下：

```bash
$ grep -F -v -f words.txt full_words.txt | wc -w
566
```

### 习题 3

用输出重定向进行原地替换只会得到空文件。`man sed` 中可以看到 sed 有 -i 选项，可以进行原地替换。

```
       -i[SUFFIX], --in-place[=SUFFIX]

              edit files in place (makes backup if SUFFIX supplied)
```

# L5 - 命令行环境

## 笔记

### 任务控制

shell 会使用 UNIX 提供的信号机制执行进程间通信。当一个进程接收到信号时，它会停止执行、处理该信号并基于信号传递的信息来改变其执行。就这一点而言，信号是一种软件中断。

#### 结束进程

*   See `man signal` for reference
*   `kill`: sends signals to a process; default is TERM
*   `SIGINT`: `^C`; interrupt program; terminate process
*   `SIGQUIT`: `^\`; quit program
*   `SIGKILL`: terminate process; kill program; cannot be captured by process and will always terminate immediately
    *   Can result in orphaned child processes
*   `SIGSTOP`: pause a process
    *   `SIGTSTP`: `^Z`; terminal stop
*   `SIGHUP`: terminal line hangup; terminate process; will be sent when terminal is closed    
    *   Use `nohup` to avoid
*   `SIGTERM`: signal requesting graceful process exit
    *   To send this signal: `kill -TERM <pid>`


下面这个 Python 程序向您展示了捕获信号 `SIGINT` 并忽略它的基本操作，它并不会让程序停止。为了停止这个程序，我们需要使用 `SIGQUIT` 信号。

```python
#!/usr/bin/env python import signal, time

def handler(signum, time):
    print("nI got a SIGINT, but I am not stopping")

signal.signal(signal.SIGINT, handler)
i = 0
while True:
    time.sleep(.1)
    print("r{}".format(i), end="")
    i += 1 
```

```
$ python sigint.py
24^C
I got a SIGINT, but I am not stopping
26^C
I got a SIGINT, but I am not stopping
30^\[1]    39913 quit       python sigint.py
```

#### 暂停和后台执行进程

使用 [`fg`](http://man7.org/linux/man-pages/man1/fg.1p.html) 或 [`bg`](http://man7.org/linux/man-pages/man1/bg.1p.html) 命令恢复暂停的工作。它们分别表示在前台继续或在后台继续。

[`jobs`](http://man7.org/linux/man-pages/man1/jobs.1p.html) 命令会列出当前终端会话中尚未完成的全部任务。可以使用 pid 引用这些任务（也可以用 [`pgrep`](http://man7.org/linux/man-pages/man1/pgrep.1.html) 找出 pid）。也可以使用百分号 + 任务编号（`jobs` 会打印任务编号）来选取该任务。如果要选择最近的一个任务，可以使用 `$!` 这一特殊参数。

命令中的 `&` 后缀可以让命令在直接在后台运行，不过它此时还是会使用 shell 的标准输出。

使用 `Ctrl-Z` 放入后台的进程仍然是终端进程的子进程，一旦关闭终端（会发送另外一个信号 `SIGHUP`），这些后台的进程也会终止。为了防止这种情况发生，可以使用 [`nohup`](http://man7.org/linux/man-pages/man1/nohup.1.html) (一个用来忽略 `SIGHUP` 的封装) 来运行程序。针对已经运行的程序，可以使用 `disown` 。

```
$ sleep 1000
^Z
[1]  + 18653 suspended  sleep 1000

$ nohup sleep 2000 &
[2] 18745
appending output to nohup.out

$ jobs
[1]  + suspended  sleep 1000
[2]  - running    nohup sleep 2000

$ bg %1
[1]  - 18653 continued  sleep 1000

$ jobs
[1]  - running    sleep 1000
[2]  + running    nohup sleep 2000

$ kill -STOP %1
[1]  + 18653 suspended (signal)  sleep 1000

$ jobs
[1]  + suspended (signal)  sleep 1000
[2]  - running    nohup sleep 2000

$ kill -SIGHUP %1
[1]  + 18653 hangup     sleep 1000

$ jobs
[2]  + running    nohup sleep 2000

$ kill -SIGHUP %2

$ jobs
[2]  + running    nohup sleep 2000

$ kill %2
[2]  + 18745 terminated  nohup sleep 2000

$ jobs 

```

### 终端多路复用

终端多路复用使我们可以分离当前终端会话并在将来重新连接。这让您操作远端设备时的工作流大大改善，避免了 `nohup` 和其他类似技巧的使用。

现在最流行的终端多路器是 [`tmux`](http://man7.org/linux/man-pages/man1/tmux.1.html)。

*   **会话** - 每个会话都是一个独立的工作区，其中包含一个或多个窗口
    *   `tmux` 开始一个新的会话
    *   `tmux new -s NAME` 以指定名称开始一个新的会话
    *   `tmux ls` 列出当前所有会话
    *   在 `tmux` 中输入 `<C-b> d`（detach），将当前会话分离
    *   `tmux a`（attach）重新连接最后一个会话。您也可以通过 `-t` 来指定具体的会话
*   **窗口** - 相当于编辑器或是浏览器中的标签页，从视觉上将一个会话分割为多个部分
    *   `<C-b> c` 创建一个新的窗口，使用 `<C-d>`关闭
    *   `<C-b> N` 跳转到第 _N_ 个窗口，注意每个窗口都是有编号的
    *   `<C-b> p`（previous）切换到前一个窗口
    *   `<C-b> n`（next）切换到下一个窗口
    *   `<C-b> ,` 重命名当前窗口
    *   `<C-b> w` 列出当前所有窗口
*   **面板** - 像 vim 中的分屏一样，面板使我们可以在一个屏幕里显示多个 shell
    *   `<C-b> "` 水平分割
    *   `<C-b> %` 垂直分割
    *   `<C-b> <方向>` 切换到指定方向的面板，<方向> 指的是键盘上的方向键
    *   `<C-b> z`（zoom）切换当前面板的缩放
    *   `<C-b> [` 开始往回卷动屏幕。您可以按下空格键来开始选择，回车键复制选中的部分
    *   `<C-b> <空格>` 在不同的面板排布间切换

扩展阅读： [这里](https://www.hamvocke.com/blog/a-quick-and-easy-guide-to-tmux/) 是一份 `tmux` 快速入门教程， [而这一篇](http://linuxcommand.org/lc3_adv_termmux.php) 文章则更加详细，它包含了 `screen` 命令。您也许想要掌握 [`screen`](http://man7.org/linux/man-pages/man1/screen.1.html) 命令，因为在大多数 UNIX 系统中都默认安装有该程序。

### 别名

```bash
# colorls
source $(dirname $(gem which colorls))/tab_complete.sh
alias ls=colorls
alias l="ls -lh"
alias ll="ls -lAh"
alias la="ls -lah"

alias hz="history | fzf"
alias mv="mv -i"
alias cp="cp -i"
alias mkdir="mkdir -p"

# To ignore an alias run it prepended with 
\ls
# Or disable an alias altogether with unalias
unalias la

# To get an alias definition just call it with alias
alias l
# Will print l='ls -lh'
```

### 配置文件（Dotfiles）

*   [shell startup scripts](https://blog.flowblok.id.au/2013-02/shell-startup-scripts.html)
*   [guide to dotfiles on github](https://dotfiles.github.io/tutorials/)
*   [example popular dotfiles](https://github.com/mathiasbynens/dotfiles)

管理配置文件的一个方法是，把它们集中放在一个文件夹中，例如 `~/.dotfiles/`，并使用版本控制系统进行管理，然后通过脚本将其 **符号链接** 到需要的地方。这么做有如下好处：

*   **安装简单**: 如果您登录了一台新的设备，在这台设备上应用您的配置只需要几分钟的时间；
*   **可以执行**: 您的工具在任何地方都以相同的配置工作
*   **同步**: 在一处更新配置文件，可以同步到其他所有地方
*   **变更追踪**: 您可能要在整个程序员生涯中持续维护这些配置文件，而对于长期项目而言，版本历史是非常重要的

一些技巧：

```bash
if [[ "$(uname)" == "Linux" ]]; then {do_something}; fi

# 使用和 shell 相关的配置时先检查当前 shell 类型
if [[ "$SHELL" == "zsh" ]]; then {do_something}; fi

# 您也可以针对特定的设备进行配置
if [[ "$(hostname)" == "myServer" ]]; then {do_something}; fi

# Test if ~/.aliases exists and source it
if [ -f ~/.aliases ]; then
    source ~/.aliases
fi
```

### 远端设备

#### SSH (Secure Shell)

```bash
# 连接设备
ssh foo@bar.mit.edu 
ssh foobar@192.168.1.42
# 如果存在配置文件，可以简写
ssh bar

# 执行命令
# 在本地查询远端 ls 的输出
ssh foobar@server ls | grep PATTERN
# 在远端对本地 ls 输出的结果进行查询
ls | ssh foobar@server grep PATTERN
```

#### SSH 密钥

基于密钥的验证机制使用了密码学中的公钥，我们只需要向服务器证明客户端持有对应的私钥，而不需要公开其私钥。这样您就可以避免每次登录都输入密码的麻烦了秘密就可以登录。

```bash
ssh-keygen -t ed25519 -C "_your_email@example.com_"
# If you are using a legacy system that doesn't support the Ed25519 algorithm, use:
ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
```

生成的 **id_rsa** 和 **id_rsa.pub** 两个文件（或者 **id_ed25519** 和 **id_ed25519.pub**），分别为的**私钥**和**公钥**。私钥等效于你的密码，所以一定要好好保存它。要检查您是否持有某个密钥对的密码并验证它，您可以运行 `ssh-keygen -y -f /path/to/key`。

`ssh` 会查询 `.ssh/authorized_keys` 来确认那些用户可以被允许登录。您可以通过下面的命令将一个公钥拷贝到这里：

```bash
cat .ssh/id_ed25519.pub | ssh foobar@remote 'cat >> ~/.ssh/authorized_keys' 
```

如果支持 `ssh-copy-id` 的话，可以使用下面这种更简单的解决方案：

```bash
ssh-copy-id -i .ssh/id_ed25519.pub foobar@remote
```

#### 通过 SSH 复制文件

使用 ssh 复制文件有很多方法：

*   `ssh+tee`, 最简单的方法是执行 `ssh` 命令，然后通过这样的方法利用标准输入实现 `cat localfile | ssh remote_server tee serverfile`。回忆一下，[`tee`](http://man7.org/linux/man-pages/man1/tee.1.html) 命令会将标准输出写入到一个文件；
*   [`scp`](http://man7.org/linux/man-pages/man1/scp.1.html) ：当需要拷贝大量的文件或目录时，使用`scp` 命令则更加方便，因为它可以方便的遍历相关路径。语法如下：`scp path/to/local_file remote_host:path/to/remote_file`；
*   [`rsync`](http://man7.org/linux/man-pages/man1/rsync.1.html) 对 `scp` 进行来改进，它可以检测本地和远端的文件以防止重复拷贝。它还可以提供一些诸如符号连接、权限管理等精心打磨的功能。甚至还可以基于 `--partial`标记实现断点续传。`rsync` 的语法和`scp`类似。

#### 端口转发

**本地端口转发** ![Local Port Forwarding](https://i.stack.imgur.com/a28N8.png%C2%A0 "本地端口转发")

**远程端口转发** ![Remote Port Forwarding](https://i.stack.imgur.com/4iK3b.png%C2%A0 "远程端口转发")

常见的情景是使用本地端口转发，即远端设备上的服务监听一个端口，而您希望在本地设备上的一个端口建立连接并转发到远程端口上。例如，我们在远端服务器上运行 Jupyter notebook 并监听 `8888` 端口。 然后，建立从本地端口 `9999` 的转发，使用 `ssh -L 9999:localhost:8888 foobar@remote_server` 。这样只需要访问本地的 `localhost:9999` 即可。

#### SSH 配置

使用 `~/.ssh/config` 文件来创建别名，类似 `scp`、`rsync`和`mosh`的这些命令都可以读取这个配置并将设置转换为对应的命令行选项。

```
Host vm
    User foobar
    HostName 172.16.174.141
    Port 2222
    IdentityFile ~/.ssh/id_ed25519
    LocalForward 9999 localhost:8888

# 在配置文件中也可以使用通配符
Host *.mit.edu
    User foobaz 
```

服务器侧的配置通常放在 `/etc/ssh/sshd_config`。您可以在这里配置免密认证、修改 shh 端口、开启 X11 转发等等。也可以为每个用户单独指定配置。

#### 杂项

连接远程服务器的一个常见痛点是遇到由关机、休眠或网络环境变化导致的掉线。如果连接的延迟很高也很让人讨厌。[Mosh](https://mosh.org/)（即 mobile shell ）对 ssh 进行了改进，它允许连接漫游、间歇连接及智能本地回显。

有时将一个远端文件夹挂载到本地会比较方便， [sshfs](https://github.com/libfuse/sshfs) 可以将远端服务器上的一个文件夹挂载到本地，然后您就可以使用本地的编辑器了。

### Shell & 框架

常见的 Shell：

- [bash](http://www.gnu.org/software/bash/)
- [zsh](https://www.zsh.org/)
- [fish](https://fishshell.com/)

常见的 Shell 框架：

- [oh-my-zsh](https://ohmyz.sh/)
- [prezto](https://github.com/sorin-ionescu/prezto)

### 终端模拟器

一些经典的模拟器：

- [xterm](https://invisible-island.net/xterm/)
- [GNOME Terminal](https://wiki.gnome.org/Apps/Terminal)
- [Konsole](https://konsole.kde.org/)
- [Xfce Terminal](https://docs.xfce.org/apps/terminal/start)
- [urxvt](http://software.schmorp.de/pkg/rxvt-unicode.html)
- [Terminator](https://gnometerminator.blogspot.com/)

一些新兴的模拟器（通常具有更好的性能，例如下面两个具有 GPU 加速）：

- [Alacritty](https://github.com/jwilm/alacritty)
- [kitty](https://sw.kovidgoyal.net/kitty/)

## 课后练习

### 任务控制

#### 习题 1

```
$ sleep 1000
^Z
[1]  + 689 suspended  sleep 1000

$ sleep 2000                                                                   
^Z
[2]  + 697 suspended  sleep 2000

$ jobs
[1]  - suspended  sleep 1000
[2]  + suspended  sleep 2000

$ bg %1
[1]  - 689 continued  sleep 1000

$ jobs
[1]  - running    sleep 1000
[2]  + suspended  sleep 2000

$ pgrep -af "sleep 1"
689 sleep 1000

$ pkill -f "sleep 1"
[1]  - 689 terminated  sleep 1000

$ jobs
[2]  + suspended  sleep 2000

$ pkill -f "sleep 2"

$ jobs
[2]  + suspended  sleep 2000

$ pkill -9 -f "sleep 2"
[2]  + 697 killed     sleep 2000

$ jobs

```

参见 `man kill`，默认发送的信号是 TERM。`-9` 等价于 `-SIGKILL` 或者 `-KILL`

#### 习题 2

```
$ sleep 10 &
[1] 1121

$ pgrep sleep | wait ; ls
[1]  + 1121 done       sleep 10

   Nothing to show here
```

```bash
$ pidwait() {
    wait $1
    echo "done"
    eval $2
}

$ sleep 10 &
[1] 1420

$ pidwait 1420 "ls"
[1]  + 1420 done       sleep 10
done

   Nothing to show here
```

# L6 - 版本控制 (Git)

## 笔记

### Git 的数据模型

Git 通过一系列快照来管理其历史记录。快照则是被追踪的最顶层的树。可以认为 `git commit` 会创建一个快照。

```
type object = blob | tree | commit

// 文件就是一组数据
type blob = array<byte>

// 一个包含文件和目录的目录
type tree = map<string, tree | file>

// 每个提交都包含一个父辈，元数据和顶层树
type commit = struct {
    parent: array<commit>
    author: string
    message: string
    snapshot: tree
}

// 还有引用（reference），比如
// HEAD, master, origin/HEAD, origin/master
// 都是引用
// 引用是指向提交的指针，与对象不同的是，它是可变的（引用可以被更新，指向新的提交）
```

实际上，Git 在储存数据时，所有的对象都会基于它们的SHA-1 hash进行寻址。Blobs、trees 和 commits 都一样，它们都是对象。当它们引用其他对象时，它们并没有真正的在硬盘上保存这些对象，而是仅仅保存了它们的哈希值作为引用。例如，上面为代码中的 `parent: array<commit>` 其实际上不是一个 commit 数组，而是一个哈希值数组，这些哈希值指向真正的对象，也就是一些 commits。

```
objects = map<string, object>

def store(object):
    id = sha1(object)
    objects[id] = object

def load(id):
    return objects[id]
```

例如，`git cat-file -p 698281b`（ 698281b 是某个tree，也就是某个文件夹的哈希值的一部分前缀）的结果是：

```
100644 blob 4448adbf7ecd394f42ae135bbeed9676e894af85    baz.txt
040000 tree c68d233a33c5c06e0340e4c224f0afca87c8ce87    foo 
```

而 `git cat-file -p 4448adb`（ 4448adb 是baz.txt 的哈希值的一部分前缀）的结果即为 baz.txte 的内容。

### Git 的命令行接口

**历史**:

*   `git log --all --graph --decorate --oneline`：可视化历史记录（有向无环图），zsh 的 git 插件定义了很多别名，比如该命令的别名是 gloga，去掉 --all 的别名是 glog
*   `git diff <filename>`：显示与上一次提交之间的差异
*   `git diff <old-revision> [<new-revision>] <filename>`：显示某个文件两个版本之间的差异，new-revision 默认是 HEAD
*   `git diff --cached <filename>`：不加 cached 标识的 diff 的意思是显示尚未暂存的改动，加了之后是查看已暂存的将要添加到下次提交里的内容

**修改、撤销和合并**：

*   `git add -p`：交互式暂存，例如交互过程中可以按 s 键进行 split，对文件中各个地方的改动分别选择暂存与否
*   `git checkout -- <file>`：丢弃（尚未暂存的）修改
*   `git reset [<tree-ish>] <file>`：取消暂存，把文件从暂存区放回工作区，<tree-ish> 默认为 HEAD
*   `git reset [--soft | --mixed [-N] | --hard | --merge | --keep] [<commit>]`：（见下图）撤销 commit，把 commit 放回暂存区（soft），或放回工作区（mixed），或丢弃（hard），本质是对 HEAD 的移动
*   `git reset [--soft | --mixed [-N] | --hard] HEAD^`，上一条的特例，比较常用
*   `git rebase <branch>`：在一个过时的分支上面开发的时候，执行 `rebase` 以此同步 `master` 分支最新变动
*   `git rebase -i HEAD~n`：交互式变基，可用于修改 commit 信息，合并 commit 等等
*   `git mergetool`：使用工具来处理合并冲突
*   `git stash`：把工作区暂存起来，允许你切换到其他分支，博主有时候在错误的分支上进行了修改，会用这个命令把修改暂存起来，然后换到正确的工作分支后使用 `git stash pop`

[![y1zWCV.png](https://s3.ax1x.com/2021/02/04/y1zWCV.png)](https://imgchr.com/i/y1zWCV)

**远端操作**：

*   `git clone --shallow`：克隆仓库，但是不包括版本历史信息
*   `git remote add <name> <url>`：添加一个远端
*   `git push <remote> <local branch>:<remote branch>`：将对象传送至远端并更新远端引用
*   `git branch --set-upstream-to=<remote>/<remote branch>`：创建本地和远端分支的关联关系

**其他**：

*   `.gitignore`: [指定](https://git-scm.com/docs/gitignore) 故意不追踪的文件
*   `git blame`：查看最后修改某行的人
*   `git bisect`：通过二分查找搜索历史记录
*   `git init --bare`：几乎用不到，在课程视频中，讲师在某一个空文件夹中使用该命令，将该文件夹作为 remote，然后将一个已有的仓库 push 到该文件夹
*   `git config --global core.excludesfile ~/.gitignore_global`：在 `~/.gitignore_global` 中创建全局忽略规则
*   [Removing sensitive data from a repository](https://docs.github.com/en/github/authenticating-to-github/removing-sensitive-data-from-a-repository)

### 杂项

*   **图形用户界面**: Git 的 [图形用户界面客户端](https://git-scm.com/downloads/guis) 有很多，但是我们自己并不使用这些图形用户界面的客户端，我们选择使用命令行接口
*   **Shell 集成**: 将 Git 状态集成到您的 shell 中会非常方便。([zsh](https://github.com/olivierverdier/zsh-git-prompt),[bash](https://github.com/magicmonty/bash-git-prompt))。[Oh My Zsh](https://github.com/ohmyzsh/ohmyzsh)这样的框架中一般以及集成了这一功能
*   **编辑器集成**: 和上面一条类似，将 Git 集成到编辑器中好处多多。[fugitive.vim](https://github.com/tpope/vim-fugitive) 是 Vim 中集成 GIt 的常用插件
*   **工作流**:我们已经讲解了数据模型与一些基础命令，但还没讨论到进行大型项目时的一些惯例 ( 有[很多](https://nvie.com/posts/a-successful-git-branching-model/) [不同的](https://www.endoflineblog.com/gitflow-considered-harmful) [处理方法](https://www.atlassian.com/git/tutorials/comparing-workflows/gitflow-workflow))
*   **GitHub**: Git 并不等同于 GitHub。 在 GitHub 中您需要使用一个被称作[拉取请求（pull request）](https://help.github.com/en/github/collaborating-with-issues-and-pull-requests/about-pull-requests)的方法来向其他项目贡献代码
*   **Other Git 提供商**: GitHub 并不是唯一的。还有像[GitLab](https://about.gitlab.com/) 和 [BitBucket](https://bitbucket.org/)这样的平台。

### 资源

*   [Pro Git](https://git-scm.com/book/en/v2)，**强烈推荐**！学习前五章的内容可以教会您流畅使用 Git 的绝大多数技巧，因为您已经理解了 Git 的数据模型，后面的章节提供了很多有趣的高级主题（[Pro Git 中文版](https://git-scm.com/book/zh/v2)）
*   如何编写 [良好的提交信息](https://tbaggery.com/2008/04/19/a-note-about-git-commit-messages.html)
*   Git 是一个 [高度可定制的](https://git-scm.com/docs/git-config) 工具
*   [Oh Shit, Git!?!](https://ohshitgit.com/) ，简短的介绍了如何从 Git 错误中恢复；
*   [Git for Computer Scientists](https://eagain.net/articles/git-for-computer-scientists/) ，简短的介绍了 Git 的数据模型
*   [Git from the Bottom Up](https://jwiegley.github.io/git-from-the-bottom-up/) 详细的介绍了 Git 的实现细节，而不仅仅局限于数据模型
*   [How to explain git in simple words](https://smusamashah.github.io/blog/2017/10/14/explain-git-in-simple-words)
*   [Learn Git Branching](https://learngitbranching.js.org/) 通过基于浏览器的游戏来学习 Git

## 课后习题

### 习题 2

是谁最后修改来 `README.md`文件？

```
$ git log --all -n1 --pretty=format:"%an" README.md
```

最后一次修改 `_config.yml` 文件中 `collections:` 行时的提交信息是什么？

```
$ git blame _config.yml | grep "collections:" | head -n1 | awk '{print $1}' | sed -E "s/\^//" | xargs git show

# OR

$ git blame _config.yml | grep "collections:" | head -n1 | awk '{print $1}' | sed -E "s/\^//" | xargs git log -n1 --pretty=format:"%s%n%n%b"
```

# L8 - 元编程

## 笔记

元编程通常又指 [用于操作程序的程序](https://en.wikipedia.org/wiki/Metaprogramming)，讲座中讨论的更多是关于开发**流程**。

### 构建系统

“构建系统”帮助我们执行一系列的“构建过程”。构建过程包括：目标（targets），依赖（dependencies），规则（rules）。您必须告诉构建系统您具体的构建目标，系统的任务则是找到构建这些目标所需要的依赖，并根据规则构建所需的中间产物，直到最终目标被构建出来。

理想的情况下，如果目标的依赖没有发生改动，并且我们可以从之前的构建中复用这些依赖，那么与其相关的构建规则并不会被执行。

`make` 是最常用的构建系统之一，您会发现它通常被安装到了几乎所有基于UNIX的系统中。`make` 的教程可以参考阮一峰的这篇文章：[Make 命令教程](https://www.ruanyifeng.com/blog/2015/02/make.html)。

其他常见的构建系统/工具：

- C 与 C++：Cmake，可以参考 [CMake 入门实战](https://www.hahack.com/codes/cmake/)
- Java：Maven，Ant，Gradle
- 前端开发：Grunt，Gulp，Webpack
- Ruby：Rake
- Rust：Cargo


### 依赖管理

#### 软件仓库

- Ubuntu：可以通过 `apt` 这个工具来访问 Ubuntu 软件包仓库
- CentOS，Redhat：通过 `yum` 这个工具来访问软件仓库
- Archlinux/Manjaro：通过 `pacman` 工具访问 Archlinux 软件仓库和 Arch 用户软件仓库（AUR，Arch User Repository）
- Ruby：通过 `gem` 工具访问 RubyGems
- Python：通过 `pip` 工具访问 Pypi

#### 版本号

不同项目所用的版本号其具体含义并不完全相同，但是一个相对比较常用的标准是[语义版本号](https://semver.org/)，这种版本号具有不同的语义，它的格式是这样的：major.minor.patch（主版本号.次版本号.补丁号）。相关规则有：

*   如果新的版本没有改变 API，请将补丁号递增；
*   如果您添加了 API 并且该改动是向后兼容的，请将次版本号递增；
*   如果您修改了 API 但是它并不向后兼容，请将主版本号递增。

这样做有很多好处，例如如果我们的项目是基于您的项目构建的，那么只要最新版本的主版本号只要没变就是安全的，次版本号不低于之前我们使用的版本即可。换句话说，如果我依赖的版本是`1.3.7`，那么使用`1.3.8`、`1.6.1`，甚至是`1.3.0`都是可以的。如果版本号是 `2.2.4` 就不一定能用了，因为它的主版本号增加了。

### 持续集成系统

持续集成，或者叫做 CI 是一种雨伞术语（umbrella term），它指的是那些“当您的代码变动时，自动运行的东西”，可以认为是一种云端构建系统。

市场上有很多提供各式各样 CI 工具的公司，例如 Travis CI、Azure Pipelines 和 GitHub Actions。

它们使用方法大同小异：在代码仓库中添加一个文件（recipe），在其中编写规则，规则包括 events 和 actions。

最常见的规则是：如果有人提交代码，执行测试套。当这个事件被触发时，CI 提供方会启动一个（或多个）虚拟机，执行您制定的规则，并且通常会记录下相关的执行结果。您可以进行某些设置，这样当测试套失败时您能够收到通知或者当测试全部通过时，您的仓库主页会显示一个徽标。

Github 还有一个维护依赖关系的 CI 工具 [Dependabot](https://dependabot.com/)。

GitHub Pages 是一个很好的例子。Pages 在每次`master`有代码更新时，会执行 Jekyll 博客软件，然后使您的站点可以通过某个 GitHub 域名来访问。对于我们来说这些事情太琐碎了，我现在我们只需要在本地进行修改，然后使用 git 提交代码，发布到远端。CI 会自动帮我们处理后续的事情。

### 测试

*   测试套（Test suite）：所有测试的统称
*   单元测试（Unit test）：一个“微型测试”，用于对某个封装的特性进行测试
*   集成测试（Integration test）: 一个“宏观测试”，针对系统的某一大部分进行，测试其不同的特性或组件是否能协同工作。
*   回归测试（Regression test）：用于保证之前引起问题的 bug 不会再次出现
*   模拟（Mocking）: 使用一个假的实现来替换函数、模块或类型，屏蔽那些和测试不相关的内容。例如，您可能会“模拟网络连接” 或 “模拟硬盘”

## 课后练习

### 习题 1

一些有用的 make [构建目标](https://www.gnu.org/software/make/manual/html_node/Standard-Targets.html#Standard-Targets)（例如本题用到了 [phony](https://www.gnu.org/software/make/manual/html_node/Phony-Targets.html)）。

```makefile
.PHONY: clean
clean:
      git ls-files -o | xargs rm
      # 这样还会删掉 gitignore 中的文件，例如一些编辑器配置文件
      # 此题也可以这样做
      # rm plot-*.png
      # rm paper.pdf
```

### 习题 3

```bash
#!/bin/sh
#
# An example hook script to verify what is about to be committed.
# Called by "git commit" with no arguments.  The hook should
# exit with non-zero status after issuing an appropriate message if
# it wants to stop the commit.

# Redirect output to stderr.
exec 1>&2

if (! make)
then
    cat <<\EOF
Error: make failed.

Excuting 'make paper.pdf'.
EOF
    make paper.pdf
    exit 1
fi
```

### 习题 4

基于 [GitHub Pages](https://help.github.com/en/actions/automating-your-workflow-with-github-actions) 创建任意一个可以自动发布的页面。添加一个[GitHub Action](https://github.com/features/actions) 到该仓库，对仓库中的所有 shell 文件执行 `shellcheck`([方法之一](https://github.com/marketplace/actions/shellcheck))。

### 习题 5

[构建属于您的](https://help.github.com/en/actions/automating-your-workflow-with-github-actions/building-actions) GitHub action，对仓库中所有的`.md`文件执行[`proselint`](http://proselint.com/) 或 [`write-good`](https://github.com/btford/write-good)，在您的仓库中开启这一功能，提交一个包含错误的文件看看该功能是否生效。

# L9 - 安全和密码学

## 笔记

- 2019 年本讲的内容为与 2020 年的普通，标题为 [安全与隐私](https://missing-semester-cn.github.io/2019/security/)，更注重于计算机用户可以如何增强隐私保护和安全
- 相关课程：计算机系统安全 ([6.858](https://css.csail.mit.edu/6.858/))
- 相关课程：密码学 ([6.857](https://courses.csail.mit.edu/6.857/)以及6.875)
- [不要试图创造或者修改加密算法](https://www.schneier.com/blog/archives/2015/05/amateurs_produc.html)
- [Cryptographic Right Answers](https://latacora.micro.blog/2018/04/03/cryptographic-right-answers.html): 解答了在一些应用环境下“应该使用什么加密？”的问题

### 熵

[熵](https://en.wikipedia.org/wiki/Entropy_(information_theory))(Entropy) 度量了不确定性并可以用来决定密码的强度。

熵的单位是 _比特_ 。对于一个均匀分布的随机离散变量，熵等于 `log_2(所有可能的个数，即 n)`。 扔一次硬币的熵是1比特。掷一次（六面）骰子的熵大约为2.58比特。

使用多少比特的熵取决于应用的威胁模型。大约40比特的熵足以对抗在线穷举攻击（受限于网络速度和应用认证机制）。而对于离线穷举攻击（主要受限于计算速度），一般需要更强的密码 (比如80比特或更多)。

### 散列函数

[密码散列函数](https://en.wikipedia.org/wiki/Cryptographic_hash_function) (Cryptographic hash function) 可以将任意大小的数据映射为一个固定大小的输出。散列函数具有如下特性：

*   确定性（deterministic）：对于不变的输入永远有相同的输出。
*   不可逆性（non-invertible）：对于`hash(m) = h`，难以通过已知的输出`h`来计算出原始输入`m`。
*   目标碰撞抵抗性/弱无碰撞（target collision resistant）：对于一个给定输入`m_1`，难以找到`m_2 != m_1`且`hash(m_1) = hash(m_2)`。
*   碰撞抵抗性/强无碰撞（collision resistant）：难以找到一组满足`hash(m_1) = hash(m_2)`的输入`m_1, m_2`（该性质严格强于目标碰撞抵抗性）。


[SHA-1](https://en.wikipedia.org/wiki/SHA-1)是Git中使用的一种散列函数，Linux 下有 `sha1sum` 工具。

虽然SHA-1还可以用于特定用途，但它已经[不再被认为](https://shattered.io/)是一个强密码散列函数。参照[密码散列函数的生命周期](https://valerieaurora.org/hash.html)这个表格了解一些散列函数是何时被发现弱点及破解的。

#### 密码散列函数的应用

*   Git中的内容寻址存储(Content addressed storage)：[散列函数](https://en.wikipedia.org/wiki/Hash_function)是一个宽泛的概念（存在非密码学的散列函数），那么Git为什么要特意使用密码散列函数？
    *   普通的散列函数没有无碰撞性，Git 使用密码散列函数，来确保分布式版本控制系统中的两个不同数据不会有相同的摘要信息（例如两个内容不同的 commit 不应该有相同的哈希值）。
*   文件的信息摘要(Message digest)：例如下载文件时，对比下载下来的文件的哈希值和官方公布的哈希值是否相同来判断文件是否损坏或者被篡改。
*   [承诺机制](https://en.wikipedia.org/wiki/Commitment_scheme)(Commitment scheme)：假设你要猜我在脑海中想的一个随机数字，我先告诉你该数字的哈希值，然后你猜数字，我再告诉你正确答案，看你是否猜对，这时你可以通过先前公布的哈希值来确认我没有作弊。

### 密钥生成函数

[密钥生成函数](https://en.wikipedia.org/wiki/Key_derivation_function) (Key Derivation Functions)与密码散列函数类似，用以产生一个固定长度的密钥。但是为了对抗穷举法攻击，密钥生成函数通常较慢。

#### 密码生成函数的应用

- 将其结果作为其他加密算法的密钥，例如对称加密算法
- 数据库中保存的用户密码为密文
    - 针对每个用户随机生成一个[盐](https://en.wikipedia.org/wiki/Salt_(cryptography))，并存储盐，以及密钥生成函数对连接了盐的明文密码生成的哈希值 `KDF(password + salt)`。
    - 在验证登录请求时，使用输入的密码连接存储的盐重新计算哈希值`KDF(input + salt)`，并与存储的哈希值对比。
    - **盐**（Salt），在[密码学](https://zh.wikipedia.org/wiki/%E5%AF%86%E7%A0%81%E5%AD%A6 "密码学")中，是指在[散列](https://zh.wikipedia.org/wiki/%E6%95%A3%E5%88%97 "散列")之前将散列内容（例如：密码）的任意固定位置插入特定的字符串。这个在散列中加入字符串的方式称为“加盐”。
    - 在大部分情况，盐是不需要保密的。
    - 通常情况下，当字段经过散列处理，会生成一段散列值，而散列后的值一般是无法通过特定算法得到原始字段的。但是某些情况，比如一个大型的[彩虹表](https://zh.wikipedia.org/wiki/%E5%BD%A9%E8%99%B9%E8%A1%A8 "彩虹表")，通过在表中搜索该SHA-1值，很有可能在极短的时间内找到该散列值对应的真实字段内容。
    - 加盐可以避免用户的短密码被彩虹表破解，也可以保护在不同网站使用相同密码的用户。

### 对称加密

```
keygen() -> key  （这是一个随机方法，例如使用 KDF(passphrase)）

encrypt(plaintext: array<byte>, key) -> array<byte>  (输出密文)
decrypt(ciphertext: array<byte>, key) -> array<byte>  (输出明文)
```

加密方法`encrypt()`输出的密文`ciphertext`很难在不知道`key`的情况下得出明文`plaintext`。

[AES](https://en.wikipedia.org/wiki/Advanced_Encryption_Standard) 是现在常用的一种对称加密系统。在 Linux 下可以使用 openssl 工具：

```bash
openssl aes-256-cbc -salt -in {源文件名} -out {加密文件名}
openssl aes-256-cbc -d -in {加密文件名} -out {解密文件名}
```

### 非对称加密

非对称加密的“非对称”代表在其环境中，使用两个具有不同功能的密钥： 一个是私钥(private key)，不向外公布；另一个是公钥(public key)，公布公钥不像公布对称加密的共享密钥那样可能影响加密体系的安全性。

```
keygen() -> (public key, private key)  (这是一个随机方法)

encrypt(plaintext: array<byte>, public key) -> array<byte>  (输出密文)
decrypt(ciphertext: array<byte>, private key) -> array<byte>  (输出明文)

sign(message: array<byte>, private key) -> array<byte>  (生成签名)
verify(message: array<byte>, signature: array<byte>, public key) -> bool  (验证签名是否是由和这个公钥相关的私钥生成的)
```

非对称的加密/解密方法和对称的加密/解密方法有类似的特征。
信息在非对称加密中使用 _公钥_ 加密， 且输出的密文很难在不知道 _私钥_ 的情况下得出明文。

在不知道 _私钥_ 的情况下，不管需要签名的信息为何，很难计算出一个可以使 `verify(message, signature, public key)` 返回为真的签名。

#### 非对称加密的应用

*   [PGP电子邮件加密](https://en.wikipedia.org/wiki/Pretty_Good_Privacy)：用户可以将所使用的公钥在线发布，比如：PGP密钥服务器或 [Keybase](https://keybase.io/)。任何人都可以向他们发送加密的电子邮件。
*   聊天加密：像 [Signal](https://signal.org/)、[Telegram](https://telegram.org/) 和 [Keybase](https://keybase.io/) 使用非对称密钥来建立私密聊天。
*   软件签名：Git 支持用户对提交(commit)和标签(tag)进行GPG签名。任何人都可以使用软件开发者公布的签名公钥验证下载的已签名软件。

#### 密钥分发

非对称加密面对的主要挑战是，如何分发公钥并对应现实世界中存在的人或组织。

- Signal的信任模型：信任用户第一次使用时给出的身份(trust on first use)，支持线下(out-of-band)面对面交换公钥（Signal里的safety number）。
- PGP使用的是[信任网络](https://en.wikipedia.org/wiki/Web_of_trust)。
- Keybase主要使用[社交网络证明 (social proof)](https://keybase.io/blog/chat-apps-softer-than-tofu)。

### 案例分析

- 密码管理器
    - 如 [KeePassXC](https://keepassxc.org/)。
- [两步验证](https://en.wikipedia.org/wiki/Multi-factor_authentication)(2FA)（多重身份验证 MFA）
    - 要求用户同时使用密码（“你知道的信息”）和一个身份验证器（“你拥有的物品”，比如[YubiKey](https://www.yubico.com/)）来消除密码泄露或者[钓鱼攻击](https://en.wikipedia.org/wiki/Phishing)的威胁。
- 全盘加密
    - 对笔记本电脑的硬盘进行全盘加密是防止因设备丢失而信息泄露的简单且有效方法。
    - Linux的 [cryptsetup + LUKS](https://wiki.archlinux.org/index.php/Dm-crypt/Encrypting_a_non-root_file_system)
    - Windows的 [BitLocker](https://fossbytes.com/enable-full-disk-encryption-windows-10/)
    - macOS的 [FileVault](https://support.apple.com/en-us/HT204837)
- 聊天加密
    - 获取联系人的公钥非常关键。为了保证安全性，应使用线下方式验证用户公钥，或者信任用户提供的社交网络证明。
- SSH
    - `ssh-keygen` 命令会生成一个非对称密钥对。公钥最终会被分发，它可以直接明文存储。但是为了防止泄露，私钥必须加密存储。
    - `ssh-keygen` 命令会提示用户输入一个密码，并将它输入 KDF 产生一个密钥。最终，`ssh-keygen` 使用对称加密算法和这个密钥加密私钥。
    - 当服务器已知用户的公钥（存储在`.ssh/authorized_keys`文件中），尝试连接的客户端可以使用非对称签名来证明用户的身份——这便是[挑战应答方式](https://en.wikipedia.org/wiki/Challenge%E2%80%93response_authentication)。 简单来说，服务器选择一个随机数字发送给客户端。客户端使用用户私钥对这个数字信息签名后返回服务器。服务器随后使用保存的用户公钥来验证返回的信息是否由所对应的私钥所签名。这种验证方式可以有效证明试图登录的用户持有所需的私钥。

## 课后练习

### 熵

1. `Entropy = log_2(100000^5) = 83`
2. `Entropy = log_2((26+26+10)^8) = 48`
3. 第一个更强。
4. 分别需要 31.7 万亿年和 692 年。

### 非对称加密

1.  `ssh-keygen -r ed25519 -o -C "your_email"`
    [生成 SSH 公钥](https://git-scm.com/book/zh/v2/服务器上的-Git-生成-SSH-公钥)
    [How To Set Up SSH Keys](https://www.digitalocean.com/community/tutorials/how-to-set-up-ssh-keys-2)
2.  [How To Use GPG to Encrypt and Sign Messages](https://www.digitalocean.com/community/tutorials/how-to-use-gpg-to-encrypt-and-sign-messages)
    [GPG入门教程](http://www.ruanyifeng.com/blog/2013/07/gpg.html)
3.  给Anish发送一封加密的电子邮件（[Anish的公钥](https://keybase.io/anish)）。
4.  `git commit -S`命令签名一个Git commit
    `git show --show-signature`命令验证 commit 的签名
    `git tag -s`命令签名一个Git标签
    `git tag -v`命令验证标签的签名
    [对提交签名](https://docs.github.com/cn/github/authenticating-to-github/signing-commits)

# L10 - 大杂烩

## 笔记

### 修改键位映射

修改键位映射可以通过软件或者硬件（支持定制固件的键盘）实现。软件可以实现更复杂的修改例如对不同的键盘或软件保存专用的映射配置。

下面是一些修改键位映射的软件：

*   macOS - [karabiner-elements](https://pqrs.org/osx/karabiner/), [skhd](https://github.com/koekeishiya/skhd) 或者 [BetterTouchTool](https://folivora.ai/)
*   Linux - [xmodmap](https://wiki.archlinux.org/index.php/Xmodmap) 或者 [Autokey](https://github.com/autokey/autokey)
*   Windows - 控制面板，[AutoHotkey](https://www.autohotkey.com/) 或者 [SharpKeys](https://www.randyrants.com/category/sharpkeys/)
*   QMK - 如果你的键盘支持定制固件，[QMK](https://docs.qmk.fm/) 可以直接在键盘的硬件上修改键位映射。比如我正在用的宁芝的 atom66（宁芝看到记得给我打钱）。

### 守护进程（daemon）

- 在后台保持运行，不需要用户手动运行或者交互
- 以守护进程运行的程序名一般以 `d` 结尾
- SSH 服务端 `sshd`，用来监听传入的 SSH 连接请求并对用户进行鉴权
- Linux 中的 `systemd`（the system daemon），用来配置和运行守护进程
    - 使用 `systemctl` 命令来与 systemd 交互
    - `systemctl enable|disable|start|stop|restart|status`
- 如果只是想定期运行一些程序，可以直接使用 [`cron`](http://man7.org/linux/man-pages/man8/cron.8.html)。它是一个系统内置的，用来执行定期任务的守护进程。

下面的配置文件使用了 systemd 来运行一个 Python 程序，`systemd` 配置文件的详细指南可参见 [freedesktop.org](https://www.freedesktop.org/software/systemd/man/systemd.service.html)。

```ini
# /etc/systemd/system/myapp.service
[Unit]
# 配置文件描述
Description=My Custom App
# 在网络服务启动后启动该进程
After=network.target

[Service]
# 运行该进程的用户
User=foo
# 运行该进程的用户组
Group=foo
# 运行该进程的根目录
WorkingDirectory=/home/foo/projects/mydaemon
# 开始该进程的命令
ExecStart=/usr/bin/local/python3.7 app.py
# 在出现错误时重启该进程
Restart=on-failure

[Install]
# 相当于Windows的开机启动。即使GUI没有启动，该进程也会加载并运行
WantedBy=multi-user.target
# 如果该进程仅需要在GUI活动时运行，这里应写作：
# WantedBy=graphical.target
# graphical.target在multi-user.target的基础上运行和GUI相关的服务
```

### FUSE

**用户空间文件系统**（**F**ilesystem in **Use**rspace，简称**FUSE**）是一个面向[类Unix](https://zh.wikipedia.org/wiki/%E7%B1%BBUnix "类Unix")计算机操作系统的软件接口，它使无特权的用户能够无需编辑[内核](https://zh.wikipedia.org/wiki/%E5%86%85%E6%A0%B8 "内核")代码而创建自己的[文件系统](https://zh.wikipedia.org/wiki/%E6%96%87%E4%BB%B6%E7%B3%BB%E7%BB%9F)。目前[Linux](https://zh.wikipedia.org/wiki/Linux "Linux")通过[内核模块](https://zh.wikipedia.org/w/index.php?title=%E5%86%85%E6%A0%B8%E6%A8%A1%E5%9D%97&action=edit&redlink=1 "内核模块（页面不存在）")对此进行支持。

一些有趣的 FUSE 文件系统包括：

*   [sshfs](https://github.com/libfuse/sshfs)：一个将所有文件系统操作都使用 SSH 转发到远程主机，由远程主机处理后返回结果到本地计算机的虚拟文件系统。这个文件系统里的文件虽然存储在远程主机，对于本地计算机上的软件而言和存储在本地别无二致
*   [rclone](https://rclone.org/commands/rclone_mount/)：将 Dropbox、Google Drive、Amazon S3、或者 Google Cloud Storage 一类的云存储服务挂载为本地文件系统
*   [gocryptfs](https://nuetzlich.net/gocryptfs/)：覆盖在加密文件上的文件系统。文件以加密形式保存在磁盘里，但该文件系统挂载后用户可以直接从挂载点访问文件的明文
*   [kbfs](https://keybase.io/docs/kbfs)：分布式端到端加密文件系统。在这个文件系统里有私密（private），共享（shared），以及公开（public）三种类型的文件夹
*   [borgbackup](https://borgbackup.readthedocs.io/en/stable/usage/mount.html)：方便用户浏览删除重复数据后的压缩加密备份

### 备份

- 复制存储在同一个磁盘上的数据**不是备份**，因为这个磁盘是一个单点故障（single point of failure）
- 同步方案**不是备份**
    - 如 Dropbox 或者 Google Drive，当数据在本地被抹除或者损坏，同步方案可能会把这些“更改”同步到云端。
    - RAID 这样的磁盘镜像方案也不是备份。它不能防止文件被意外删除、损坏、或者被勒索软件加密。
- 不要盲目信任备份方案。用户应该经常检查备份是否可以用来恢复数据。
    - 云端应用的重大发展使得我们很多的数据只存储在云端。但用户应该有这些数据的离线备份。

有效备份方案的核心特性：

- 版本控制
- 删除重复数据
- 安全性（别人需要有什么信息或者工具才可以访问或者完全删除你的数据及备份）

该课程2019年关于备份的 [课堂笔记](https://missing-semester-cn.github.io/2019/backups)。

### API（应用程序接口）

- 大多数线上服务提供的 API 具有类似的格式。它们的结构化 URL 通常使用 `api.service.com` 作为根路径。
    - 例如可以发送一个 GET 请求（比如使用 `curl`）到[`https://api.weather.gov/points/42.3604,-71.094`](https://missing-semester-cn.github.io/2020/potpourri/%60https://api.weather.gov/points/42.3604,-71.094%60)来获取天气信息
- 通常这些返回都是 `JSON` 格式，你可以使用 [`jq`](https://stedolan.github.io/jq/) 等工具来选取需要的部分。
- 有些需要认证的 API 通常要求用户在请求中加入某种私密令牌（secret token）来完成认证。大多数 API 都会使用 [OAuth](https://www.oauth.com/)。
- [IFTTT](https://ifttt.com/) 这个网站可以将很多 API 整合在一起，让某 API 发生的特定事件触发在其他 API 上执行的任务。IFTTT 的全称 If This Then That 足以说明它的用法，比如在检测到用户的新推文后，自动发布在其他平台。

### 常见命令行标志参数及模式

*   `--help` 或 `-h` 或者类似的标志参数（flag）来显示简略用法
*   会造成不可撤回操作的工具一般会提供“空运行”（dry run）标志参数和“交互式”（interactive）标志参数
*   会造成破坏性结果的工具一般默认进行非递归的操作，但是支持使用“递归”（recursive）标志函数（通常是 `-r`）
*   `--version` 或者 `-V` 标志参数可以让工具显示它的版本信息
*   `--verbose` 或者 `-v` 标志参数来输出详细的运行信息。多次使用这个标志参数，比如 `-vvv`，可以让工具输出更详细的信息（经常用于调试）
*   `--quiet` 标志参数来抑制除错误提示之外的其他输出。
*   使用 `-` 代替输入或者输出文件名意味着工具将从标准输入（standard input）获取所需内容，或者向标准输出（standard output）输出结果，可以参考之前的笔记：[计算机教育中缺失的一课 - MIT - L4 - 数据整理](https://segmentfault.com/a/1190000039141914)
*   有的时候你可能需要向工具传入一个 _看上去_ 像标志参数的普通参数，这时候你可以使用特殊参数 `--` 让某个程序 _停止处理_ `--` 后面出现的标志参数以及选项（以 `-` 开头的内容）：
    * `rm -- -r` 会让 `rm` 将 `-r` 当作文件名；
    * `ssh machine --for-ssh -- foo --for-foo` 的 `--` 会让 `ssh` 知道 `--for-foo` 不是 `ssh` 的标志参数。

### 窗口管理器

大部分操作系统默认的窗口管理方式都是“拖拽”式的，这被称作堆叠式（floating/stacking）管理器。另外一种管理器是平铺式（tiling）管理器，其使用逻辑和 [tmux](https://github.com/tmux/tmux) 管理终端窗口的方式类似（参考之前的笔记：[计算机教育中缺失的一课 - MIT - L5 - 命令行环境](https://segmentfault.com/a/1190000039160431)），可以让我们在完全不使用鼠标的情况下使用键盘切换、缩放、以及移动窗口。

- Linux
    - [awesome](https://awesomewm.org/)
    - [i3](https://i3wm.org/)
- macOS
    - [yabai](https://github.com/koekeishiya/yabai)
    - [Divvy](https://mizage.com/divvy/)
- Windows
    - [FancyZones](https://docs.microsoft.com/en-us/windows/powertoys/fancyzones)

### VPN

关于这一部分，课程的 Lecture Note 写得已经十分简洁，直接摘录下来。

> VPN 现在非常火，但我们不清楚这是不是因为[一些好的理由](https://gist.github.com/joepie91/5a9909939e6ce7d09e29)。你应该了解 VPN 能提供的功能和它的限制。使用了 VPN 的你对于互联网而言，**最好的情况**下也就是换了一个网络供应商（ISP）。所有你发出的流量看上去来源于 VPN 供应商的网络而不是你的“真实”地址，而你实际接入的网络只能看到加密的流量。
>
> 虽然这听上去非常诱人，但是你应该知道使用 VPN 只是把原本对网络供应商的信任放在了 VPN 供应商那里——网络供应商 _能看到的_ ，VPN 供应商 _也都能看到_ 。如果相比网络供应商你更信任 VPN 供应商，那当然很好。反之，则连接VPN的价值不明确。机场的不加密公共热点确实不可以信任，但是在家庭网络环境里，这个差异就没有那么明显。
>
> 你也应该了解现在大部分包含用户敏感信息的流量已经被 HTTPS 或者 TLS 加密。这种情况下你所处的网络环境是否“安全”不太重要：供应商只能看到你和哪些服务器在交谈，却不能看到你们交谈的内容。
>
> 这一切的大前提都是“最好的情况”。曾经发生过 VPN 提供商错误使用弱加密或者直接禁用加密的先例。另外，有些恶意的或者带有投机心态的供应商会记录和你有关的所有流量，并很可能会将这些信息卖给第三方。找错一家 VPN 经常比一开始就不用 VPN 更危险。
>
> MIT 向有访问校内资源需求的成员开放自己运营的 [VPN](https://ist.mit.edu/vpn)。如果你也想自己配置一个 VPN，可以了解一下 [WireGuard](https://www.wireguard.com/) 以及 [Algo](https://github.com/trailofbits/algo)。

### Markdown

[Markdown](https://commonmark.org/help/) 是一个轻量化的标记语言（markup language），也致力于将人们编写纯文本时的一些习惯标准化。

### Hammerspoon (macOS 桌面自动化)

[Hammerspoon](https://www.hammerspoon.org/) 是面向 macOS 的一个桌面自动化框架。它允许用户编写和操作系统功能挂钩的 Lua 脚本，从而与键盘、鼠标、窗口、文件系统等交互。

*   [Getting Started with Hammerspoon](https://www.hammerspoon.org/go/)：Hammerspoon 官方教程
*   [Sample configurations](https://github.com/Hammerspoon/hammerspoon/wiki/Sample-Configurations)：Hammerspoon 官方示例配置
*   [Anish’s Hammerspoon config](https://github.com/anishathalye/dotfiles-local/tree/mac/hammerspoon)：讲师 Anish 的 Hammerspoon 配置

### 开机引导以及 Live USB

在计算机启动时，[BIOS](https://en.wikipedia.org/wiki/BIOS) 或者 [UEFI](https://en.wikipedia.org/wiki/Unified_Extensible_Firmware_Interface) 会在加载操作系统之前对硬件系统进行初始化，这被称为引导（booting）。在 BIOS 菜单中你可以对硬件相关的设置进行更改，也可以在引导菜单中选择从硬盘以外的其他设备加载操作系统——比如 Live USB。

[Live USB](https://en.wikipedia.org/wiki/Live_USB) 是包含了完整操作系统的闪存盘。Live USB 的用途非常广泛，包括：

*   作为安装操作系统的启动盘；
*   在不将操作系统安装到硬盘的情况下，直接运行 Live USB 上的操作系统；
*   对硬盘上的相同操作系统进行修复；
*   恢复硬盘上的数据。

Live USB 通过在闪存盘上 _写入_ 操作系统的镜像制作，写入不是单纯的往闪存盘上复制 `.iso` 文件。可以使用 [UNetbootin](https://unetbootin.github.io/)、[Rufus](https://github.com/pbatard/rufus)、[UltraISO](https://www.ultraiso.com/) 等 Live USB 写入工具制作。

### 虚拟技术

[虚拟机](https://en.wikipedia.org/wiki/Virtual_machine)（Virtual Machine）以及如[容器化](https://en.wikipedia.org/wiki/OS-level_virtualization)（containerization）(亦称操作系统层虚拟化）等工具可以帮助你模拟一个包括操作系统的完整计算机系统。

- [Vagrant](https://www.vagrantup.com/)：一个构建和配置虚拟开发环境的工具。它支持用户在配置文件中写入比如操作系统、系统服务、需要安装的软件包等描述，然后使用 `vagrant up` 命令在各种环境（VirtualBox，KVM，Hyper-V等）中启动一个虚拟机。
- [Docker](https://www.docker.com/)：一个使用容器化概念的与 Vagrant 类似的工具，在后端服务的部署中应用广泛。
- [VPS](https://en.wikipedia.org/wiki/Virtual_private_server)（虚拟专用服务器）:将一台[服务器](https://zh.wikipedia.org/wiki/%E6%9C%8D%E5%8A%A1%E5%99%A8 "服务器")分割成多个虚拟专用服务器的服务
    - 实现VPS的技术分为容器技术和虚拟机技术
    - 国外的大型云主机服务商有 [Amazon AWS](https://aws.amazon.com/)，[Google Cloud](https://cloud.google.com/)，[DigitalOcean](https://www.digitalocean.com/)
    - [CSAIL OpenStack instance](https://tig.csail.mit.edu/shared-computing/open-stack/)：供 MIT CSAIL 的成员免费申请使用的虚拟机

### 交互式记事本编程

> [交互式记事本](https://en.wikipedia.org/wiki/Notebook_interface)可以帮助开发者进行与运行结果交互等探索性的编程。现在最受欢迎的交互式记事本环境大概是 [Jupyter](https://jupyter.org/)。它的名字来源于所支持的三种核心语言：Julia、Python、R。[Wolfram Mathematica](https://www.wolfram.com/mathematica/) 是另外一个常用于科学计算的优秀环境。


# L11 - Q&A

## 笔记

### OS 学习资料

- [MIT’s 6.828](https://pdos.csail.mit.edu/6.828/) - 研究生阶段的操作系统课程，带你实现一个 OS
- 现代操作系统 - Andrew S. Tanenbaum，对各种概念做了系统的讲解
- FreeBSD的设计与实现（ _The Design and Implementation of the FreeBSD Operating System_ ） - 关于FreeBSD OS 不错的资源(注意，FreeBSD OS 不是 Linux)
- [用 Rust 写操作系统](https://os.phil-opp.com/)

### `source script.sh` 和 `./script.sh`

不同点在于哪个会话执行这个命令。 对于 `source` 命令来说，命令是在当前的bash会话中执行的，因此当 `source` 执行完毕，对当前环境的任何更改（例如更改目录或是定义函数）都会留存在当前会话中。 单独运行 `./script.sh` 时，当前的bash会话将启动新的bash会话（实例），并在新实例中运行命令 `script.sh`。

### 性能分析工具

- 最简单但是有效的：在代码中添加打印运行时间的语句，通过二分法逐步定位到花费时间最长的代码段。
- Valgrind 的 [Callgrind](http://valgrind.org/docs/manual/cl-manual.html) 可以让你运行程序并计算所有的时间花费以及所有调用堆栈。然后，它会生成带注释的代码版本，其中包含每行花费的时间。注意它不支持线程。
- 特定的编程语言可能会有自带的或者特定的第三方的分析工具
- 用于用户程序内核跟踪的[eBPF](http://www.brendangregg.com/blog/2019-01-01/learn-ebpf-tracing.html)、低级的性能分析工具 [`bpftrace`](https://github.com/iovisor/bpftrace)：分析系统调用中的等待时间，因为有时代码中最慢的部分是系统等待磁盘读取或网络数据包之类的事件

### 浏览器插件

- [uBlock Origin](https://github.com/gorhill/uBlock)：[用途广泛（wide-spectrum）](https://github.com/gorhill/uBlock/wiki/Blocking-mode)的拦截器
    - [简易模式（easy mode）](https://github.com/gorhill/uBlock/wiki/Blocking-mode:-easy-mode)
    - [中等模式（medium mode）](https://github.com/gorhill/uBlock/wiki/Blocking-mode:-medium-mode)
    - [强力模式（hard mode）](https://github.com/gorhill/uBlock/wiki/Blocking-mode:-hard-mode)
- [Stylus](https://github.com/openstyles/stylus/)：自定义CSS样式加载到网站
    - 不要使用Stylish，它会[窃取浏览记录](https://www.theregister.co.uk/2018/07/05/browsers_pull_stylish_but_invasive_browser_extension/)
    - 可以使用其他用户编写并发布在[userstyles.org](https://userstyles.org/)中的样式
- 全页屏幕捕获：完整的页面截屏
    - 内置于 Firefox 和 [Chrome 扩展程序](https://chrome.google.com/webstore/detail/full-page-screen-capture/fdpohaocaechififmbbbbbknoalclacl?hl=en)中
- [多账户容器](https://addons.mozilla.org/en-US/firefox/addon/multi-account-containers/)：将Cookie分为“容器”从而允许你以不同的身份浏览web网页并且/或确保网站无法在它们之间共享信息
- 密码集成管理器：可以使用火狐和谷歌自带的密码管理器，也可以使用第三方专门的密码管理器，通常拥有更强大的功能。使用密码管理器也可以防止钓鱼网站，因为管理器不会在假冒的域名站点弹出自动填充。

### 数据整理工具

- 在数据整理一讲中提到的分别针对 JSON 和 HTML 的 jq 和 pup
- Perl 语言**非常**擅长处理文本，值得进行学习，但它是一种“Write Only”的语言，因为写出来的代码可读性非常差
- Vim 也可以用来整理数据，例如利用 Vim 的宏
- Python 的 [pandas](https://pandas.pydata.org/) 库是整理表格数据（或类似格式）的好工具
- [Pandoc](https://www.pandoc.org/)：a universal document converter，可以在各种文档之间进行转换，HTML、Markdown、LaTex、docx、XML 等等
- R语言（一种有争议的[不好](http://arrgh.tim-smith.us/)的语言）作为一种主要用于统计分析的编程语言，在管道的最后一步（比如画图展示）非常有用，其绘图库 [ggplot2](https://ggplot2.tidyverse.org/) **非常**强大。

### Docker 与虚拟机的区别

- 虚拟机会执行整个的 OS 栈，包括内核（即使这个内核和主机内核相同）
- 容器与主机分享内核（在Linux环境中，有LXC机制来实现），当然容器内部感知不到，仍像是在使用自己的硬件启动程序
- 容器的隔离性较弱而且只有在主机运行相同的内核时才能正常工作
    - 例如，如果你在macOS 上运行 Docker，Docker 需要启动 Linux虚拟机去获取初始的 Linux内核，这样的开销仍然很大
- Docker 是容器的特定实现，它是为软件部署而定制的，有一些奇怪之处，例如
    - 在默认情况下，Docker 容器没有任何形式的持久化存储，关闭之后数据消失，而与之对应虚拟机通常有一个虚拟硬盘文件保存在主机上

### 如何选择操作系统

- 可以使用任何 Linux 发行版（Distro）去学习 Linux 与 UNIX 的特性和其内部工作原理
- 发行版之间的根本区别是发行版如何处理软件包更新
    - Arch Linux 采用滚动更新策略，用了最前沿的软件包（bleeding-edge），但软件可能并不稳定
    - Debian，CentOS 或 Ubuntu LTS 的更新策略要保守得多，因此更加稳定
- Mac OS 是介于 Windows 和 Linux 之间的一个操作系统
    - Mac OS 是基于BSD 而不是 Linux
- 另一种值得体验的是 FreeBSD
    - 与 Linux 相比，BSD 生态系统的碎片化程度要低得多，并且说明文档更加友好
- 作为程序员，你为什么还在用 Windows？除非你开发 Windows 应用程序或需要使用某些 Windows 系统更好支持的功能（例如对游戏的驱动程序支持）（有被冒犯到。。。）
- 对于双系统，我们认为最有效的是 macOS 的 bootcamp，因为长期来看，任何其他组合都可能会出现问题，尤其是当你结合了其他功能比如磁盘加密

### Vim 还是 Emacs

Emacs 不使用 vim 的模式编辑，但是这些功能可以通过 Emacs 插件比如 [Evil](https://github.com/emacs-evil/evil) 或 [Doom Emacs](https://github.com/hlissner/doom-emacs) 来实现。 Emacs的优点是可以用 Lisp 语言进行扩展（Lisp 比 vim 默认的脚本语言 vimscript 要更好用）。

### 机器学习应用的技巧

- 机器学习应用需要进行许多实验，探索数据，可以使用 Shell 轻松快速地搜索这些实验结果，并且以合理的方式汇总。
- 使用课程中介绍过的数据整理的工具，通过使用 JSON 文件记录实验的所有相关参数，让你的实验结果变得井井有条且可复现。
- 如果不使用集群提交 GPU 作业，那你应该研究如何使这些过程自动化。

### 两步验证（2FA）

最简单的情形是可以通过接收手机的 SMS 来实现（尽管 SMS 2FA 存在 [已知问题](https://www.kaspersky.com/blog/2fa-practical-guide/24219/)）。我们推荐使用 [YubiKey](https://www.yubico.com/) 之类的 [U2F](https://en.wikipedia.org/wiki/Universal_2nd_Factor) 方案。

### 如何选择浏览器

- Chrome 的渲染引擎是 Blink，JS 引擎是 V8。
- Firefox 的渲染引擎是 Gecko，JS 引擎是 SpiderMonkey。
- 其他浏览器大多都是 Chrome 的变种，用着 Chromium 内核，运行着同样的引擎，例如新版的 Microsoft Edge。至于 Safari 则基于 WebKit(与Blink类似的引擎)。这些浏览器仅仅是更糟糕的 Chrome 版本。
- Firefox 与 Chrome 的在各方面不相上下，但在隐私方面更加出色。
- Firefox 正在使用 Rust 重写他们的渲染引擎，名为 Servo。
- 一款目前还没有完成的叫 Flow 的浏览器，它实现了全新的渲染引擎，有望比现有引擎速度更快。

