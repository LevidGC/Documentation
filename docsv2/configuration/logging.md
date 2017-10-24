---
layout: docs-default
---

# 日志 (Logging)

IdentityServer 有两个日志相关的特性。开发时日志和生产时事件（[参见这里](events.html)）。

开发时日志会产生相当多的输出，大多数对开发者自定义 IdentityServer 来说都有用。日志可能存储像密码之类的敏感信息，因此对于生成时使用这通常是不适合的。

IdentityServer 使用 [LibLog](https://github.com/damianh/LibLog) 完成日志。Liblog 会自动选取以下的日志类库：

* NLog
* Enterprise Library
* SeriLog
* Log4Net
* Loupe

这里没有与 IdentityServer3 相关的配置——您只需要在宿主中配置当中的一个日志框架即可。

*警告：LibLog 会选取排序最上面的一个类库而将其余的全部抛弃。所以举个例子，如果您已经对 SeriLog 有了一个引用，那么再尝试配置 Log4net 将 **不会** 起作用。*

## 配置诊断 (Configuring Diagnostics)

`LoggingOptions` 类有以下设置：

* `EnableWebApiDiagnostics`
   * 如果启用，Web API 内部诊断日志将会转发给日志提供器
* `WebApiDiagnosticsIsVerbose`
   * 如果启用，Web API 诊断日志将会被设置为 verbose 
* `EnableHttpLogging`
   * 如果启用，HTTP 请求和响应将会被记录
* `EnableKatanaLogging`
   * 如果启用，Katana 日志输出将会被记录（这常常对于诊断外部身份提供器有用）

## 示例：使用 Serilog 记录 System.Diagnostics 跟踪 (Example: Using Serilog to log to System.Diagnostics tracing)

下面的示例整合进了 [Serilog](http://serilog.net/) 并用它来记录诊断跟踪（将其放置在 Startup 或者您的托管代码中）。
**Note:** Serilog provides various logging sinks as separate packages, so you may need to install the Serilog.Sinks.Trace package to get `WriteTo.Trace()` to work as expected. 

```csharp
Log.Logger = new LoggerConfiguration()
    .MinimumLevel.Debug()
    .WriteTo.Trace()
    .CreateLogger();
```

Add the following snippet to your configuration file to funnel all logging messages to a simple text file.
We use [Baretail](https://www.baremetalsoft.com/baretail/) for viewing the log files.

```xml
<system.diagnostics>
  <trace autoflush="true"
         indentsize="4">
    <listeners>
      <add name="myListener"
           type="System.Diagnostics.TextWriterTraceListener"
           initializeData="Trace.log" />
      <remove name="Default" />
    </listeners>
  </trace>
</system.diagnostics>
```

**Note:** If you use this method you need to ensure that the account running the application pool has write access 
to the directory containing the log file. 
If you don't specify a path, this will be the application directory, which is not recommended for production scenarios. 
For production log to a file **outside** the application directory.

## Example: Log to the console
Logging to the console gives you a friction free and immediate insight into the internals of IdentityServer. Serilog has a nice colored
console logging sink called `Serilog.Sinks.Literate`. Wire it up like this:

```csharp
Log.Logger = new LoggerConfiguration()
    .MinimumLevel.Debug()
    .WriteTo.LiterateConsole()
    .CreateLogger();
```

## Instrumenting your own code
You can also use the logging system in your own extensibility code.

Add a static `ILog` instance to your class

```csharp
private readonly static ILog Logger = LogProvider.For<MyClass>();
```
Log your messages using the logger

```csharp
Logger.Debug("Getting claims for identity token");
```

## Using your own Logging Infrastructure
You may have an existing logging infrastructure in place and want IdentityServer logging to use that.
The recommended approach for this is to write a custom sink using one of the supported logging frameworks (our favourite is Serilog).
You can find a sample [here](https://github.com/IdentityServer/IdentityServer3.Samples/tree/dev/source/Logging).

See [here](http://leastprivilege.com/2015/10/22/identityserver3-logging-monitoring-using-serilog-and-seq/) for a post about logging and eventing.

## Suppressing all logging output
**(added in v2.5)**

For certain scenarios (e.g. production) you want to make sure that no logging output is produced. For this you can configure a no-op logger (typically done in `Startup` or in the hosting code):

```csharp
LogProvider.SetCurrentLogProvider(new NoopLogProvider());
```
