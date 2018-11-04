+++
title = "Dockerfile 参考文档 （一）"
date = "2018-09-02T12:01:39+08:00"
draft = false
menu = "main"
Categories = ["Docker", "Dockerfile", "Guide"]
Description = "Dockerfile参考文档, 写好Dockerfile的一个非常重要的文档。"
Tags = ["docker","guide","reference","dockerfile","开源"]
+++

## Dockerfile

`Dockerfile` 是由一系列命令和参数构成的脚本，一个 `Dockerfile` 里面包含了构建整个 `image` 的完整命令。Docker通过 `docker build` 执行`Dockerfile` 中的一系列命令自动构建 `image`。

`Dockerfile` 其语法非常简单，此页面描述了您可以在 `Dockerfile` 中使用的命令。 阅读此页面后，你可以参阅[Dockerfile最佳实践][7]。

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