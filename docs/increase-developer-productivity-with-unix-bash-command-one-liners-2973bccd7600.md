# 使用 Unix Bash 命令一行程序提高开发人员的工作效率

> 原文：<https://itnext.io/increase-developer-productivity-with-unix-bash-command-one-liners-2973bccd7600?source=collection_archive---------0----------------------->

## 用一些魔法击中你的终端。有用的 OSX/Linux/Unix bash 终端命令…适合一行！

![](img/cd7dc6529372774defc693848584eb53.png)

# 和你的终端交朋友

如果你像我一样，当涉及到你的软件开发实践时，你总是在寻找可以提高**效率**、**简化、**和**自动化**的方法。总是在想有没有一种**更快的**或者**更容易的**方式，而*对于大多数事情*，**是有的！**

**然而**，我们真正关心的只是那些**耗时**和**频繁使用**的任务的自动化(或提高效率)。让它们更有效率**和更容易**记忆**，让你更有可能拿出你的终端并**使用它们，**而不是加载一个 **GUI** 应用程序和**手动**编辑或删除(你的宝贵时间)。**

## **终端不喜欢多行命令**

**我花了一段时间和我的终端交朋友，但是我们**终于**到了那里。我意识到，如果你正在执行的命令是**多行**、**复杂**和**难以编写**，那么使用终端是一件**痛苦的事情。****

**我最终疏忽的另一个原因是，一开始我不知道很多命令。**

> **…因为我肯定没有人知道刚出娘胎的巴什。我们都从某处开始。**

**因此，当我第一次开始学习编写代码时，我通过命令行**完成的每个任务通常不是**最快的或最有效的**的**，它只是我在堆栈溢出上发现的**第一个方法**，它工作时没有考虑到某些事情已经严重出错。**

**我花了一些时间养成了一个习惯**用我的终端作为**解决问题**的首选解决方案。找到高效的**单行**命令对养成这个习惯大有帮助。我对某个特定命令是一个被证明有效的方法**越有**信心**，就越容易**记住**和**重用**。**

**随后，我开始积累一些我认为合适的命令，作为命令行中常见问题/任务的成熟解决方案。**

# **在变魔术之前，给初学者一个小提示**

****这些一行程序中的一些**不是很容易记住，所以要真正获得更复杂的**命令所带来的**生产力优势**，您应该习惯于创建**可执行的 bash** (。sh)脚本。****

**这允许您通过键入`cmd`而不是`./cmd.sh`或`bash cmd.sh`(其中`cmd.sh`是包含命令的文件)来**执行**bash 文件，其中**包含**一行程序。**

**举个简单的例子，假设你想找到你的“秘密文件”(一个简单的例子)。您可以创建一个可执行文件…**

## **代码**

**`**$** find . -name secret_file`**

## **将代码保存到文件 find_secret.sh**

**`**$** echo "find . -name secret_file" > find_secret.sh`**

## **使可执行**

**`**$** chmod +x find_secret.sh`**

## **符号链接可执行文件，所以它在您的路径中**

**`**$** sudo ln -s ~/find_secret.sh /usr/local/bin/find_secret`**

**注意:您可以随意命名符号链接。我只是省略了。易于使用的 sh 扩展。**

**但是你可能不想每次都硬编码整个命令并寻找“秘密文件”…**

**所以您可以`**$** echo "find . -name $1" > findn.sh` ( **find_n** 一个更贴切的名字，因为`-name`是动态的)允许您将一个变量字符串作为参数($1)传递给执行。**

**通过对附加步骤的相关修改…**

**`chmod +x findn.sh && sudo ln -s ~/findn.sh /usr/local/bin/findn`**

## **现在使用它🚀**

**`**$** find_secret`或`**$** findn "secret_file"`**

**正如 Fred Abood 提到的，您可以为常用命令使用别名(这更适合这个特殊的例子)。**

```
**$** echo "alias findn='find . -name'" >> ~/.bash_profile && source ~/.bash_profile
```

> ****source** 是必需的如果你想在同一个 shell 实例中执行 findn，它会强制 shell 重新加载~/。bash_profile**
> 
> **替换 **~/。bash_profile** for **~/。bashrc** 如果你喜欢，或者 **~/。zshrc** 如果使用 [zsh](https://ohmyz.sh)**

**这意味着你可以找到你的“秘密文件”,别名如下…**

```
**$** findn "secret_file"
```

# **魔力**

**我假设您熟悉 Unix 和 Bash 命令的基础，比如`ls`、`rm`、`curl`等。所以我不会用最基本的来烦你。这些最多是中级的命令。**

**让我们从简单的开始…**

## **删除文件中的重复行**

```
**$** awk ‘!seen[$0]++’ file.txt | tee output.txt
```

> **使用 [awk](https://www.geeksforgeeks.org/awk-command-unixlinux-examples/) 将`file.txt` 的唯一行输出到文件`output.txt`中，使用[三通](https://shapeshed.com/unix-tee/)。**

## **将外部资源下载到本地目录**

```
**$** wget -O ./image.png "https://resourcewebsite.com/path/to/resource.png"
```

> **使用 [wget](https://linux.die.net/man/1/wget) 将外部资源作为文件`image.png`下载到当前工作目录中。**

## **找到那些过大的文件**

```
**$** find . -type f -size +500M
```

> **使用[查找](https://shapeshed.com/unix-find/)在当前工作目录中搜索大于指定大小的文件。**

## **查找包含文本的文件**

```
**$** grep -lir "some text" *
```

> **使用 [grep](https://www.geeksforgeeks.org/grep-command-in-unixlinux/) 递归搜索所有文件中的文本，忽略大小写，输出匹配的文件。**

## **哪个进程正在使用宝贵的内存**

```
**$** ps aux | awk '{if ($5 != 0 ) print $2,$5,$6,$11}' | sort -k2n
```

> **使用 [ps](https://shapeshed.com/unix-ps/) 输出内存使用率最高的进程的排序列表。**

## **显示 10 个最大的打开文件**

```
**$** lsof / | awk '{ if($7 > 1048576) print $7/1048576 "MB" " " $9 " " $1 }' | sort -n -u | tail
```

> **使用 [lsof](https://www.tutorialspoint.com/unix_commands/lsof.htm) 列出打开的文件，按大小排序。**

## **打印最常用的命令**

```
**$** cat ~/.bash_history | tr "\|\;" "\n" | sed -e "s/^ //g" | cut -d " " -f 1 | sort | uniq -c | sort -n | tail -n 15
```

> **使用[卡特彼勒](https://www.cyberciti.biz/faq/linux-unix-appleosx-bsd-cat-command-examples/)输出最常用的命令(通过。bash_history 文件)。**

## **运行命令并自动复制到剪贴板(MacOS)**

```
**$** echo "This echo can be replaced with a more useful cmd" | tee >(pbcopy)
```

> **使用[回显](https://www.computerhope.com/unix/uecho.htm)，将文本打印到标准输出，然后通过管道(|)将输出通过`pbcopy`重定向到剪贴板。**

## **向 SSH 代理添加一个 SSH 密钥**

```
**$** eval "$(ssh-agent -s)" && ssh-add -K ~/.ssh/id_rsa
```

> **使用 [eval](https://www.tutorialspoint.com/unix_commands/eval.htm) ，作为 shell 命令有效地执行参数。[带 ssh-add 的 ssh-agent](https://www.ssh.com/ssh/agent) 用于添加密钥。如果您的 ssh-agent 不记得您的 ssh 密钥，这很有用。**

## **生成随机的 32 个字符的密码**

```
**$** tr -dc 'a-zA-Z0-9~!@#$%^&*_()+}{?></";.,[]=-' < /dev/urandom | fold -w 32 | head -n 1
```

> **使用 [tr](https://www.geeksforgeeks.org/tr-command-in-unix-linux-with-examples/) ，使用`/dev/urandom`作为标准输入生成 32 个随机字符。**

## **运行最后一个命令**

```
**$** !! or **$** sudo !!
```

**额外收获:运行以' ln '开头的最后一个命令**

```
**$ !**ln
```

## **file.txt 中每行显示(n)个字符**

```
**$** xargs -n 3 ./file.txt
```

> **使用 [xargs](https://shapeshed.com/unix-xargs/) 显示文件中每行 3 个字符。**

## **查找并删除文件**

```
**$** rm `find . -name "*.html"` 
```

> **使用 [rm](https://www.computerhope.com/unix/urm.htm) 删除使用[命令替换](https://www.gnu.org/software/bash/manual/html_node/Command-Substitution.html)的`find . -name "*.html"`输出的文件。**

****或**正如 [Petru Cervac](https://medium.com/u/b8af0a59db94?source=post_page-----2973bccd7600--------------------------------) 指出的，`find`命令有一个`-delete`标志。所以同样的结果可以用..**

```
**$** find . -name "*.html" -delete
```

## **删除包含“secret”的行**

```
**$** sed "/secret/Id" filename.txt // case insensitive
**$** sed "/secret/d" filename.txt // case sensitive
```

> **使用 [sed](https://www.geeksforgeeks.org/sed-command-in-linux-unix-with-examples/) 从 filename.txt 中删除行**

## **打印包含“秘密”的行+(前 3 行+后 3 行)+颜色🌈**

```
**$** grep -C 3 'secret' ./file_with_secrets.txt --color
```

## **预先考虑&同时附加到每一行**

```
**$** sed -e 's/.*/PREFIX: & :SUFFIX/' /tmp/file
```

**用例:将行转换成逗号分隔的列表，例如一、二、三**

```
**$** sed -e "s/.*/&,/" test.txt | xargs
```

## **删除空行**

```
**$** sed -i '/^$/d'
```

## **从您的前置摄像头(或网络摄像头)拍摄截图**

```
**$** mplayer tv:// -tv driver=v4l2:width=640:height=480:device=/dev/video0 -fps 15 -vf screenshot
```

> **使用 [mplayer](http://www.mplayerhq.hu/DOCS/HTML/en/commandline.html) 截取屏幕截图，假设网络摄像头为`/dev/video0`，每秒 15 帧。**

## **将当前目录下文件名中的所有空格都改为下划线(没有人希望文件名中包含 spaces.txt)**

```
**$** for i in *; do mv "$i" ${i// /_};done
```

> **使用 [mv](https://shapeshed.com/unix-mv/) 和一个 for 循环来移动(有效地重命名)所有带空格的文件。**

## **将视频目录转换为 mp4**

```
**$** for INPUT in *.avi ; do echo "${INPUT%.avi}" ; done | xargs -i -P9  HandBrakeCLI -i "{}".avi -o "{}".mp4
```

> **对普通人来说可能没那么有用。使用[手刹](https://handbrake.fr/docs/en/latest/cli/cli-options.html)转换。avi 视频到. mp4。**

## **想要更多吗？**

 **[## bashoneliners.com

### 一个实用的、解释清楚的 Bash 一行程序和 shell 脚本技巧、窍门、GNU Linux 代码片段的集合…

www.bashoneliners.com](http://www.bashoneliners.com/oneliners/popular/)**  **[## bash-one line

### 我很高兴你在这里！几年前我在研究生物信息学，对那些单个单词的狂欢感到惊讶…

onceupon.github.io](https://onceupon.github.io/Bash-Oneliner/)** **[](https://www.geeksforgeeks.org/sed-command-in-linux-unix-with-examples/) [## Linux/Unix 中的 Sed 命令及示例

### UNIX 中的 SED 命令代表流编辑器，它可以对文件执行许多功能，如搜索、查找和…

www.geeksforgeeks.org](https://www.geeksforgeeks.org/sed-command-in-linux-unix-with-examples/) [](https://www.geeksforgeeks.org/awk-command-unixlinux-examples/) [## Unix/Linux 中的 AWK 命令及示例

### Awk 是一种用于操作数据和生成报告的脚本语言。awk 命令编程语言…

www.geeksforgeeks.org](https://www.geeksforgeeks.org/awk-command-unixlinux-examples/) 

认为这些俏皮话是驯服的吗？有更好的吗？

## 请随意评论或留言，我会将它们添加到列表中！**