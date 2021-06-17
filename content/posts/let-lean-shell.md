---
title: "一起来学 Shell 吧"
date: 2020-06-16T22:17:51+08:00
tags: ["theme"]
categories: ["starting"]
banner: "img/banners/banner-1.jpg"
---
it 
我们现在大部分开发现在都是使用macos进行开发. macos由于是基于Unix系统开发的. 所以系统本身很好的支持了bash.（虽然不同系统下有些地方有些差异）. 这就使我们可以很方便的使用shell来优化我们的工作效率. 

### sh, bash, zsh, fish 是什么, 有什么区别
我个人的理解呢, 是这样的. 他们都属于shell, 但是他们所支持的语法糖跟一些高级特性, 比如命令补全. 这些不一样.

其中比较流行的就是bash. 所以我们在看到shell脚本基本都能看到一行 `#!/bin/bash`. 用来指定此脚本使用特定的sh来执行

我个人平时办公最常用的是`zsh`. 当然我推荐新手也用这个. 这个对于学习跟办公来说效率是最高的.

其中 `sh` 的大小是最小的. 所以大部分linux发行版本内置都是 `sh`. 

如下我们可以看到. 除了 `sh` 才 31k . 其他外壳的sh都是几百k.

除了写一些嵌入式的时候需要考虑有限的存储跟内存用选用比较小的sh以外. 像我平时在公司的dockerfile都会特意安装一个bash

```bash
ls -lh /bin/  | grep sh
-r-xr-xr-x  1 root  wheel   609K Aug 11 04:56 bash
-rwxr-xr-x  1 root  wheel   517K Aug 11 04:56 csh
-rwxr-xr-x  1 root  wheel   108K Aug 11 04:56 dash
-r-xr-xr-x  1 root  wheel   1.2M Aug 11 04:56 ksh
-rwxr-xr-x  1 root  wheel    31K Aug 11 04:56 sh
-rwxr-xr-x  1 root  wheel   517K Aug 11 04:56 tcsh
-rwxr-xr-x  1 root  wheel   623K Aug 11 04:56 zsh
```

### 内置变量
shell跟大部分编程语言一样. 都有一些内置变量提供给开发者. 以方便实现某些功能. 由于内置变量特别多. 这里就说几个特别重要的

### `$?`
这个内置变量特别重要. 它指的是当前 terminal 最后一次运行都错误码. 除了0表示正常, 其余错误码都是对应程序有错误.

这也是写C 或者 一些CLI 工具的时候. 需要 `return 0` 或者 `exit(0);` 的原因. 这个变量存在的目的主要是方便程序和程序之间的通信.

比如做devops的时候. CI 需要知道当前pipeline是否执行成功. 就是使用的这个变量.

还有我们经常会看到这样的sh脚本

```bash
ls -lh /bin/  | grep sh | awk '{print $9}' | xargs echo
```

linux 管道使每个单一指责的命令工具很灵活的组合起来. 每次记录最后运行是否成功这个状态. 使每一个task 都能够在上一个task正常的执行完后再执行.

### `$$`, `$PPID`

`$$` 很容易理解. 就是获取当前脚本运行的pid. 

`$PPID` 就可以获取当前脚本的父级的pid. 如果当前是子任务的话.

### `$PWD`, `$HOME`

`$PWD` 可以获取当前所在目录

`$HOME` 则是获取根目录

```bash
➜ ~ echo $HOME
/Users/albert
➜  ~ echo $PWD
/Users/albert
```

### `$@`, `$0`, `$number`

`$@` 可以理解为 js 中的arguments.相当于获取当前脚本所有的参数. 

例如: 
```bash
ls 1 2 3 4
```

则 `$@` 就表示 `1 2 3 4`

`$0` 表示当前脚本名. 这里要注意的是. 我们调用脚本时使用的是什么路径. 这里就是什么路径

例如:
```bash
➜ bash ./1.sh
./1.sh
➜ bash $(pwd)/1.sh
/Users/albert/code/CodingMonkey/1.sh
```

`$number` 指的是 `$1, $2, $3 ... $n`. 这里指的就是脚本传入第n个参数.

### 声明变量
像所有其他编程语言一样，Bash 支持变量。这里说一下shell变量要注意的地方

变量赋值的语法非常严格，等号（=）两边不能有空格

像这样声明是会报语法错误的
```bash
#!/bin/bash
name = "Albert"
echo $name
```

正确的声明姿势
```bash
#!/bin/bash
name="Albert"
echo $name
```

### 流程控制 (if/else)
shell的if else也是需要特别注意的. 细讲的话一篇文章都讲不完. 所以这里只挑常用的场景讲.

首先上示例

```bash
if [[ $VAR -gt 10 ]]
then
  echo "The variable is greater than 10."
elif [[ $VAR -eq 10 ]]
then
  echo "The variable is equal to 10."
else
  echo "The variable is less than 10."
fi
```

其实也就是

```bash
if [[ condition ]]
then
   # logic
elif [[ condition ]]
then
   # logic
fi
```

只不过 `condition` 需要特别注意的是. 不像我们熟悉的`javascript`, 或者 `php` 一样.  可以写 `>`, `=` 对应支持的 `condition` 我上一个列表

- `-n VAR` - Var的长度是否不为0. 也可以理解成变量不为空
- `-z VAR` - VAR变量是否为空.
- `STRING1 = STRING2` - STRING1 是否等于STRING2
- `STRING1 != STRING2` - STRING1 是否不等于STRING2
- `INTEGER1 -eq INTEGER2` - 一样也是用于判断是否相等的. 不同的是这个是用于int类型
- `INTEGER1 -gt INTEGER2` - 用于int类型判断是否不相等
- `INTEGER1 -lt INTEGER2` - 相当于 `INTEGER1 < INTEGER2`
- `INTEGER1 -ge INTEGER2` - 相当于 `INTEGER1 >= INTEGER2`
- `INTEGER1 -le INTEGER2` - 相当于 `INTEGER1 <= INTEGER2`
- `-h FILE` - 文件存在并且文件是符号链接
- `-r FILE` - 文件是否可读
- `-w FILE` - 文件是否可写
- `-x FILE` - 文件是否可执行
- `-d FILE` - 是否是一个目录
- `-e FILE` - 是否是一个文件
- `-f FILE` - 这个也是是否是文件. 跟 `-e` 的区别是. linux 里面很多东西都是用文件来描述的. 比如网卡. socket. 而 `-e` 就是这些文件. 都属于文件. `-f` 则排除了这些特殊文件类型. 大部分情况下用 `-f` 比较多. 那比如你写一个脚本去检查本地mysql服务有没有启动. 因为有时候 mysql 会使用本地 unix 协议来启动服务. 这时候就得用 `-e`

上面列举了这么多。我们来写几个常用的场景

判断是否有参数
```bash
#!/bin/bash

env=${1:-""}

if [[ -z "$env" ]]; then
    echo -n "发布环境(rc|pro):"
    read -r env
fi
```

检测本地是否有安装docker
```bash
# detect docker
if ! [ -x "$(command -v docker)" ]; then
  # auto install docker
  # eq macos
  if [ "$(uname)" == "Darwin" ];then
    wget https://download.docker.com/mac/stable/Docker.dmg -O "$HOME/Downloads/docker.dmg";
    open "$HOME/Downloads/docker.dmg";
    echo "Please install docker and retry";
    exit;
  else
    curl -sSL https://get.docker.com | sh
  fi
fi
```

这里检测是否有安装docker其实就是通过 `command` 获取来docker的路径后. 通过判断对应路径是否可执行来实现的. 对应我们上面condition列表的 `-x FILE`

### 循环控制 loop
shell 我们最常用的就是 `for` 循环

无限循环
```bash
for (( ; ; ))
do
   echo "无限循环 [ CTRL+C 停止]"
done
```

`for in`

```bash
for i in 1 2 3 4 5 6
do
   echo $i
done
```

实际举例 - 获取所有html文件
```bash
#!/bin/bash
for html in $(find "$BUILDOUT_DIR" -type f -name "*.html")
do
    # logic
done
```

执行id 100-300的任务
```bash
for id in {100...300}
do
    php cmcli jobs:run $i
done
```

### Reference
- https://zh.wikipedia.org/wiki/Bash
- https://man.linuxde.net/find-2
- https://www.shellcheck.net/
- https://opensource.com/business/16/3/top-linux-shells
- https://linuxize.com/post/bash-if-else-statement/