---
title: macOS 终端(Terminal) 命令行 教程
catalog: true
date: 2017-09-22 23:10:52
subtitle:
header-img: mac.jpg
tags:
- iOS开发
---
## 一、为什么要使用命令行？
* 许多功能在图形界面不提供，只有通过命令行来实现。
* Finder 会隐藏许多你不太会需要的文件，而 Command line 则可以访问所有文件。
* Command line 可以通过 SSH 远程访问你的 Mac 或者 Linux。
* 普通用户可以通过 `sudo` 命令获得 root 用户权限。
* 如果你开启手动输入用户名登陆模式，登陆时在用户名处输入 `>console` 可以直接进入命令行界面。随后你仍然需要登录到一个账户。

## 二、初识Command Line
* 许多命令会花费一些时间来执行，然而这中间不会给出任何提示或者进度条。一般结束后会出现一个`用户名$`的标记。如果没有出现，那么说明最后一条命令正在执行。
* 一条命令包括 Command Name、Options、Arguments、Extras 四个部分，后三个部分是可选的。
Options 部分用 `-` 作为前导符。许多命令的 Options 部分只包含单个字母，这时可以合并。例如，`ls -lA` 和 `ls -l -A` 是等效的。
Arguments 部分用来细化这个命令或指定这个命令具体的实施对象，
Extras 部分则用来进一步实现其他功能。
* 例如：下列命令包含前三个部分，用于强行删除 Junk.app 这个程序。
`$ rm -R /Applications/Junk.app`
* 如果你输入了错误的命令，系统会返回一些错误信息。但是系统却不会阻止你做任何事（例如删除整个用户文件夹）。

## 三、 关于 man 命令
虽然有上千条命令，每条命令还有许多可选参数和具体的使用方式，但是你却不需要记住这些命令。你只需要记住一个：`man`。

`man` 其实就是一个命令使用指南，在命令行中输入 `man command-name`  即可获取。例如，你想知道 `ls` 这个命令怎么使用，输入 `man ls` 即可进入使用指南页面。

使用指南往往很长，所以你可以使用▲（上箭头）或▼（下箭头）来上下移动，使用`空格`来翻页，输入 `/` 和关键字来按照关键字搜索，按 `Q` 来退出使用指南页面。

输入 `man -k` 和关键字来对整个使用指南数据库进行搜索。

## 四、 命令行，文件和路径
如果知道如何使用命令是掌握 Command line 的第一步，那么第二步就是学习如何在 Command line 中使用文件路径。这比使用 Finder 更加快捷。

**注意**：Command line 工具是大小写敏感的，并且对于文件名，必须包括扩展名。例如，你想找 iTunes 这个程序，输入 itunes 是无效的，必须输入 iTunes.app。

**两种路径：绝对路径 和 相对路径**
* 绝对路径：完整描述一个文件的位置，总是以斜杠 `/` 开头。例如 `/Users/michelle/Public/Drop Box`。
* 相对路径：是指相对于当前的目录，可以通过 `pwd` 查看当前目录。当新打开 Terminal 时，Command line 默认是 home folder 目录。这时上面例子文件夹的相对路径写作 Public/Drop Box。显然它从当前目录开始。和 HTML 类似，你也可以使用两个点 `..` 来代表父目录，这样就可以用相对路径表示上级或同级目录了。例如输入 `cd ..`，或者 `cd ../..`

**切换到其他路径和目录**

如果你想将当前 command line 会话切换到其他目录，需要用到三个命令：`pwd`、`ls` 和 `cd`。
* `pwd` 的含义是 “print working directory”，会显示当前目录的绝对路径。
* `ls` 的含义是 “list directory contents”，它会列出当前目录的内容。
* `cd` 的含义是 “change directory”，它会改变当前目录到指定的目录。如果不指定，则会返回 home folder。

**处理特殊字符**
如果目录中有特殊字符（空格，括号，引号，`[]`，`!`，`$`，`&`，`*`，`;`，`|`，`\`），那么直接输入空格会造成系统识别困难，必须使用特殊的语法来表示这些字符。例如上例中，空格前添加反斜杠 `\` 即可：`cd Punlic/Drop\ Box/`。除了反斜杠，也可以用引号的方法：`cd "Public/Drop Box"`。

也可以把文件从 Finder 拖到 Terminal 窗口来创建绝对路径，这会方便一些，因为上面提到的所有特殊字符在拖动后都会自动变成系统可识别的表示方法。

Tab Complete 是 Command line 中最能给你节省时间的特性之一，利用它的自动完成文件、目录名称功能还可以防止输入错误。使用 `cd` 进入你的 home folder，使用 `cd P` 命令，然后按下tab按键。可能会听到错误音，因为你的 home folder 内有多个 P 开头的文件夹。再按一次 tab，Terminal 将会为你列出 P 开头的两个文件夹：Public 和 Pictures。按 U，再按 tab，Terminal 则会自动为你补全 Public/。Tab complete 同样会处理那些特殊字符。注意，这会在末尾保留 `/` 符号，大部分时候这没问题，但如果出错，移除多余的 `/` 试一试。

另外，`~` 在command line 中可以代表当前用户的 home folder。例如 `~/Public/Drop\ Box/` 是合法的。

**查看隐藏文件**
为了简化工作，Command line 和 Finder 都会隐藏许多文件和文件夹。如果想查看隐藏文件，这在 Command line 中非常简单。首先，许多隐藏文件的隐藏是通过隐藏属性在 Finder 中隐藏的，而 Command line 会忽略这些属性，所以这些文件会在 Command line 中显示。另外，`ls` 命令会隐藏文件名以 `.` 开头的文件，但是这些文件却可以被显示出来，方法是利用 `-a` 选项。例如：`$ ls -la`，这里添加了 `-l` 选项，目的是控制输出格式。如果你注意输出内容的话，会发现还包括 `.` 和 `..` 两项，它们分别表示当前文件夹和父文件夹（如图）。如果你不想显示这两项，只需要把 `-a` 改成 `-A` 即可。

![Terminal ls -la命令](http://upload-images.jianshu.io/upload_images/2708793-9c31069f26f76822.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/540)

**前往其他卷**
在 Command line 中，系统卷（root volume）是由开始的一个正斜杠表示的。在 Command line 中其他卷看起来就在文件系统中一个叫做 Volumes 的文件夹中。下面的命令清晰地显示出这种逻辑关系：从 home folder 出发，最终前往一个叫 Macintosh HD 的卷

![Macintosh HD 卷](http://upload-images.jianshu.io/upload_images/2708793-9f26561a5e6b1710.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/340)


## 五、用Command-Line管理文件
**检视文件**
有许多基础命令用来定位、检视文件和文件夹，包括 `cat`, `less`, `which`, `file` 以及 `find`。别忘了可以利用 `man` 命令来查阅每个命令的使用指南。

**cat**
`cat` 是 “concatenate” 的意思，会按顺序读取文件并输出到 Terminal 窗口，语法为 `cat` 后接你需要查看的文件的路径。`cat` 命令也可以用 `>>` 来增加文本文件的内容，例如命令 `cat textOne.txt >> textTwo.txt`，会把 textOne.txt 的内容添加到 textTwo.txt 的结尾。这个 `>>` 就属于之前提到的“Extras”。

**less**
这个命令更适合用来查看长文本文件，因为它可以查找文本。语法为 `less` 后接文件路径，和 `cat` 一样。用 `less` 命令打开的文件其实和你查看命令使用指南的时候使用的是一个查看器，所以操作是相同的，同样可以使用▲（上箭头）或▼（下箭头）来上下移动文本，使用 `空格` 来翻页，输入 `/` 和关键字来按照关键字搜索，按 `Q` 来退出使用指南页面。除此之外，按 `V` 键来使用 `vi` 文本编辑器。

**which**
这个命令会定位某个命令的文件路径。它会告诉你你执行某个具体命令的时候，在使用哪个文件。语法为 `which` 后接某个命令。如图：
![终端 which 命令](http://upload-images.jianshu.io/upload_images/2708793-7d74e405e4e83f15.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/400)

**file**
这个命令会尝试根据文件的内容输出文件类型。如果一个文件缺失了扩展名，那么这个命令可能会非常有用。语法为 `file` 后接文件路径。如图，此例为一个 PNG 文件，还给出了它的尺寸、颜色数等信息。

![终端 file命令](http://upload-images.jianshu.io/upload_images/2708793-78cfa379ad9a28cf.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/540)

**find**
这个命令用来根据搜索关键词定位文件路径。 `find` 命令不使用 Spotlight 搜索服务，但是它允许你设置非常具体的搜索条件，以及通配符（稍后介绍）。语法为 `find` 后接搜索的起始路径，后接定义搜索的选项，后接搜索内容（包含在引号里）。例如：`find Desktop -name "*.png"`，意思是查询桌面上以 png 结尾的文件。

**注意**
如果你要搜索根目录，使用 `-x` 选项来避免搜索 /Volumes 文件夹。
如果想使用 Soptlight 搜索服务，使用 `mdfind` 命令后接搜索关键词即可。

**使用通配符（Wildcard Characters）**
* 星号（＊，Asterisk）——代表任何长度的任何字符。例如 `*.tiff` 代表所有格式为tiff的文件。
* 问号（?，Question mark）——代表任何单个字符。例如 `b?ok` 匹配 book 但是不匹配 brook。
* 方括号（[]，Square brackets）——定义一定范围的字符，例如 `[Dd]ocument` 匹配 Document 以及 document；`doc[1-9]` 匹配doc1, doc2, …, doc9。

**使用递归命令**
简单来说，递归命令可以允许命令不执行于一个特定文件，而是指定的路径下的所有文件。大多数命令包含一个 `-r` 或者 `-R` 选项，来设定你想递归地执行这个命令。例如下面的例子，添加 `-R` 后 `ls` 命令的执行方式：`ls -R Public`，这会列出 Public 中的所有文件，若有文件夹，则会继续列出文件夹中的内容。

**编辑文件和文件夹**
有许多基础的命令用来编辑文件和文件夹，包括 `mkdir`, `cp`, `mv`, `rm`, `rmdir` 以及 `vi`。下面我们来简要地介绍一下这些命令。

**mkdir**
“make diretory” 的缩写，用来创建文件夹，语法为 `mkdir` 后接新文件夹的目录。可以用 `-p` 选项，来一起创建路径中不存在的文件夹。

**cp**
“copy” 的缩写，用来把文件从一处复制到另一处。语法为 `cp` 后接原始路径，再接目标路径。如果你想复制整个文件夹和所有内容，需要添加 `-R` 选项。如果指定的目标路径不含文件名，则 `cp` 命令会按原名复制。如果指定的目标路径包括文件名，则会复制为你指定的文件名。如果仅指定新文件名，则会在原处以新名称创建文件副本。注意，系统会自动替换同名文件而不出现提示。

**mv**
“move”的缩写，用来移动文件。语法为 `mv` 后接原路径，再接新路径。`mv` 的指定路径规则与 `cp` 是一样的（如果仅指定新文件名，它就成了重命名命令）。

**rm**
“remove” 的缩写，会永久删除文件。注意，Command-line 中没有废纸篓。语法为 `rm` 后接文件路径。如果希望安全删除文件，可以使用 `srm` 命令。`rmdir` 和 `rm -R rmdir`是 “remove directory” 的缩写，这个命令会永久删除文件夹。语法为 `rmdir` 后接希望删除目录的路径。然而，`rmdir` 命令无法删除含有任何其他文件的文件夹，所以大多数情形下 `rmdir` 命令是不适用的。不过，你可以利用 `rm` 添加 `-R` 选项来删除文件夹及包含的所有文件。

**vi**
代表 “visual”（视觉的）。`vi` 是 Command line 中最常见的文本编辑器。用 `vi` 打开文本文件，只需要输入 `vi` 后接文件路径即可。macOS X 还提供了 nano，一个更加现代的文本编辑器。然而 `vi` 是默认的文本编辑器，所以掌握 `vi` 是很有用的。和 `less` 命令类似，`vi` 命令会占用整个 Terminal 空间来显示文件内容。打开后，会处于 “command模式”，`vi` 会等你输入一些预定义字符来告诉 `vi` 你想做什么。可以使用键盘上的箭头键单纯地浏览文件，想编辑时，按 A 或者 I 开始（进入编辑模式）。文字会插入到光标处。如果你想保存，需要先退出编辑模式，进入 command 模式，方法是按下 esc 键。回到 command 模式后，按住 shift 同时按两次 Z 来保存并退出。如果你不想保存，在 command 模式输入 `:quit!` 并按 return 直接退出。

## 六、用Command-Line管理系统
**使用su来切换用户**
**su**
命令代表 “substitute user identity”，允许你在命令行中切换到另一个用户账户。语法为 `su` 后接用户的短名称。然后会要求你输入密码（但是输入的时候不会显示）。执行完毕后，命令的前缀会改变，表示你拥有其他用户的权利。你可以利用 `who -m` 命令来验证当前登陆的身份。切换后，你会一直保持该用户身份，直至退出 Terminal 或者输入 `exit` 命令。

**关于sudo的使用**
* **sudo概述**
更强大的命令就是 `sudo`，代表 “substitute user do”，或者，更恰当地，“super user do”。用 `sudo` 执行一个命令会使用 root 账户权限。当然，使用之前需要 administrator 账户（管理员账户）的授权（输入密码）。
默认情况下，任何管理员账户都可以使用 `sudo` 来获取 root 权限，甚至当 root 账户在图形界面被禁用的情况下，`sudo` 依然有效。这个命令是很多情况下我们不得不使用 Terminal 的原因，同样也是给每个用户管理员身份的危险所在。不过，你可以调整 `sudo` 的配置文件，来限制它的使用。

```
$ cat secret.txt
cat: secret.txt: Permission denied
$ sudo cat secret.txt
Password:
This is the contents of the secret.txt text file that the user account renfei does not normally have access permissions to read. However, because he is an administrative user, she can use the sudo command to envoke root user access and read the contents of this file.
```

**提示**：如果由于你忘了使用 `sudo` 而导致命令行返回一个错误，只需输入 `sudo !!` 就可以用 `sudo` 来执行上一条指令。不恰当地使用 `sudo` 可以轻易破坏你的系统设置。命令行只会在第一次执行严重破坏性行为之前提示。

* **使用 sudo 切换 Shell**
如果你是一个管理员用户，你需要执行很多条需要 root 权限的命令，你可以临时切换整个命令行 shell 来取得 root 级别的访问权限。方法就是先输入 `sudo -s`，回车后再键入你的密码。

## 七、其他 Command-Line 技巧提示
* 输入命令 `open .` 可以打开当前的位置。
* 在 Terminal 的偏好里面可以设定它的外观和风格。
* 中止一个正在执行的命令，可以使用组合键 control + C。
* 在执行前编辑命令，只需要使用箭头和键盘上的其他字母。
* 没有输入任何命令时，你可以用▲和▼来浏览历史命令。同样可以编辑和再次执行。
* 你也可以使用history命令查看历史记录。
* 你可以使用组合键 control + L 清屏。


以上内容参考《Mac OS X Support Essentials》教程， Command Line 一节