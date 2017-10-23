---
layout: docs-default
---

验证中间件使用标准的 Katana [日志](https://github.com/aspnet/AspNetKatana) 设施。

默认情况，Katana 在 .NET 中使用 `TraceSource` 机制来做日志。将以下片段添加到您的配置文件来将日志记录到文件： 

```xml
<system.diagnostics>
  <trace autoflush="true" />

  <sources>
    <source name="Microsoft.Owin">
      <listeners>
        <add name="KatanaListener" />
      </listeners>
    </source>
  </sources>

  <sharedListeners>
    <add name="KatanaListener"
          type="System.Diagnostics.TextWriterTraceListener"
          initializeData="katana.trace.log"
          traceOutputOptions="ProcessId, DateTime" />
  </sharedListeners>

  <switches>
    <add name="Microsoft.Owin"
          value="Verbose" />
  </switches>
</system.diagnostics>
```