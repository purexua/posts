## bash shell 别名 alias

> [!TIP]
>
> Author: purexua
>
> Date: 2024-2-2
>
> Tag: **Linux** | **bash/shell** 

### 什么是 alias

> 一个bash shell的别名就是命令的快捷方式。
>
> 别名命令允许用户通过输入一个单词来启动任何命令或一组命令（包括选项和文件名）。使用别名命令显示所有定义的别名列表。您可以将用户定义的别名添加到~/.bashrc文件中。通过使用这些别名，您可以减少输入时间，聪明地工作，并提高命令提示符下的生产力。

### 如何列出bash别名

输入以下别名命令：

`alias`

示例输出：

```bash
alias cp='cp -i'
alias egrep='egrep --color=auto'
alias fgrep='fgrep --color=auto'
alias grep='grep --color=auto'
alias l.='ls -d .* --color=auto'
alias ll='ls -l --color=auto'
alias ls='ls --color=auto'
alias mv='mv -i'
alias rm='rm -i'
alias which='alias | /usr/bin/which --tty-only --read-alias --show-dot --show-tilde'
```

*默认情况下，alias命令显示为当前用户定义的别名列表。*

### 如何定义或创建一个bash shell的别名

使用以下语法创建别名：

```bash
alias name=value
alias name='command'
alias name='command arg1 arg2'
alias name='/path/to/script'
alias name='/path/to/script.pl arg1'
```

在这个例子中，通过输入以下命令并按下回车键，为常用的清屏命令创建别名 c：

`alias c='clear'`

然后，为了清除屏幕，你只需要输入字母"c"并按下[回车]键：

`c`

### 如何临时禁用别名？

创建一个名为vnstat的别名：

> alias c=clear
>
> c

现在暂时禁用vnstat别名，输入：

> \c

另一个选项是输入完整的命令路径：

> /usr/bin/clear

### 如何删除/移除一个bash别名

您需要使用名为 unalias 的命令来删除别名。其语法如下：

> unalias aliasname
> unalias c

您还需要使用文本编辑器 从 ~/.bashrc 文件中删除别名。

***保存并关闭文件。系统范围的别名（即所有用户的别名）可以放在/etc/bashrc文件中。请注意，alias命令内置于各种shell中，包括ksh、tcsh/csh、ash、bash等。***

### 关于特权访问的说明

您可以在 ~/.bashrc 中添加以下代码：

```bash
# if user is not root, pass all commands via sudo #
if [ $UID -ne 0 ]; then
    alias reboot='sudo reboot'
    alias update='sudo apt-get upgrade'
fi
```

### 关于特定操作系统的别名的说明

您可以使用case语句将以下代码添加到 ~/.bashrc 中：

```bash
### Get os name via uname ###
_myos="$(uname)"
 
### add alias as per os using $_myos ###
case $_myos in
   Linux) alias foo='/path/to/linux/bin/foo';;
   FreeBSD|OpenBSD) alias foo='/path/to/bsd/bin/foo' ;;
   SunOS) alias foo='/path/to/sunos/bin/foo' ;;
   *) ;;
esac
```

> [!NOTE]
> 如果你知道并使用其他可以减少输入的bash/ksh/csh别名，请在评论区分享。