---
layout: post
date: 2023-05-09
categories: tips, etw, c#
title: Create custom ETW events
---

It is often useful to use `Console.WriteLine` to debug code and see what a variable's value is at a point in code. But this approach is not useful for production testing and code can be left in by accident. This is where Event Tracing for Windows (ETW) comes in, it is an event logging system built into windows that only logs when a listener is attached to the process.

An example of a simple event logger is:

```
using System.Diagnostics.Tracing;

namespace MyApp.Tracing;

[EventSource(Name = "MyEvents")]
public class MyEvents : EventSource
{
    public static readonly MyEvents Log = new();
    public class Keywords
    {
        public const EventKeywords Request = (EventKeywords)1;
    }
    public class Tasks
    {
        public const EventTask Request = (EventTask)1;
    }
    internal const int RequestStartId = 1;
    internal const int RequestStopId = 2;

    [Event(RequestStartId, Level = EventLevel.Informational, Opcode = EventOpcode.Start, Task = Tasks.Request, Keywords = Keywords.Request)]
    public void RequestStart(string name)
    {
        if (IsEnabled())
        {
            WriteEvent(RequestStartId, name);
        }
    }

    [Event(RequestStopId, Level = EventLevel.Informational, Opcode = EventOpcode.Stop, Task = Tasks.Request, Keywords = Keywords.Request)]
    public void RequestStop(long elapsedMilliseconds = 0)
    {
        if (IsEnabled())
        {
            WriteEvent(RequestStopId, elapsedMilliseconds);
        }
    }
}
```

Events can then be logged in your code by calling `MyEvents.Log.RequestStart("name")` and `MyEvents.Log.RequestStop(100)`.

The two event functions contain a check for `IsEnabled()` that will mean the event is only recorded if there is a listener attached to the process. Make any expensive calls during the logging process within this check so that you dont slow the process when you are not recording.

The Keywords and Tasks enums enable you to create your own groupings that can be shown when viewing the events to enable you to filter by specific events.

When you have the program running you can attach a program that captures events, such as [PerfView](https://github.com/microsoft/perfview/releases) or [dotnet trace](https://learn.microsoft.com/en-us/dotnet/core/diagnostics/dotnet-trace). To capture the events you will have to specify the provider, it is passed as a comma seperated string where each provider is in the format `provider:kewords:level:values`, if not specified keywords and levels default to all. So for recording the events above you would use a provider string of `*MyEvents` 

If you are also using an `EventCounter` in your event logger you would set the provider with `*MyEvents:EventCounterIntervalSec=1`, this will record the counter value every 1 second. 

Because these events only fire if a process is listening for them you can leave them in the code when it goes up to production. This also means that if you have an issue on production you can attach a trace tool to the production process to capture the events.