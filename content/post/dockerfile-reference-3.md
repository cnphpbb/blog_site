+++
title = "Dockerfile 参考文档 (三)"
date = 2018-09-02T12:01:39+08:00
draft = true
menu = "main"
Categories = ["Docker", "Dockerfile", "Guide"]
Description = "Dockerfile 参考文档, 写好Dockerfile。"
Tags = ["docker","guide","reference","dockerfile","开源"]
+++

## Dockerfile

`Dockerfile` 是由一系列命令和参数构成的脚本，一个 `Dockerfile` 里面包含了构建整个 `image` 的完整命令。Docker通过 `docker build` 执行`Dockerfile` 中的一系列命令自动构建 `image`。

`Dockerfile` 其语法非常简单，此页面描述了您可以在 `Dockerfile` 中使用的命令。 阅读此页面后，你可以参阅[Dockerfile最佳实践][7]。

## Base Guide

### Usage

`docker build` 命令从 `Dockerfile` 和 `context` 构建image。`context` 是 `PATH` 或`URL` 处的文件。`PATH` 本地文件目录。 `URL` 是Git repository的位置。

`context` 以递归方式处理。因此，`PATH` 包括任何子目录，`URL` 包括repository及submodules。一个使用当前目录作为 `context` 的简单构建命令：

```shell
$ docker build .
Sending build context to Docker daemon  6.51 MB
...
```

构建由Docker守护程序运行，而不是由CLI运行。构建过程所做的第一件事是将整个context（递归地）发送给守护进程。大多数情况下，最好是将 `Dockerfile` 和所需文件复制到一个空的目录，再到这个目录进行构建。

> `警告`：不要使用根目录 `/` 作为PATH，因为它会导致构建将硬盘驱动器的所有内容传输到Docker守护程序。

build时添加文件，通过 `Dockerfile` 引用指令中指定的文件，例如 `COPY` 指令。要增加构建的性能，请通过将`.dockerignore` 文件添加到 `context` 目录中来排除文件和目录。有关如何创建 [`.dockerignore`](#.dockerignore-file) 文件的信息，请参阅此页上的文档。

一般的，`Dockerfile` 位于 `context` 的根中。但使用 `-f` 标志可指定Dockerfile的位置。

```bash
$ docker build -f /path/to/a/Dockerfile .
```

如果build成功，您可以指定要保存新image的repository和tag：

```bash
$ docker build -t cnphpbb/myapp .
```

要在构建后将image标记为多个repositories，请在运行构建命令时添加多个 `-t` 参数：

```bash
$ docker build -t cnphpbb/myapp:1.0.2 -t cnphpbb/myapp:latest .
```

**Docker**守护程序一个接一个地运行 `Dockerfile` 中的指令，如果需要，将每个指令的结果提交到一个新image，最后输出新映像的**ID**。**Docker**守护进程将自动清理您发送的context。

请注意，每个指令独立运行，并导致创建一个新 image  - 因此 `RUN cd / tmp` 对下一个指令不会有任何影响。

只要有可能，**Docker**将重新使用中间**images**（缓存），以显着加速 `docker build` 过程。 这由控制台输出中的使用缓存消息指示。

```bash
$ docker build -t svendowideit/ambassador .
Sending build context to Docker daemon 15.36 kB
Step 1 : FROM alpine:3.2
 ---> 31f630c65071
Step 2 : MAINTAINER SvenDowideit@home.org.au
 ---> Using cache
 ---> 2a1c91448f5f
Step 3 : RUN apk update &&      apk add socat &&        rm -r /var/cache/
 ---> Using cache
 ---> 21ed6e7fbb73
Step 4 : CMD env | grep _TCP= | (sed 's/.*_PORT_\([0-9]*\)_TCP=tcp:\/\/\(.*\):\(.*\)/socat -t 100000000 TCP4-LISTEN:\1,fork,reuseaddr TCP4:\2:\3 \&/' && echo wait) | sh
 ---> Using cache
 ---> 7ea8aef582cc
Successfully built 7ea8aef582cc
```

构建成功后，就可以准备[Pushing a repository to its registry][1]。

### Format

`Dockerfile` 的格式如下：

```dockerfile
# Comment
INSTRUCTION arguments
```

`INSTRUCTION` 是不区分大小写的，不过建议大写。
`Dockerfile `中的指令第一个指令必需是 `FROM` ，指定构建镜像的[Base Image][2]。

`Dockerfile` 中以 `#` 开头的行都将视为注释，除非是 `[Parser directives]()` 解析器指令。不支持连续注释符。

```dockerfile
# Comment
RUN echo 'we are running some # of cool things'
```

### Parser directives

解析器指令是可选的，并且影响处理 `Dockerfile` 中后续行的方式。解析器指令不会向构建中添加图层，并且不会显示在构建步骤。解析器指令是以 `# directive = value` 形式写成一种特殊类型的注释。单个指令只能使用一次。

一旦注释，空行或构建器指令已经被处理，Docker不再寻找解析器指令。相反，它将任何格式化为解析器指令作为注释，并且不尝试验证它是否可能是解析器指令。因此，所有解析器指令必须位于 `Dockerfile` 的最顶端。

解析器指令不区分大小写。然而，约定是他们是小写的。公约还要包括一个空白行，遵循任何解析器指令。解析器指令不支持行连续字符。

由于这些规则，以下示例都无效：

因行延续，无效：

```dockerfile
# direc \
tive=value
```

因出现两次，无效：

```dockerfile
# directive=value1
# directive=value2

FROM ImageName
```

因写在构建指令后，无效：

```dockerfile
FROM ImageName
# directive=value
```

因写在不是解析器指令之后，无效：

```dockerfile
# About my dockerfile
FROM ImageName
# directive=value
```

未知指令视为注释，之后的解析器指令也随之，无效：

```dockerfile
# unknowndirective=value
# knowndirective=value
```

解析器指令中允许使用非换行符空格，下面几行被视为相同：

```dockerfile
#directive=value
# directive =value
#   directive= value
# directive = value
#     dIrEcTiVe=value
```

支持以下解析器指令： * escape

### escape

```dockerfile
# escape=\ (backslash)
或者
# escape=` (backtick)
```

`escape` 指令设置用于在 `Dockerfile` 中转义字符的字符。如果未指定，则缺省转义字符为 `\`。

转义字符既用于转义行中的字符，也用于转义换行符。这允许 `Dockerfile` 指令跨越多行。注意，不管 `escape` 解析器指令是否包括在 `Dockerfile` 中，在 `RUN` 命令中不执行转义，除非在行的末尾。

将转义字符设置为 \` 在 `Windows`上特别有用，其中 `\` 是目录路径分隔符。 \` 与[Windows PowerShell][3]一致。

请考虑以下示例，这将在Windows上以非显而易见的方式失败。第二行末尾的第二个\将被解释为换行符，而不是从第一个\转义的目标。类似地，假设第三行结尾处的\实际上作为一条指令处理，它将被视为行继续。这个dockerfile的结果是第二行和第三行被认为是单个指令：

```dockerfile
FROM windowsservercore
COPY testfile.txt c:\\
RUN dir c:\
```

结构是：

```shell
PS C:\John> docker build -t cmd .
Sending build context to Docker daemon 3.072 kB
Step 1 : FROM windowsservercore
 ---> dbfee88ee9fd
Step 2 : COPY testfile.txt c:RUN dir c:
GetFileAttributesEx c:RUN: The system cannot find the file specified.
PS C:\John>
```

上述的一个解决方案是使用 `/` 作为 `COPY` 指令和 `dir` 的目标。然而，这种语法，最好的情况是混乱，因为它在 `Windows` 上是不平常的路径，最坏的情况下，错误倾向，因为并不是 `Windows` 上的所有命令支持 `/` 作为路径分隔符。

通过添加转义解析器指令，下面的Dockerfile在Windows上使用文件路径的自然平台语义成功：

```dockerfile
# escape=`

FROM windowsservercore
COPY testfile.txt c:\
RUN dir c:\
```

结果是：

```shell
PS C:\John> docker build -t succeeds --no-cache=true .
Sending build context to Docker daemon 3.072 kB
Step 1 : FROM windowsservercore
 ---> dbfee88ee9fd
Step 2 : COPY testfile.txt c:\
 ---> 99ceb62e90df
Removing intermediate container 62afbe726221
Step 3 : RUN dir c:\
 ---> Running in a5ff53ad6323
 Volume in drive C has no label.
 Volume Serial Number is 1440-27FA

 Directory of c:\

03/25/2016  05:28 AM    <DIR>          inetpub
03/25/2016  04:22 AM    <DIR>          PerfLogs
04/22/2016  10:59 PM    <DIR>          Program Files
03/25/2016  04:22 AM    <DIR>          Program Files (x86)
04/18/2016  09:26 AM                 4 testfile.txt
04/22/2016  10:59 PM    <DIR>          Users
04/22/2016  10:59 PM    <DIR>          Windows
               1 File(s)              4 bytes
               6 Dir(s)  21,252,689,920 bytes free
 ---> 2569aa19abef
Removing intermediate container a5ff53ad6323
Successfully built 2569aa19abef
PS C:\John>
```

### Environment replacement

环境变量（使用[ENV](#env)语句声明）也可以在某些指令中用作要由 `Dockerfile` 解释的变量。还可以处理转义，以将类似变量的语法包含在语句中。

环境变量在 `Dockerfile` 中用 `$variable_name` 或 `${variable_name}` 表示。它们被等同对待，并且括号语法通常用于解决不带空格的变量名的问题，例如 `${foo}_bar` 。

`${variable_name}` 语法还支持以下指定的一些标准 `bash` 修饰符：

* `${variable:-word}` 表示如果设置了 `variable` ，则结果将是该值。如果 `variable` 未设置，那么word将是结果。
* `${variable:+word}` 表示如果设置了 `variable` ，那么 `word` 将是结果，否则结果是空字符串。

在所有情况下，`word` 可以是任何字符串，包括额外的环境变量。

可以通过在变量之前添加 `\` 来转义：`\$foo` 或 `\${foo}` ，分别转换为 `$foo` 和 `${foo}` 。

示例（解析的表示显示在 `#` 后面）：

```dockerfile
FROM busybox
ENV foo /bar
WORKDIR ${foo}   # WORKDIR /bar
ADD . $foo       # ADD . /bar
COPY \$foo /quux # COPY $foo /quux
```

`Dockerfile` 中的以下指令列表支持环境变量：

* ADD
* COPY
* ENV
* EXPOSE
* LABEL
* USER
* WORKDIR
* VOLUME
* STOPSIGNAL

以及：

* ONBUILD（当与上面支持的指令之一组合时）

> 注意：在1.4之前，ONBUILD指令不支持环境变量，即使与上面列出的任何指令相结合。

环境变量替换将在整个命令中对每个变量使用相同的值。换句话说，在这个例子中：

```dockerfile
ENV abc=hello
ENV abc=bye def=$abc
ENV ghi=$abc
```

将导致 `def` 值为 `hello`，不再 `bye`。然而， `ghi`的值为 `bye`，因为它不是设置 `abc` 为 `bye` 的相同命令的一部分。

### .dockerignore file

在docker CLI将上下文发送到docker守护程序之前，它会在上下文的根目录中查找名为 `.dockerignore` 的文件。如果此文件存在，CLI将修改上下文以排除匹配其中模式的文件和目录。这有助于避免不必要地向守护程序发送大型或敏感文件和目录，并可能使用 `ADD` 或 `COPY` 将其添加到映像。

CLI将 `.dockerignore` 文件解释为换行符分隔的模式列表，类似于Unix `shell` 的file globs。为了匹配的目的，上下文的根被认为是工作目录和根目录。例如，模式 `/foo/bar` 和 `foo/bar` 都排除了 `PATH` 的 `foo` 子目录中的名为 `bar` 的文件或目录，或者位于 `URL` 处的git repository的根目录中。也不排除任何其他。

如果 `.dockerignore` 文件中的一行以第1列中以 `#` 开头，则此行被视为注释，并在CLI解释之前被忽略。

这里是一个例子 `.dockerignore` 文件：

```dockerignore
# comment
*/temp*
*/*/temp*
temp?
```

此文件导致以下构建行为：

|规则|行为|
|:---|:---|
| `# comment` | 忽略 |
| `*/temp*` | 在根的任何直接子目录中排除其名称以temp开头的文件和目录。 例如，普通文件/somedir/temporary.txt被排除，目录/somedir/temp也被排除。|
| `*/*/temp*` | 从根目录下两级的任何子目录中排除以temp开头的文件和目录。 例如，排除了/somedir/subdir/temporary.txt。|
| `temp?` | 排除根目录中名称为temp的单字符扩展名的文件和目录。 例如，/tempa和/tempb被排除。|

匹配是使用Go的[filepath.Match][4]规则完成的。 预处理步骤删除前导和尾随空格并消除 `.` 和 `..` 元素使用Go的[filepath.Clean][5]。预处理后为空的行将被忽略。

除了Go的filepath.Match规则，Docker还支持一个特殊的通配符字符串 `**` ，它匹配任何数量的目录（包括零）。 例如，`**/*.go` 将排除所有目录中找到的以 `.go` 结尾的所有文件，包括构建上下文的根。

行开头 `!` （感叹号）可用于排除例外。 以下是使用此机制的 `.dockerignore` 文件示例：

```dockerignore
*.md
!README.md
```

除了 `README.md` 之外的所有markdown文件都从上下文中排除。

放置 `!` 异常规则影响行为：匹配特定文件的 `.dockerignore` 的最后一行确定它是包括还是排除。思考下面的例子：

```dockerignore
*.md
!README*.md
README-secret.md
```

除了 `README-secret.md` 之外的 `README` 文件，上下文中排除所有markdown文件。

现在思考这个例子：

```dockerignore
*.md
README-secret.md
!README*.md
```

包括所有**README**文件。 中间行没有效果，因为最后的 `!README*.md` 与 `README-secret.md`匹配。

您甚至可以使用 `.dockerignore` 文件来排除 `Dockerfile` 和 `.dockerignore` 文件。这些文件仍然发送到守护程序，因为它需要它们来完成它的工作。但是 `ADD` 和 `COPY` 命令不会将它们复制到映像。

最后，您可能需要指定要包括在上下文中的文件，而不是要排除的文件。 要实现这一点，指定 `*` 作为第一个模式，后面跟一个或多个 `!` 异常模式。

> `注意`：由于历史原因 `.` 模式。被忽略。


## CLI Guide

### FROM

```dockerfile
FROM <image>
# 或则
FROM <image>:<tag>
# 或则
FROM <image>@<digest>
```

 `FROM` 指令为后续指令设置[Base Image][2]。因此，有效的 `Dockerfile` 必须具有 `FROM` 作为其第一条指令。image可以是任何有效的image - 可以从[Public Repositories][6] **pulling an image**。

* `FROM` 必须是 `Dockerfile` 中的第一个非注释指令。
* `FROM` 可以在单个 `Dockerfile` 中多次出现，以创建多个图像。只需记下在每个新的 `FROM` 命令之前由提交输出的最后一个image ID。
* `tag` 或 `digest` 是可选的。如果省略其中任何一个，构建器将默认使用 `latest` 。如果构建器与 `tag` 值不匹配，则构建器将返回错误。

### MAINTAINER

```dockerfile
MAINTAINER <name>
```

`MAINTAINER` 指令允许您设置生成的images的作者字段。

### RUN

RUN有2种形式：

* `RUN <command>`（*shell*形式，命令在shell中运行，Linux上为 `/bin/sh -c`，Windows上为cmd /S/C）
* `RUN ["executable"，"param1"，"param2"]`（exec 形式）

`RUN` 指令将在当前image之上的新层中执行任何命令，并提交结果。生成的已提交image将用于 `Dockerfile` 中的下一步。

分层 `RUN` 指令和生成提交符合Docker的核心概念，其中提交很轻量，可以从image历史中的任何点创建容器，就像源代码控制一样。

`exec` 形式使得可以避免shell字符串变化，以及使用不包含指定的shell可执行文件的基本image来运行RUN命令。

可以使用 `shell` 命令更改 shell 表单的默认 shell。

在shell形式中，可以使用 `\` （反斜杠）将单个 `RUN` 指令继续到下一行。
例如，考虑这两行：`RUN /bin/bash -c 'source $HOME/.bashrc ; \ echo $HOME'` 它们等同于这一行： `RUN /bin/bash -c 'source $HOME/.bashrc ; echo $HOME'`

> `注意` ：要使用不同的 shell ，而不是 `/bin/sh`，请使用在所需 shell 中传递的 exec 形式。例如，`RUN [“/bin/bash”,“-c”,“echo hello”]` 
>
> `注意` ：exec形式作为JSON数组解析，这意味着您必须在单词之外使用双引号（”）而不是单引号（’）。
>
> `注意` ：与shell表单不同，exec表单不调用命令shell。这意味着正常的shell处理不会发生。例如， `RUN ["echo","$HOME"]` 不会在 `$HOME` 上进行可变替换。如果你想要shell处理，那么使用shell形式或直接执行一个shell，例如： `RUN ["sh","-c","echo $HOME"]` 。当使用exec形式并直接执行shell时，正如shell形式的情况，它是做环境变量扩展的shell，而不是docker。
>
> `注意` ：在JSON形式中，有必要转义反斜杠。这在Windows上特别相关，其中反斜杠是路径分隔符。因为不是有效的JSON，并且以意外的方式失败，以下行将被视为shell形式： `RUN ["c:\windows\system32\tasklist.exe"]` 此示例的正确语法为： `RUN ["c:\\windows\\system32\\tasklist.exe"]` 

用于RUN指令的高速缓存在下一次构建期间不会自动失效。用于诸如 `RUN apt-get dist-upgrade` 之类的指令的高速缓存将在下一次构建期间被重用。可以通过使用 `--no-cache` 标志来使用于RUN指令的高速缓存无效，例如 `docker build --no-cache`。

有关详细信息，请参阅[Dockerfile最佳实践指南][7]。

用于 `RUN` 指令的高速缓存可以通过 `ADD` 指令无效。有关详细信息，请参见[下文][#copy]。

### Known issues（RUN）

* [Issue 783][8]是关于在使用AUFS文件系统时可能发生的文件权限问题。例如，您可能会在尝试 `rm` 文件时注意到它。对于具有最近aufs版本的系统（即，可以设置 `dirperm1` 安装选项），docker将尝试通过使用 `dirperm1` 选项安装image来自动解决问题。有关 `dirperm1` 选项的更多详细信息，请参见 [aufs手册页][9] 如果您的系统不支持 `dirperm1`，则该问题描述了一种解决方法。

### CMD

CMD指令三种形式：

* CMD ["executable","param1","param2"] (exec form, 首选形式)
* CMD ["param1","param2"] (as default parameters to ENTRYPOINT)
* CMD command param1 param2 (shell form)

在 `Dockerfile` 中只能有一个 `CMD` 指令。如果您列出多个 `CMD`，则只有最后一个 `CMD` 将生效。

`CMD` 的主要目的是为执行容器提供默认值。这些默认值可以包括可执行文件，或者它们可以省略可执行文件，在这种情况下，您还必须指定 `ENTRYPOINT` 指令。

> `注意` ：如果使用 `CMD` 为 `ENTRYPOINT` 指令提供默认参数，`CMD` 和 `ENTRYPOINT` 指令都应以JSON数组格式指定。
>
> `注意` ：exec形式作为JSON数组解析，这意味着您必须在单词之外使用双引号（”）而不是单引号（’）。
>
> `注意` ：与shell表单不同，exec表单不调用命令shell。这意味着正常的shell处理不会发生。例如， `CMD ["echo"，"$HOME"]` 不会在 `$HOME` 上进行可变替换。如果你想要shell处理，那么使用shell形式或直接执行一个shell，例如： `CMD ["sh","-c","echo $HOME"]` 。当使用exec形式并直接执行shell时，正如shell形式的情况，它是做环境变量扩展的shell，而不是docker。

当以shell或exec格式使用时，`CMD` 指令设置运行image时要执行的命令。

如果使用 `CMD` 的shell形式，那么 `<command>` 将在 `/bin/sh -c` 中执行：

```dockerdile
FROM ubuntu
CMD echo "This is a test." | wc -
```

如果你想运行你的 `<command>` 没有shell，那么你必须将该命令表示为一个JSON数组，并给出可执行文件的完整路径。此数组形式是 `CMD` 的首选格式。任何其他参数必须单独表示为数组中的字符串：

```dockerfile
FROM ubuntu
CMD ["/usr/bin/wc","--help"]
```

如果你希望你的容器每次运行相同的可执行文件，那么你应该考虑使用ENTRYPOINT结合CMD。 请参阅[ENTRYPOINT][#entrypoint]。

如果用户指定 `docker run` 参数，那么它们将覆盖 `CMD` 中指定的默认值。

> `注意` ：不要将 `RUN` 和 `CMD` 混淆。`RUN` 实际上运行一个命令并提交结果;`CMD` 在构建时不执行任何操作，但指定了image的预期命令。

### LABEL

```dockerfile
LABEL <key>=<value> <key>=<value> <key>=<value> ...
```

`LABEL` 指令向image添加元数据。`LABEL` 是键值对。要在 `LABEL` 值中包含空格，请使用引号和反斜杠，就像在命令行解析中一样。几个使用示例：

```dockerfile
LABEL "com.example.vendor"="ACME Incorporated"
LABEL com.example.label-with-value="foo"
LABEL version="1.0"
LABEL description="This text illustrates \
that label-values can span multiple lines."
```

image可以有多个label。要指定多个label，Docker建议在可能的情况下将标签合并到单个 `LABEL` 指令中。每个 `LABEL` 指令产生一个新层，如果使用许多标签，可能会导致效率低下的图像。该示例产生单个图像层。

```dockerfile
LABEL multi.label1="value1" multi.label2="value2" other="value3"
```

上面的也可写为：

```dockerfile
LABEL multi.label1="value1" \
      multi.label2="value2" \
      other="value3"
```

标签是添加的，包括 `LABEL` 在 `FROM` images中。如果Docker遇到已经存在的label/key，则新值将覆盖具有相同键的任何先前标签。

要查看image的labels，请使用 `docker inspect` 命令。

```dockerfile
"Labels": {
    "com.example.vendor": "ACME Incorporated"
    "com.example.label-with-value": "foo",
    "version": "1.0",
    "description": "This text illustrates that label-values can span multiple lines.",
    "multi.label1": "value1",
    "multi.label2": "value2",
    "other": "value3"
},
```

### EXPOSE

```dockerfile
EXPOSE <port> [<port>...]
```

`EXPOSE` 指令通知Docker容器在运行时侦听指定的网络端口。 `EXPOSE` 不使主机的容器的端口可访问。为此，必须使用 `-p` 标志发布一系列端口，或者使用 `-P` 标志发布所有暴露的端口。您可以公开一个端口号，并用另一个端口号在外部发布。

要在主机系统上设置端口重定向，请参阅[使用-P标志][10]。Docker网络功能支持创建网络，无需在网络中公开端口，有关详细信息，请参阅[此功能的概述][11]）。

### ENV

```dockerfile
ENV <key> <value>
ENV <key>=<value> ...
```

`ENV` 指令将环境变量 `<key>` 设置为值 `<value>`。该值将在所有”descendant” `Dockerfile` 命令的环境中，并且可以在许多中被[替换为inline][12]。

`ENV` 指令有两种形式。第一种形式， `ENV <key> <value>` ，将单个变量设置为一个值。第一个空格后面的整个字符串将被视为 `<value>`  - 包括空格和引号等字符。

第二种形式， `ENV <key> = <value> ...` ，允许一次设置多个变量。注意，第二种形式在语法中使用等号（=），而第一种形式不使用。与命令行解析类似，引号和反斜杠可用于在值内包含空格。

例如：

```dockerfile
ENV myName="John Doe" myDog=Rex\ The\ Dog \
    myCat=fluffy
# 和
ENV myName John Doe
ENV myDog Rex The Dog
ENV myCat fluffy
```

将在最终容器中产生相同的净结果，但第一种形式是优选的，因为它产生单个高速缓存层。

使用 `ENV` 设置的环境变量将在从生成的image运行容器时保留。您可以使用 `docker inspect` 查看值，并使用 `docker run --env <key> = <value>` 更改它们。

> `注意` ：环境持久性可能会导致意外的副作用。例如，将 `ENV DEBIAN_FRONTEND` 设置为非交互式可能会使apt-get用户混淆基于Debian的映像。要为单个命令设置值，请使用 `RUN <key> = <value> <command>` 。

### ADD

两种形式：

* `ADD <src>... <dest>`
* `ADD ["<src>",... "<dest>"]` (对于包含空格的路径，此形式是必需的)

`ADD` 指令从 `<src>` 复制新文件，目录或远程文件 `URL` ，并将它们添加到容器的文件系统，路径 `<dest>` 。

可以指定多个 `<src>` 资源，但如果它们是文件或目录，那么它们必须是相对于正在构建的源目录（构建的context）。

每个 `<src>` 可能包含通配符，匹配将使用Go的[filepath.Match][4]规则完成。 例如：

```dockerfile
ADD hom* /mydir/        # adds all files starting with "hom"
ADD hom?.txt /mydir/    # ? is replaced with any single character, e.g., "home.txt"
```

`<dest>` 是绝对路径或相对于 `WORKDIR` 的路径，源将在目标容器中复制到其中。

```dockerfile
ADD test relativeDir/          # adds "test" to `WORKDIR`/relativeDir/
ADD test /absoluteDir/         # adds "test" to /absoluteDir/
```

所有新文件和目录都使用UID和GID为0创建。

在 `<src>` 是远程文件URL的情况下，目标将具有600的权限。如果正在检索的远程文件具有HTTP  `Last-Modified` 标头，则来自该标头的时间戳将用于设置目的地上的 `mtime` 文件。然而，像在 `ADD` 期间处理的任何其它文件一样， `mtime` 将不包括在确定文件是否已经改变并且高速缓存应该被更新。

> `注意` ：如果通过传递一个 `Dockerfile` 通过STDIN（ `docker build - <somefile` ）构建，没有构建上下文，所以 `Dockerfile` 只能包含一个基于URL的 `ADD` 指令。您还可以通过STDIN传递压缩归档文件：（ `docker build - <archive.tar.gz` ），归档根目录下的 `Dockerfile` 和归档的其余部分将在构建的上下文中使用。
>
> `注意` ：如果您的URL文件使用身份验证保护，则您需要使用 `RUN wget` ， `RUN curl` 或从容器内使用其他工具，因为ADD指令不支持身份验证。
>
> `注意` ：如果 `<src>` 的内容已更改，第一个遇到的 `ADD` 指令将使来自Dockerfile的所有后续指令的高速缓存无效。这包括使用于RUN指令的高速缓存无效。有关详细信息，请参阅[Dockerfile最佳实践指南][7]。

`ADD` 遵守以下规则：

* `<src>` 路径必须在构建的上下文中;你不能 `ADD ../something /something` ，因为docker构建的第一步是发送上下文目录（和子目录）到docker守护进程。如果 `<src>` 是URL并且 `<dest>` 不以尾部斜杠结尾，则从URL下载文件并将其复制到 `<dest>` 。
* 如果 `<src>` 是URL并且 `<dest>` 以尾部斜杠结尾，则从URL中推断文件名，并将文件下载到 `<dest>/<filename>` 。例如， `ADD http://example.com/foobar /` 会创建文件 `/ foobar` 。网址必须有一个非平凡的路径，以便在这种情况下可以发现一个适当的文件名（`http://example.com` 不会工作）。
* 如果 `<src>` 是目录，则复制目录的整个内容，包括文件系统元数据。

  > `注意` ：目录本身不被复制，只是其内容。

* 如果 `<src>` 是识别的压缩格式（identity，gzip，bzip2或xz）的本地tar存档，则将其解包为目录。来自远程URL的资源不会解压缩。当目录被复制或解压缩时，它具有与 `tar -x` 相同的行为：结果是以下的联合：

  1. 无论在目的地路径和
  2. 源树的内容，冲突以逐个文件为基础解析为“2.”。

> `注意` ：文件是否被识别为识别的压缩格式，仅基于文件的内容，而不是文件的名称。例如，如果一个空文件以.tar.gz结尾，则不会被识别为压缩文件，并且不会生成任何解压缩错误消息，而是将该文件简单地复制到目的地。

* 如果 `<src>` 是任何其他类型的文件，它会与其元数据一起单独复制。在这种情况下，如果 `<dest>` 以尾部斜杠 `/` 结尾，它将被认为是一个目录，并且 `<src>` 的内容将被写在 `<dest>/base(<src>)` 。
* 如果直接或由于使用通配符指定了多个 `<src>` 资源，则 `<dest>` 必须是目录，并且必须以斜杠 `/` 结尾。
* 如果 `<dest>` 不以尾部斜杠结尾，它将被视为常规文件，`<src>` 的内容将写在 `<dest>`。
* 如果 `<dest>` 不存在，则会与其路径中的所有缺少的目录一起创建。

### COPY

两种形式：

* `COPY <src>... <dest>`
* `COPY ["<src>",... "<dest>"]` (this form is required for paths containing whitespace)

基本和 `ADD` 类似，不过 `COPY` 的 `<src>` 不能为URL。

### ENTRYPOINT

两种形式：

* ENTRYPOINT [“executable”, “param1”, “param2”] （exec 形式, 首选）
* ENTRYPOINT command param1 param2 (shell 形式)

`ENTRYPOINT` 允许您配置容器，运行执行的可执行文件。

例如，以下将使用其默认内容启动nginx，侦听端口80：

```shell
docker run -i -t --rm -p 80:80 nginx
```

`docker run <image>` 的命令行参数将附跟在 exec 形式的 `ENTRYPOINT` 中的所有元素之后，并将覆盖使用 `CMD` 指定的所有元素。这允许将参数传递到入口点，即 `docker run <image> -d` 将把 `-d` 参数传递给入口点。您可以使用 `docker run --entrypoint `标志覆盖 `ENTRYPOINT` 指令。

*shell* 形式防止使用任何 `CMD` 或运行命令行参数，但是缺点是您的 `ENTRYPOINT` 将作 `/bin/sh -c` 的子命令启动，它不传递信号。这意味着可执行文件将不是容器的 `PID 1` ，并且不会接收Unix信号，因此您的可执行文件将不会从 `docker stop <container>` 接收到 `SIGTERM` 。

只有 `Dockerfile` 中最后一个 `ENTRYPOINT` 指令会有效果。

#### Exec form ENTRYPOINT example

您可以使用 `ENTRYPOINT` 的*exec*形式设置相当稳定的默认命令和参数，然后使用任一形式的 `CMD` 设置更可能更改的其他默认值。

```dockerfile
FROM ubuntu
ENTRYPOINT ["top", "-b"]
CMD ["-c"]
```

运行容器时，您可以看到top是唯一的进程：

```shell
$ docker run -it --rm --name test  top -H
top - 08:25:00 up  7:27,  0 users,  load average: 0.00, 0.01, 0.05
Threads:   1 total,   1 running,   0 sleeping,   0 stopped,   0 zombie
%Cpu(s):  0.1 us,  0.1 sy,  0.0 ni, 99.7 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
KiB Mem:   2056668 total,  1616832 used,   439836 free,    99352 buffers
KiB Swap:  1441840 total,        0 used,  1441840 free.  1324440 cached Mem

  PID USER      PR  NI    VIRT    RES    SHR S %CPU %MEM     TIME+ COMMAND
    1 root      20   0   19744   2336   2080 R  0.0  0.1   0:00.04 top
```

要进一步检查结果，可以使用 `docker exec` ：

```shell
$ docker exec -it test ps aux
USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root         1  2.6  0.1  19752  2352 ?        Ss+  08:24   0:00 top -b -H
root         7  0.0  0.1  15572  2164 ?        R+   08:25   0:00 ps aux
```

并且你可以优雅地请求 `top` 使用 `docker stop test` 关闭。

以下 `Dockerfile` 显示使用 `ENTRYPOINT` 在前台运行 `Apache`（即，作为 `PID 1` ）：

```dockerfile
FROM debian:stable
RUN apt-get update && apt-get install -y --force-yes apache2
EXPOSE 80 443
VOLUME ["/var/www", "/var/log/apache2", "/etc/apache2"]
ENTRYPOINT ["/usr/sbin/apache2ctl", "-D", "FOREGROUND"]
```

如果需要为单个可执行文件编写启动脚本，可以使用 `exec` 和 `gosu` 命令确保最终可执行文件接收到Unix信号：

```shell
#!/bin/bash
set -e

if [ "$1" = 'postgres' ]; then
    chown -R postgres "$PGDATA"

    if [ -z "$(ls -A "$PGDATA")" ]; then
        gosu postgres initdb
    fi

    exec gosu postgres "$@"
fi

exec "$@"
```

最后，如果需要在关闭时进行一些额外的清理（或与其他容器通信），或者协调多个可执行文件，您可能需要确保 `ENTRYPOINT` 脚本接收到Unix信号，传递它们，然后做一些更多的工作：

```shell
#!/bin/sh
# Note: I've written this using sh so it works in the busybox container too

# USE the trap if you need to also do manual cleanup after the service is stopped,
#     or need to start multiple services in the one container
trap "echo TRAPed signal" HUP INT QUIT TERM

# start service in background here
/usr/sbin/apachectl start

echo "[hit enter key to exit] or run 'docker stop <container>'"
read

# stop service and clean up here
echo "stopping apache"
/usr/sbin/apachectl stop

echo "exited $0"
```

如果你用 `docker run -it --rm -p 80:80 --name test apache` 运行image，则可以使用 `docker exec` 或 `docker top` 检查容器的进程，然后请求脚本停止Apache：

```shell
$ docker exec -it test ps aux
USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root         1  0.1  0.0   4448   692 ?        Ss+  00:42   0:00 /bin/sh /run.sh 123 cmd cmd2
root        19  0.0  0.2  71304  4440 ?        Ss   00:42   0:00 /usr/sbin/apache2 -k start
www-data    20  0.2  0.2 360468  6004 ?        Sl   00:42   0:00 /usr/sbin/apache2 -k start
www-data    21  0.2  0.2 360468  6000 ?        Sl   00:42   0:00 /usr/sbin/apache2 -k start
root        81  0.0  0.1  15572  2140 ?        R+   00:44   0:00 ps aux
$ docker top test
PID                 USER                COMMAND
10035               root                {run.sh} /bin/sh /run.sh 123 cmd cmd2
10054               root                /usr/sbin/apache2 -k start
10055               33                  /usr/sbin/apache2 -k start
10056               33                  /usr/sbin/apache2 -k start
$ /usr/bin/time docker stop test
test
real    0m 0.27s
user    0m 0.03s
sys 0m 0.03s
```

> `注意` ：您可以使用 `--entrypoint` 覆盖 `ENTRYPOINT` 设置，但这只能将二进制设置为*exec*（不使用 `sh -c` ）。
>
> `注意` ：*exec*形式作为JSON数组解析，这意味着您必须在单词之外使用双引号（”）而不是单引号（’）。
>
> `注意` ：与 `shell` 形式不同，*exec*形式不调用命令shell。这意味着正常的shell处理不会发生。例如， `ENTRYPOINT ["echo","$ HOME"]` 不会在 `$HOME` 上进行可变替换。如果你想要shell处理，那么使用shell形式或直接执行一个shell，例如： `ENTRYPOINT ["sh","-c","echo $HOME"]` 。当使用exec形式并直接执行shell时，正如shell形式的情况，它是做环境变量扩展的shell，而不是docker。

#### Shell form ENTRYPOINT example

您可以为 `ENTRYPOINT` 指定一个纯字符串，它将在 `/bin/sh -c` 中执行。这中形式将使用shell处理来替换shell环境变量，并且将忽略任何 `CMD` 或 `docker run` 命令行参数。要确保 `docker stop` 将正确地发出任何长时间运行的 `ENTRYPOINT` 可执行文件，您需要记住用 `exec` 启动它：

```dockerfile
FROM ubuntu
ENTRYPOINT exec top -b
```

运行此image时，您将看到单个 `PID 1` 进程：

```shell
$ docker run -it --rm --name test top
Mem: 1704520K used, 352148K free, 0K shrd, 0K buff, 140368121167873K cached
CPU:   5% usr   0% sys   0% nic  94% idle   0% io   0% irq   0% sirq
Load average: 0.08 0.03 0.05 2/98 6
  PID  PPID USER     STAT   VSZ %VSZ %CPU COMMAND
    1     0 root     R     3164   0%   0% top -b
```

它将在 `docker stop` 干净的退出：

```shell
$ /usr/bin/time docker stop test
test
real    0m 0.20s
user    0m 0.02s
sys 0m 0.04s
```

如果忘记将 `exec` 添加到您的 `ENTRYPOINT` 的开头：

```dockerfile
FROM ubuntu
ENTRYPOINT top -b
CMD --ignored-param1
```

然后，您可以运行它（给它一个名称为下一步）：

```shell
$ docker run -it --name test top --ignored-param2
Mem: 1704184K used, 352484K free, 0K shrd, 0K buff, 140621524238337K cached
CPU:   9% usr   2% sys   0% nic  88% idle   0% io   0% irq   0% sirq
Load average: 0.01 0.02 0.05 2/101 7
  PID  PPID USER     STAT   VSZ %VSZ %CPU COMMAND
    1     0 root     S     3168   0%   0% /bin/sh -c top -b cmd cmd2
    7     1 root     R     3164   0%   0% top -b
```

您可以从`top`的输出中看到指定的 `ENTRYPOINT` 不是 `PID 1`。

如果你然后运行docker停止测试，容器将不会完全退出 - 停止命令将强制发送SIGKILL超时后：

```shell
$ docker exec -it test ps aux
PID   USER     COMMAND
    1 root     /bin/sh -c top -b cmd cmd2
    7 root     top -b
    8 root     ps aux
$ /usr/bin/time docker stop test
test
real    0m 10.19s
user    0m 0.04s
sys 0m 0.03s
```

[1]: https://docs.docker.com/engine/tutorials/dockerrepos/#contributing-to-docker-hub
[2]: https://docs.docker.com/engine/reference/glossary/#base-image
[3]: https://technet.microsoft.com/en-us/library/hh847755.aspx
[4]: http://golang.org/pkg/path/filepath#Match
[5]: http://golang.org/pkg/path/filepath/#Clean
[6]: https://docs.docker.com/engine/tutorials/dockerrepos/
[7]: https://docs.docker.com/engine/userguide/eng-image/dockerfile_best-practices/#build-cache
[8]: https://github.com/docker/docker/issues/783
[9]: https://github.com/sfjro/aufs3-linux/tree/aufs3.18/Documentation/filesystems/aufs
[10]: https://docs.docker.com/engine/reference/run/#expose-incoming-ports
[11]: https://docs.docker.com/engine/userguide/networking/
[12]: https://docs.docker.com/engine/reference/builder/#environment-replacement