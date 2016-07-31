---
published: true
layout: post
description: how can we see what is happening under the hood
tags: [nancy]
comments: true
---
[Back Post - made available on 2014-07-22]

One key aspect of a framework is to gain tracing and diagnostic information. One thing I find useful is time taken for a request, and in this post I look into how this information can be gathered.

The good news is Nancy includes a diagnostic/set-up site located @

>your-site/_Nancy

in my example it is: 

>http://localhost:1234/_Nancy

# Set-up diagnostics
By default you cannot access this site, it will ask you to set-up a password. 

To do this we need to create a bootstrap class. This is used to enable the route tracing and also provide the password to the _Nancy diagnostics site.

{% highlight csharp %}
public class CustomBootstrapper : DefaultNancyBootstrapper
{
    protected override void ApplicationStartup(TinyIoCContainer container, IPipelines pipelines)
    {
        StaticConfiguration.EnableRequestTracing = true;

        //code to be added here
    }

    protected override DiagnosticsConfiguration DiagnosticsConfiguration
    {
        get { return new DiagnosticsConfiguration { Password = @"A2mVtH/XRT\p,B" }; }
    }
}
{% endhighlight %}

This example is using the default bootstrap, which uses *TinyIoC*, you can provide another base implementation.

# Add request timing
In version 0.18.0 the request tracing does not include the time taken for a request. however its not hard to add it. The framework provides a Trace log which we can write to during a request.

In the code snippet above replace the following line:

    //code to be added here

With:

{% highlight csharp %}
//see how long a request took
pipelines.BeforeRequest.AddItemToStartOfPipeline(
    ctx =>
    {
        Stopwatch timer = Stopwatch.StartNew();
        ctx.Items.Add("timer", timer);
        return null;
    });

pipelines.AfterRequest.AddItemToEndOfPipeline(
    ctx =>
    {
        if (!ctx.Items.ContainsKey("timer"))
        {
            return; //the diagnostics site will not have this key, unsure why
        }
        var timer = (Stopwatch)ctx.Items["timer"];
        timer.Stop();
        ctx.Trace.TraceLog.WriteLog(log => log.AppendLine(string.Format("Request took: {0} ms", timer.ElapsedMilliseconds)));
    });
{% endhighlight %}

now each request will be timed on how long they take. To view it open the tracing select a session, then select a request, at the bottom of the page there is the Trace log, where this information is added.