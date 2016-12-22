+++
date = "2015-10-10T13:44:31+08:00"
title = "Golang程序配置方案小结"
draft = false
Categories = ["Golang", "Config"]
Description = "Golang程序配置方案小结"
menu = "main"
Tags = ["golang", "config","solutions"]
+++

在Twitter上看到一篇关于Golang程序配置方案总结的系列文章（一个mini series，共6篇），原文链接：[在这里](https://sfxpt.wordpress.com/2015/06/16/providing-%20options-for-go-applications/)。我觉得不错，这里粗略整理（非全文翻译）一下，供大家参考。

## 一、背景

无论使用任何编程语言开发应用，都离不开配置数据。配置数据提供的形式有多样，不外乎命令行选项(options)、参数（parameters)，环境变量（envvars)以及配置文件等。Golang也不例外。Golang内置flag标准库，可以用来支持部分命令行选项和参数的解析；Golang通过os包提 供的方法可以获取当前环境变量；但Golang没有规定标准配置文件格式(虽说内置支持xml、json)，多通过第三方 包来解决配置文件读取的问题。Golang配置相关的第三方包很多，作者在本文中给出的配置方案中就包含了主流的第三方配置数据操作包。

文章作者认为一个良好的应用配置层次应该是这样的：

1. 程序内内置配置项的初始默认值
1. 配置文件中的配置项值可以覆盖(override)程序内配置项的默认值。
1. 命令行选项和参数值具有最高优先级，可以override前两层的配置项值。

下面就按作者的思路循序渐进探讨golang程序配置方案。

## 二、解析命令行选项和参数

这一节关注golang程序如何访问命令行选项和参数。

golang对访问到命令行参数提供了内建的支持：

```
//cmdlineargs.go
package main

import (
    //      "fmt"
    "os"
    "path/filepath"
)

func main() {
    println("I am ", os.Args[0])

    baseName := filepath.Base(os.Args[0])
    println("The base name is ", baseName)

    // The length of array a can be discovered using the built-in function len
    println("Argument # is ", len(os.Args))

    // the first command line arguments
    if len(os.Args) > 1 {
        println("The first command line argument: ", os.Args[1])
    }
}

```
执行结果如下：

```
  $go build cmdlineargs.go
  $cmdlineargs test one
  I am  cmdlineargs
  The base name is  cmdlineargs
  Argument # is  3
  The first command line argument:  test
```

对于命令行结构复杂一些的程序，我们最起码要用到golang标准库内置的flag包：

```
//cmdlineflag.go
package main

import (
    "flag"
    "fmt"
    "os"
    "strconv"
)

var (
    // main operation modes
    write = flag.Bool("w", false, "write result back instead of stdout\n\t\tDefault: No write back")

    // layout control
    tabWidth = flag.Int("tabwidth", 8, "tab width\n\t\tDefault: Standard")

    // debugging
    cpuprofile = flag.String("cpuprofile", "", "write cpu profile to this file\n\t\tDefault: no default")
)

func usage() {
    // Fprintf allows us to print to a specifed file handle or stream
    fmt.Fprintf(os.Stderr, "\nUsage: %s [flags] file [path ...]\n\n",
        "CommandLineFlag") // os.Args[0]
    flag.PrintDefaults()
    os.Exit(0)
}

func main() {
    fmt.Printf("Before parsing the flags\n")
    fmt.Printf("T: %d\nW: %s\nC: '%s'\n",
        *tabWidth, strconv.FormatBool(*write), *cpuprofile)

    flag.Usage = usage
    flag.Parse()

    // There is also a mandatory non-flag arguments
    if len(flag.Args()) < 1 {
        usage()
    }

    fmt.Printf("Testing the flag package\n")
    fmt.Printf("T: %d\nW: %s\nC: '%s'\n",
        *tabWidth, strconv.FormatBool(*write), *cpuprofile)

    for index, element := range flag.Args() {
        fmt.Printf("I: %d C: '%s'\n", index, element)
    }
}
```
这个例子中：
- 说明了三种类型标志的用法：Int、String和Bool。
- 说明了每个标志的定义都由类型、命令行选项文本、默认值以及含义解释组成。
- 最后说明了如何处理标志选项(flag option)以及非option参数。

不带参数运行：

```
$cmdlineflag
Before parsing the flags
T: 8
W: false
C: ''

Usage: CommandLineFlag [flags] file [path ...]

  -cpuprofile="": write cpu profile to this file
        Default: no default
  -tabwidth=8: tab width
        Default: Standard
  -w=false: write result back instead of stdout
        Default: No write back
```
带命令行标志以及参数运行(一个没有flag，一个有两个flag)：
```
$cmdlineflag aa bb
Before parsing the flags
T: 8
W: false
C: ''
Testing the flag package
T: 8
W: false
C: ''
I: 0 C: 'aa'
I: 1 C: 'bb'
```
```
$cmdlineflag -tabwidth=2 -w aa
Before parsing the flags
T: 8
W: false
C: ''
Testing the flag package
T: 2
W: true
C: ''
I: 0 C: 'aa'
```
从例子可以看出，简单情形下，你无需编写自己的命令行parser或使用第三方包，使用go内建的flag包即可以很好的完成工作。但是golang的 flag包与命令行Parser的事实标准：Posix getopt（C/C++/Perl/Shell脚本都可用）相比，还有较大差距，主要体现在：

1. 无法支持区分long option和short option，比如：-h和–help。
1. 不支持short options合并，比如：ls -l -h <=> ls -hl
1. 命令行标志的位置不能任意放置，比如无法放在non-flag parameter的后面。

不过毕竟flag是golang内置标准库包，你无须付出任何cost，就能使用它的功能。另外支持bool型的flag也是其一大亮点。

## 三、TOML，Go配置文件的事实标准（这个可能不能得到认同）

命令行虽然是一种可选的配置方案，但更多的时候，我们使用配置文件来存储静态的配置数据。就像Java配xml，ruby配yaml，windows配 ini，Go也有自己的搭配组合，那就是TOML（Tom's Obvious, Minimal Language）。

初看toml语法有些类似windows ini，但细致研究你会发现它远比ini强大的多，下面是一个toml配置文件例子：
```
# This is a TOML document. Boom.

title = "TOML Example"

[owner]
name = "Lance Uppercut"
dob = 1979-05-27T07:32:00-08:00 # First class dates? Why not?

[database]
server = "192.168.1.1"
ports = [ 8001, 8001, 8002 ]
connection_max = 5000
enabled = true

[servers]

  # You can indent as you please. Tabs or spaces. TOML don't care.
  [servers.alpha]
  ip = "10.0.0.1"
  dc = "eqdc10"

  [servers.beta]
  ip = "10.0.0.2"
  dc = "eqdc10"

[clients]
data = [ ["gamma", "delta"], [1, 2] ]

# Line breaks are OK when inside arrays
hosts = [
  "alpha",
  "omega"
]
```
看起来很强大，也很复杂，但解析起来却很简单。以下面这个toml 文件为例：
```
Age = 25
Cats = [ "Cauchy", "Plato" ]
Pi = 3.14
Perfection = [ 6, 28, 496, 8128 ]
DOB = 1987-07-05T05:45:00Z
```
和所有其他配置文件parser类似，这个配置文件中的数据可以被直接解析成一个golang struct：
```
type Config struct {
  Age int
  Cats []string
  Pi float64
  Perfection []int
  DOB time.Time // requires `import time`
}
```
其解析的步骤也很简单：
```
var conf Config
if _, err := toml.Decode(tomlData, &conf); err != nil {
  // handle error
}
```
是不是简单的不能简单了！

不过toml也有其不足之处。想想如果你需要使用命令行选项的参数值来覆盖这些配置文件中的选项，你应该怎么做？事实上，我们常常会碰到类似下面这种三层配置结构的情况：

1. 程序内内置配置项的初始默认值
1. 配置文件中的配置项值可以覆盖(override)程序内配置项的默认值。
1. 命令行选项和参数值具有最高优先级，可以override前两层的配置项值。

在go中，toml映射的结果体字段没有初始值。而且go内建flag包也没有将命令行参数值解析为一个go结构体，而是零散的变量。这些可以通过第三方工具来解决，但如果你不想用第三方工具，你也可以像下面这样自己解决，虽然难看一些。
```
func ConfigGet() *Config {
    var err error
    var cf *Config = NewConfig()

    // set default values defined in the program
    cf.ConfigFromFlag()
    //log.Printf("P: %d, B: '%s', F: '%s'\n", cf.MaxProcs, cf.Webapp.Path)

    // Load config file, from flag or env (if specified)
    _, err = cf.ConfigFromFile(*configFile, os.Getenv("APPCONFIG"))
    if err != nil {
        log.Fatal(err)
    }
    //log.Printf("P: %d, B: '%s', F: '%s'\n", cf.MaxProcs, cf.Webapp.Path)

    // Override values from command line flags
    cf.ConfigToFlag()
    flag.Usage = usage
    flag.Parse()
    cf.ConfigFromFlag()
    //log.Printf("P: %d, B: '%s', F: '%s'\n", cf.MaxProcs, cf.Webapp.Path)

    cf.ConfigApply()

    return cf
}
```
就像上面代码中那样，你需要：

1. 用命令行标志默认值设置配置(cf)默认值。
1. 接下来加载配置文件
1. 用配置值(cf)覆盖命令行标志变量值
1. 解析命令行参数
1. 用命令行标志变量值覆盖配置(cf)值。

少一步你都无法实现三层配置能力。

## 四、超越TOML

本节将关注如何克服TOML的各种局限。

为了达成这个目标，很多人会说：使用[viper](https://github.com/spf13/viper)，不过在介绍viper这一重量级选手 之前，我要为大家介绍另外一位不那么知名的选手：[multiconfig](https://github.com/koding/multiconfig)。

有些人总是认为大的就是好的，但我相信适合的还是更好的。因为：

1、viper太重量级，使用viper时你需要pull另外20个viper依赖的第三方包
2、事实上，viper单独使用还不足以满足需求，要想得到viper全部功能，你还需要另外一个包配合，而后者又依赖13个外部包
3、与viper相比，multiconfig使用起来更简单。

好了，我们再来回顾一下我们现在面临的问题：

1、在程序里定义默认配置，这样我们就无需再在toml中定义它们了。
2、用toml配置文件中的数据override默认配置
3、用命令行或环境变量的值override从toml中读取的配置。

下面是一个说明如何使用multiconfig的例子：
```
func main() {
    m := multiconfig.NewWithPath("config.toml") // supports TOML and JSON

    // Get an empty struct for your configuration
    serverConf := new(Server)

    // Populated the serverConf struct
    m.MustLoad(serverConf) // Check for error

    fmt.Println("After Loading: ")
    fmt.Printf("%+v\n", serverConf)

    if serverConf.Enabled {
        fmt.Println("Enabled field is set to true")
    } else {
        fmt.Println("Enabled field is set to false")
    }
}
```
这个例子中的toml文件如下：
```
Name              = "koding"
Enabled           = false
Port              = 6066
Users             = ["ankara", "istanbul"]

[Postgres]
Enabled           = true
Port              = 5432
Hosts             = ["192.168.2.1", "192.168.2.2", "192.168.2.3"]
AvailabilityRatio = 8.23
```
toml映射后的go结构如下：
```
type (
    // Server holds supported types by the multiconfig package
    Server struct {
        Name     string
        Port     int `default:"6060"`
        Enabled  bool
        Users    []string
        Postgres Postgres
    }

    // Postgres is here for embedded struct feature
    Postgres struct {
        Enabled           bool
        Port              int
        Hosts             []string
        DBName            string
        AvailabilityRatio float64
    }
)
```
multiconfig的使用是不是很简单，后续与viper对比后，你会同意我的观点的。

multiconfig支持默认值，也支持显式的字段赋值需求。
支持toml、json、结构体标签（struct tags)以及环境变量。
你可以自定义配置源（例如一个远程服务器），如果你想这么做的话。
可高度扩展（通过loader接口），你可以创建你自己的loader。

下面是例子的运行结果，首先是usage help：
```
$cmdlinemulticonfig -help
Usage of cmdlinemulticonfig:
  -enabled=false: Change value of Enabled.
  -name=koding: Change value of Name.
  -port=6066: Change value of Port.
  -postgres-availabilityratio=8.23: Change value of Postgres-AvailabilityRatio.
  -postgres-dbname=: Change value of Postgres-DBName.
  -postgres-enabled=true: Change value of Postgres-Enabled.
  -postgres-hosts=[192.168.2.1 192.168.2.2 192.168.2.3]: Change value of Postgres-Hosts.
  -postgres-port=5432: Change value of Postgres-Port.
  -users=[ankara istanbul]: Change value of Users.

Generated environment variables:
   SERVER_NAME
   SERVER_PORT
   SERVER_ENABLED
   SERVER_USERS
   SERVER_POSTGRES_ENABLED
   SERVER_POSTGRES_PORT
   SERVER_POSTGRES_HOSTS
   SERVER_POSTGRES_DBNAME
   SERVER_POSTGRES_AVAILABILITYRATIO
```
```
$cmdlinemulticonfig
After Loading:
&{Name:koding Port:6066 Enabled:false Users:[ankara istanbul] Postgres:{Enabled:true Port:5432 Hosts:[192.168.2.1 192.168.2.2 192.168.2.3] DBName: AvailabilityRatio:8.23}}
Enabled field is set to false
```
检查一下输出结果吧，是不是每项都符合我们之前的预期呢！

## 五、Viper

我们的重量级选手[viper](https://github.com/spf13/viper)该出场了！

毫无疑问，viper非常强大。但如果你想用命令行参数覆盖预定义的配置项值，viper自己还不足以。要想让viper爆发，你需要另外一个包配合，它就是cobra（https://github.com/spf13/cobra）。

不同于注重简化配置处理的multiconfig，viper让你拥有全面控制力。不幸的是，在得到这种控制力之前，你需要做一些体力活。

我们再来回顾一下使用multiconfig处理配置的代码：
```
func main() {
    m := multiconfig.NewWithPath("config.toml") // supports TOML and JSON

    // Get an empty struct for your configuration
    serverConf := new(Server)

    // Populated the serverConf struct
    m.MustLoad(serverConf) // Check for error

    fmt.Println("After Loading: ")
    fmt.Printf("%+v\n", serverConf)

    if serverConf.Enabled {
        fmt.Println("Enabled field is set to true")
    } else {
        fmt.Println("Enabled field is set to false")
    }
}
```
这就是使用multiconfig时你要做的所有事情。现在我们来看看使用viper和cobra如何来完成同样的事情：
```
func init() {
    mainCmd.AddCommand(versionCmd)

    viper.SetEnvPrefix("DISPATCH")
    viper.AutomaticEnv()

    /*
      When AutomaticEnv called, Viper will check for an environment variable any
      time a viper.Get request is made. It will apply the following rules. It
      will check for a environment variable with a name matching the key
      uppercased and prefixed with the EnvPrefix if set.
    */

    flags := mainCmd.Flags()

    flags.Bool("debug", false, "Turn on debugging.")
    flags.String("addr", "localhost:5002", "Address of the service")
    flags.String("smtp-addr", "localhost:25", "Address of the SMTP server")
    flags.String("smtp-user", "", "User to authenticate with the SMTP server")
    flags.String("smtp-password", "", "Password to authenticate with the SMTP server")
    flags.String("email-from", "noreply@example.com", "The from email address.")

    viper.BindPFlag("debug", flags.Lookup("debug"))
    viper.BindPFlag("addr", flags.Lookup("addr"))
    viper.BindPFlag("smtp_addr", flags.Lookup("smtp-addr"))
    viper.BindPFlag("smtp_user", flags.Lookup("smtp-user"))
    viper.BindPFlag("smtp_password", flags.Lookup("smtp-password"))
    viper.BindPFlag("email_from", flags.Lookup("email-from"))

  // Viper supports reading from yaml, toml and/or json files. Viper can
  // search multiple paths. Paths will be searched in the order they are
  // provided. Searches stopped once Config File found.

    viper.SetConfigName("CommandLineCV") // name of config file (without extension)
    viper.AddConfigPath("/tmp")          // path to look for the config file in
    viper.AddConfigPath(".")             // more path to look for the config files

    err := viper.ReadInConfig()
    if err != nil {
        println("No config file found. Using built-in defaults.")
    }
}
```
可以看出，你需要使用BindPFlag来让viper和cobra结合一起工作。但这还不算太糟。

cobra的真正威力在于提供了subcommand能力。同时cobra还提供了与posix 全面兼容的命令行标志解析能力，包括长短标志、内嵌命令、为command定义你自己的help或usage等。

下面是定义子命令的例子代码：
```
// The main command describes the service and defaults to printing the
// help message.
var mainCmd = &cobra.Command{
    Use:   "dispatch",
    Short: "Event dispatch service.",
    Long:  `HTTP service that consumes events and dispatches them to subscribers.`,
    Run: func(cmd *cobra.Command, args []string) {
        serve()
    },
}

// The version command prints this service.
var versionCmd = &cobra.Command{
    Use:   "version",
    Short: "Print the version.",
    Long:  "The version of the dispatch service.",
    Run: func(cmd *cobra.Command, args []string) {
        fmt.Println(version)
    },
}
```
有了上面subcommand的定义，我们就可以得到如下的help信息了：
```
Usage:
  dispatch [flags]
  dispatch [command]

Available Commands:
  version     Print the version.
  help        Help about any command

Flags:
      –addr="localhost:5002": Address of the service
      –debug=false: Turn on debugging.
      –email-from="noreply@example.com": The from email address.
  -h, –help=false: help for dispatch
      –smtp-addr="localhost:25": Address of the SMTP server
      –smtp-password="": Password to authenticate with the SMTP server
      –smtp-user="": User to authenticate with the SMTP server

Use "dispatch help [command]" for more information about a command.
```
## 六、小结

以上例子的完整源码在作者的[github repository](https://github.com/suntong/lang/tree/master/lang/Go/src/sys)里可以找到。


***
关于golang配置文件，我个人用到了toml这一层次，因为不需要太复杂的配置，不需要环境变量或命令行override默认值或配置文件数据。不过 从作者的例子中可以看到multiconfig、viper的确强大，后续在实现复杂的golang应用时会考虑真正应用。
