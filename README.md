# Logrus <img src="http://i.imgur.com/hTeVwmJ.png" width="40" height="40" alt=":walrus:" class="emoji" title=":walrus:"/> [![Build Status](https://travis-ci.org/sirupsen/logrus.svg?branch=master)](https://travis-ci.org/sirupsen/logrus) [![GoDoc](https://godoc.org/github.com/sirupsen/logrus?status.svg)](https://godoc.org/github.com/sirupsen/logrus)

Logrus是一个结构化日志打印器, 与标准库完全兼容。<br>

**Logrus处于维护模式.** 我们不会介绍新功能，它太难了，会破坏很多人的项目, 这是你想从日志库中获取的最后一件事。<br>

但是这不意味着logrus已死，Logurs将会继续维护，修复bug，提升性能。<br>

我相信Logrus最大的贡献是在golang项目的中结构化日志，被广泛使用。似乎没有理由，中断迭代进入Logrus v2版本，自动go语言社区被简历起来。很多奇妙的选择跳了出来。如果使用我们所知道的重新设计，Logrus看起来像那些有关Go中的结构化日志的信息。如：[Zerolog][zerolog], [Zap][zap], [Apex][apex]。

[zerolog]: https://github.com/rs/zerolog
[zap]: https://github.com/uber-go/zap
[apex]: https://github.com/apex/log

**看到奇怪的大小写敏感问题?**<br>
在过去，可能导入Logrus的时候，既可以大写，也可以小写。但是由于Go封装环境，这在社区引起了一个争论。所以我们需要一个标准。一些环境遇到了大写变体的问题，因此决定将小写作为标准。所有使用logus的东西，都需要使用小写`github.com/sirupsen/logrus`。
<br>
对于修复Glide, 请查看 [这些评论](https://github.com/sirupsen/logrus/issues/553#issuecomment-306591437).<br>
关于这个问题的深入讨论，请查看[这些评论](https://github.com/sirupsen/logrus/issues/570#issuecomment-313933276).<br>

在开发过程中更好的使用演示编码 (当终端设备连接上时, 否则显示纯文本):

![Colored](http://i.imgur.com/PY7qMwd.png)

使用 `log.SetFormatter(&log.JSONFormatter{})`, 轻松解析logstash或Splunk:

```json
{"animal":"walrus","level":"info","msg":"A group of walrus emerges from the
ocean","size":10,"time":"2014-03-10 19:57:38.562264131 -0400 EDT"}

{"level":"warning","msg":"The group's number increased tremendously!",
"number":122,"omg":true,"time":"2014-03-10 19:57:38.562471297 -0400 EDT"}

{"animal":"walrus","level":"info","msg":"A giant walrus appears!",
"size":10,"time":"2014-03-10 19:57:38.562500591 -0400 EDT"}

{"animal":"walrus","level":"info","msg":"Tremendously sized cow enters the ocean.",
"size":9,"time":"2014-03-10 19:57:38.562527896 -0400 EDT"}

{"level":"fatal","msg":"The ice breaks!","number":100,"omg":true,
"time":"2014-03-10 19:57:38.562543128 -0400 EDT"}
```

使用默认的`log.SetFormatter(&log.TextFormatter{})`当终端设备未连上时,输出兼容
[logfmt](http://godoc.org/github.com/kr/logfmt) 格式:

```text
time="2015-03-26T01:27:38-04:00" level=debug msg="Started observing beach" animal=walrus number=8
time="2015-03-26T01:27:38-04:00" level=info msg="A group of walrus emerges from the ocean" animal=walrus size=10
time="2015-03-26T01:27:38-04:00" level=warning msg="The group's number increased tremendously!" number=122 omg=true
time="2015-03-26T01:27:38-04:00" level=debug msg="Temperature changes" temperature=-4
time="2015-03-26T01:27:38-04:00" level=panic msg="It's over 9000!" animal=orca size=9009
time="2015-03-26T01:27:38-04:00" level=fatal msg="The ice breaks!" err=&{0x2082280c0 map[animal:orca size:9009] 2015-03-26 01:27:38.441574009 -0400 EDT panic It's over 9000!} number=100 omg=true
```

为了确保这个行为，即使附加了TTY，也要按以下来设置：

```go
	log.SetFormatter(&log.TextFormatter{
		DisableColors: true,
		FullTimestamp: true,
	})
```

#### 日志方法名
如果你希望增加这个调用方法到一个字段中，通过指示这个记录器：
```go
log.SetReportCaller(true)
```
这里将会增加一个调用的方法
```json
{"animal":"penguin","level":"fatal","method":"github.com/sirupsen/arcticcreatures.migrate","msg":"a penguin swims by",
"time":"2014-03-10 19:57:38.562543129 -0400 EDT"}
```

```text
time="2015-03-26T01:27:38-04:00" level=fatal method=github.com/sirupsen/arcticcreatures.migrate msg="a penguin swims by" animal=penguin
```
注意这样将会增加可衡量的开销。这个花费根据go的版本不一样而不同，但是介于20%到40%之间（在最近测试的1.6和1.7中）。
```
go test -bench=.*CallerTracing
```

#### 用例
使用Logurs最简单的方式就是打包级别的到处记录器。

```go
package main

import (
  log "github.com/sirupsen/logrus"
)

func main() {
  log.WithFields(log.Fields{
    "animal": "walrus",
  }).Info("A walrus appears")
}
```
注意，logrus完全与标准库中的log包兼容，所以你可以替代你的log包，而使用`log "github.com/sirupsen/logrus"`，然后你将有更灵活的日志工具。你可以按照你的需要做自定义设置。
```go
package main

import (
  "os"
  log "github.com/sirupsen/logrus"
)

func init() {
  // Log as JSON instead of the default ASCII formatter.
  log.SetFormatter(&log.JSONFormatter{})

  // Output to stdout instead of the default stderr
  // Can be any io.Writer, see below for File example
  log.SetOutput(os.Stdout)

  // Only log the warning severity or above.
  log.SetLevel(log.WarnLevel)
}

func main() {
  log.WithFields(log.Fields{
    "animal": "walrus",
    "size":   10,
  }).Info("A group of walrus emerges from the ocean")

  log.WithFields(log.Fields{
    "omg":    true,
    "number": 122,
  }).Warn("The group's number increased tremendously!")

  log.WithFields(log.Fields{
    "omg":    true,
    "number": 100,
  }).Fatal("The ice breaks!")

  // A common pattern is to re-use fields between logging statements by re-using
  // the logrus.Entry returned from WithFields()
  contextLogger := log.WithFields(log.Fields{
    "common": "this is a common field",
    "other": "I also should be logged always",
  })

  contextLogger.Info("I'll be logged with common and other field")
  contextLogger.Info("Me too")
}
```
对于更高级的用法，例如从同一日志记录到多个位置应用程序，还可以创建`logrus` Logger的实例

```go
package main

import (
  "os"
  "github.com/sirupsen/logrus"
)

// Create a new instance of the logger. You can have any number of instances.
var log = logrus.New()

func main() {
  // The API for setting attributes is a little different than the package level
  // exported logger. See Godoc.
  log.Out = os.Stdout

  // You could set this to any `io.Writer` such as a file
  // file, err := os.OpenFile("logrus.log", os.O_CREATE|os.O_WRONLY|os.O_APPEND, 0666)
  // if err == nil {
  //  log.Out = file
  // } else {
  //  log.Info("Failed to log to file, using default stderr")
  // }

  log.WithFields(logrus.Fields{
    "animal": "walrus",
    "size":   10,
  }).Info("A group of walrus emerges from the ocean")
}
```

#### 字段
Logrus鼓励用细致、结构化的日志。通过日志字段代替冗长、难以解析的错误消息。例如，替代`log.Fatalf("Failed
to send event %s to topic %s with key %d")`，你应该记录更多可以发现的：

```go
log.WithFields(log.Fields{
  "event": event,
  "topic": topic,
  "key": key,
}).Fatal("Failed to send event")
```
我们发现这个api将会迫使你考虑记录更多有用的日志信息。我们经历过无数情况，仅仅简单的增加一个日志字段，就已经节约了我们很多时间。`WithFields`调用是可选的。
一般来说，使用Logrus中`printf`一系列函数作为提示时，应该加上一个字段。但是，你仍然可以使用`printf`一系列函数。

#### 默认字段
通常，将字段_always_附加到日志语句中很有帮助。例如，你可能想始终记录`request_id` and `user_ip`，而不是重复的写`log.WithFields(log.Fields{"request_id": request_id, "user_ip": user_ip})`，你可以创建一个`logrus.Entry`来替代：

```go
requestLogger := log.WithFields(log.Fields{"request_id": request_id, "user_ip": user_ip})
requestLogger.Info("something happened on that request") # will log request_id and user_ip
requestLogger.Warn("something not great happened")
```

#### 钩子

你可以添加日志级别的钩子。例如：发送错误给一个额外的追踪服务，将信息发送到StatsD或者多个地方，如日志系统。
Logrus附带[钩子](hooks/)，可以在init函数中引入附带的钩子，或者引入你自定义的钩子。

```go
import (
  log "github.com/sirupsen/logrus"
  "gopkg.in/gemnasium/logrus-airbrake-hook.v2" // the package is named "airbrake"
  logrus_syslog "github.com/sirupsen/logrus/hooks/syslog"
  "log/syslog"
)

func init() {

  // Use the Airbrake hook to report errors that have Error severity or above to
  // an exception tracker. You can create custom hooks, see the Hooks section.
  log.AddHook(airbrake.NewHook(123, "xyz", "production"))

  hook, err := logrus_syslog.NewSyslogHook("udp", "localhost:514", syslog.LOG_INFO, "")
  if err != nil {
    log.Error("Unable to connect to local syslog daemon")
  } else {
    log.AddHook(hook)
  }
}
```

注意: 日志系统钩子也支持本地日志系统(例如："/dev/log" 或者 "/var/run/syslog" 或者 "/var/run/log"). 关于详情, 请查阅 [syslog hook README](hooks/syslog/README.md).

一系列当前已知的服务钩子可以在wiki中找到[page](https://github.com/sirupsen/logrus/wiki/Hooks)


#### 日志级别
Logrus 已经有7个日志级别 （跟踪）Trace, （调试）Debug, （信息）Info, （警告）Warning,（错误）Error, （致命）Fatal and （恐慌）Panic.

```go
log.Trace("Something very low level.")
log.Debug("Useful debugging information.")
log.Info("Something noteworthy happened!")
log.Warn("You should probably take a look at this.")
log.Error("Something failed but I'm not quitting.")
// Calls os.Exit(1) after logging
log.Fatal("Bye.")
// Calls panic() after logging
log.Panic("I'm bailing.")
```
你可以设置日志级别在`Logger`中，接下来它就仅仅记录这个级别或这个级别以上的日志：

```go
// Will log anything that is info or above (warn, error, fatal, panic). Default.
log.SetLevel(log.InfoLevel)
```
在调试或冗长环境中设置`log.Level = logrus.DebugLevel`可能是有用的。

#### 输入
除了在字段中添加 `WithField` 或 `WithFields`，一些字段是自动添加到所有日志记录时间中的：

1. `time`   时间戳，记录创建时自动添加。
2. `msg`   日志消息传递给 `{Info,Warn,Error,Fatal,Panic}` 后，`AddFields` 调用。 例如： `Failed to send event.`
3. `level` 日志级别。 例如： `info`.

#### Environments

Logrus没有环境的概念。

如果你希望钩子和格式化程序仅仅在特殊环境中使用，你应该自己处理。例如，如果你的应用程序就有全局变量`Environment`，它可以用来表示所处环境：

```go
import (
  log "github.com/sirupsen/logrus"
)

init() {
  // do something here to set environment depending on an environment variable
  // or command-line flag
  if Environment == "production" {
    log.SetFormatter(&log.JSONFormatter{})
  } else {
    // The TextFormatter is default, you don't actually have to do this.
    log.SetFormatter(&log.TextFormatter{})
  }
}
```

这种配置是预期使用`logrus`，但是JSON在生产环境下使用，如果你做日志聚合，使用Splunk或Logstash。

#### 格式程序

* `logrus.TextFormatter`. 如果是用TTY输出，则用颜色记录事件，否则没有颜色。
  * *注意:* 强制使用颜色输出（即使没有TTY）, 设置 `ForceColors`字段为 `true`.  强制不使用颜色输出（即使有TTY），设置`DisableColors` 字段为`true`. 对于windows, 请查阅
    [github.com/mattn/go-colorable](https://github.com/mattn/go-colorable).
* 启用颜色后，默认情况下级别将被截断为4个字符。如果要禁止截断，设置`DisableLevelTruncation`字段为 `true`.
* 当输出到TTY时, 可视地向下扫描所有级别均相同宽度的列通常会很有帮助。设置`PadLevelText`字段为 `true` 通过将填充添加到级别文本来启用此行为。
  * 所有选项都列在[generated docs](https://godoc.org/github.com/sirupsen/logrus#TextFormatter).
* `logrus.JSONFormatter`. 将字段记录为JSON。
  * 所有选项都列在 [generated docs](https://godoc.org/github.com/sirupsen/logrus#JSONFormatter).

第三方日志格式化程序：

* [`FluentdFormatter`](https://github.com/joonix/log). Formats entries that can be parsed by Kubernetes and Google Container Engine.
* [`GELF`](https://github.com/fabienm/go-logrus-formatters). Formats entries so they comply to Graylog's [GELF 1.1 specification](http://docs.graylog.org/en/2.4/pages/gelf.html).
* [`logstash`](https://github.com/bshuster-repo/logrus-logstash-hook). Logs fields as [Logstash](http://logstash.net) Events.
* [`prefixed`](https://github.com/x-cray/logrus-prefixed-formatter). Displays log entry source along with alternative layout.
* [`zalgo`](https://github.com/aybabtme/logzalgo). Invoking the Power of Zalgo.
* [`nested-logrus-formatter`](https://github.com/antonfisher/nested-logrus-formatter). Converts logrus fields to a nested structure.
* [`powerful-logrus-formatter`](https://github.com/zput/zxcTool). get fileName, log's line number and the latest function's name when print log; Sava log to files.
* [`caption-json-formatter`](https://github.com/nolleh/caption_json_formatter). logrus's message json formatter with human-readable caption added.

You can define your formatter by implementing the `Formatter` interface,
requiring a `Format` method. `Format` takes an `*Entry`. `entry.Data` is a
`Fields` type (`map[string]interface{}`) with all your fields as well as the
default ones (see Entries section above):

```go
type MyJSONFormatter struct {
}

log.SetFormatter(new(MyJSONFormatter))

func (f *MyJSONFormatter) Format(entry *Entry) ([]byte, error) {
  // Note this doesn't include Time, Level and Message which are available on
  // the Entry. Consult `godoc` on information about those fields or read the
  // source of the official loggers.
  serialized, err := json.Marshal(entry.Data)
    if err != nil {
      return nil, fmt.Errorf("Failed to marshal fields to JSON, %v", err)
    }
  return append(serialized, '\n'), nil
}
```

#### Logger as an `io.Writer`

Logrus can be transformed into an `io.Writer`. That writer is the end of an `io.Pipe` and it is your responsibility to close it.

```go
w := logger.Writer()
defer w.Close()

srv := http.Server{
    // create a stdlib log.Logger that writes to
    // logrus.Logger.
    ErrorLog: log.New(w, "", 0),
}
```

Each line written to that writer will be printed the usual way, using formatters
and hooks. The level for those entries is `info`.

This means that we can override the standard library logger easily:

```go
logger := logrus.New()
logger.Formatter = &logrus.JSONFormatter{}

// Use logrus for standard log output
// Note that `log` here references stdlib's log
// Not logrus imported under the name `log`.
log.SetOutput(logger.Writer())
```

#### Rotation

Log rotation is not provided with Logrus. Log rotation should be done by an
external program (like `logrotate(8)`) that can compress and delete old log
entries. It should not be a feature of the application-level logger.

#### Tools

| Tool | Description |
| ---- | ----------- |
|[Logrus Mate](https://github.com/gogap/logrus_mate)|Logrus mate is a tool for Logrus to manage loggers, you can initial logger's level, hook and formatter by config file, the logger will be generated with different configs in different environments.|
|[Logrus Viper Helper](https://github.com/heirko/go-contrib/tree/master/logrusHelper)|An Helper around Logrus to wrap with spf13/Viper to load configuration with fangs! And to simplify Logrus configuration use some behavior of [Logrus Mate](https://github.com/gogap/logrus_mate). [sample](https://github.com/heirko/iris-contrib/blob/master/middleware/logrus-logger/example) |

#### Testing

Logrus has a built in facility for asserting the presence of log messages. This is implemented through the `test` hook and provides:

* decorators for existing logger (`test.NewLocal` and `test.NewGlobal`) which basically just adds the `test` hook
* a test logger (`test.NewNullLogger`) that just records log messages (and does not output any):

```go
import(
  "github.com/sirupsen/logrus"
  "github.com/sirupsen/logrus/hooks/test"
  "github.com/stretchr/testify/assert"
  "testing"
)

func TestSomething(t*testing.T){
  logger, hook := test.NewNullLogger()
  logger.Error("Helloerror")

  assert.Equal(t, 1, len(hook.Entries))
  assert.Equal(t, logrus.ErrorLevel, hook.LastEntry().Level)
  assert.Equal(t, "Helloerror", hook.LastEntry().Message)

  hook.Reset()
  assert.Nil(t, hook.LastEntry())
}
```

#### Fatal handlers

Logrus can register one or more functions that will be called when any `fatal`
level message is logged. The registered handlers will be executed before
logrus performs an `os.Exit(1)`. This behavior may be helpful if callers need
to gracefully shutdown. Unlike a `panic("Something went wrong...")` call which can be intercepted with a deferred `recover` a call to `os.Exit(1)` can not be intercepted.

```
...
handler := func() {
  // gracefully shutdown something...
}
logrus.RegisterExitHandler(handler)
...
```

#### Thread safety

By default, Logger is protected by a mutex for concurrent writes. The mutex is held when calling hooks and writing logs.
If you are sure such locking is not needed, you can call logger.SetNoLock() to disable the locking.

Situation when locking is not needed includes:

* You have no hooks registered, or hooks calling is already thread-safe.

* Writing to logger.Out is already thread-safe, for example:

  1) logger.Out is protected by locks.

  2) logger.Out is an os.File handler opened with `O_APPEND` flag, and every write is smaller than 4k. (This allows multi-thread/multi-process writing)

     (Refer to http://www.notthewizard.com/2014/06/17/are-files-appends-really-atomic/)
