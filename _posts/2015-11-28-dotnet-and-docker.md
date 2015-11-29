---
published: true
layout: post
title: .Net and Docker
description: running Nancy inside a docker container.
date: 2015-11-28
tags: [docker, .net, nancy]
comments: false
---

Using Mono, we can take advantage of both linux and docker, which equals some really exciting stuff. In this post we will take a C#, .NET 4.6 and Nancy "hello world" example and run this in a docker container. 

##Pre-Req

for this example, the following tools are being used (replace with your tools of choice)

- VS Code or Studio - to edit source code and compile
- Docker 1.7 + (im using vagrant *box-cutter/ubuntu1404-docker* image)
- Docker Hub
- Servers with docker on them (I like to use Rancher, as it makes this far easier to work with)

##Step 1 - code

Docker containers are designed to run a process and exit when they are complete. In our case we want a blocking application.

1. Create a normal **console line** application, using .NET 4.6 (for now im using full .NET), I callled it "TestMvc.Host" 
2. Add a reference to NancyFX (make sure you get the latest)
3. write your code, note that when you start your host, that you block the main thread using Thread.Sleep.
4. Compile your code (you can compile using Mono or MS .NET),

{% highlight csharp %}
static void Main(string[] args)
{
    Console.CancelKeyPress += (sender, eventArgs) =>
    {
        Console.WriteLine("exiting...");
    };

    Console.WriteLine("About to start server");
    using (var host = new NancyHost(new Uri("http://localhost:80")))
    {
        host.Start();

        Console.WriteLine("started, press \"ctrl + c\" to exit");
        Thread.Sleep(Timeout.Infinite);
    }
}
{% endhighlight %}

The sample Nancy Module in this test app

{% highlight csharp %}
public class HelloModule : NancyModule
{
    public HelloModule()
    {
        Get["/"] = parameters => $"Hello from {Environment.OSVersion.Platform}";
    }
}
{% endhighlight %}

**note:** the Thread.Sleep is intensional, as it will not cause the container to exit. do not use ReadKey or ReadLine.

**note 2:** we are hosting our service on port **80**.

<figure>
	<a href="http://dbones.github.io/images/posts/2015/net-docker/onWindows.JPG"><img src="http://dbones.github.io/images/posts/2015/net-docker/onWindows.JPG"></img></a>
	<figcaption><a href="http://dbones.github.io/images/posts/2015/net-docker/onWindows.JPG" title="on windows">on windows</a>.</figcaption>
</figure>

##Step 2 - Compiling a docker image

For this part I cheat a little and use VM setup via Vagrant using the *box-cutter/ubuntu1404-docker* image. 

1. Create a new folder, i will call this **releaseImage**, this will be used to used to 
2. Copy the compiled filed into the new **releaseImage**, all the dll's and exe.

<figure>
	<a href="http://dbones.github.io/images/posts/2015/net-docker/vmFolder.JPG"><img src="http://dbones.github.io/images/posts/2015/net-docker/vmFolder.JPG"></img></a>
	<figcaption><a href="http://dbones.github.io/images/posts/2015/net-docker/vmFolder.JPG" title="contents of the releaseImage folder">contents of the releaseImage folder</a>.</figcaption>
</figure>

3. Create a file called "Dockerfile" no extension, if you do this in VS code it will supply some level of syntax support.

Dockerfile

{% highlight text %}
FROM mono
COPY . /serv
CMD [ "mono",  "/serv/TestMvc.Host.exe" ]
EXPOSE 80
{% endhighlight %}