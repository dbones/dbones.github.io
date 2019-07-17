---
published: true
layout: post
title: .net core 3 - the new host.
description: .net core 3 - host now supports the port and adapter concept
date: 2019-07-16
tags: [dotnet]
comments: false
---

.NET core 3 has a few changes, one of interest is the Host, some may look at it and this that this is a small change.

However, to me, it has a large impact.... 

## TLDR;

- the new host provides better support for ports and adapters.
- this is an answer for my question:  [WebHost as a IHostedService](https://github.com/aspnet/Hosting/issues/1369)
- more info on upgrading: [Migrate from ASP.NET Core 2.2 to 3.0](https://docs.microsoft.com/en-us/aspnet/core/migration/22-to-30?view=aspnetcore-2.2&tabs=visual-studio)

## Background

Im a fan of Hexagonal/Clean/Onion Architecture, if we look at the first one (hexagonal) also called Ports and Adapters, where the

- Ports are the exposed domain's functionality, such as domain services or Commands and Queries.
- Adapters, adapt messages from external technologies to the ports, such as Web Controllers, AMQP Consumers etc

In Services and Micro-services some people forget that HTTP is an adapter which may OPTIONALLY be supported, we may have a service which interacts only with AMQP.

The HTTP should be just another Hosted (background) Service and made configurable like another other Adapter

# Could this have been done in .NET 2.2

From an initial look around it seemed that the 2.2 WebHost 

- did not run the HTTP webserver as a simple background service
- would instantiate the background services on its start.

It felt like there was a lot going on there.

Some people looked into running the WebHost in its own Background module before:

- [Generic Host - .net core 2.2](https://dev.to/jeikabu/microservice-based-application-with-aspnet-core-generic-host-cel) - the team here move the ```WebHostBuilder``` into a background service.

Its actually a pretty cool workaround, but....

# Enter .Net Core 3

In .NET core 3 we can setup the application like so:

```
//.NET Core 3 (preview)
public class Program
{
    public static void Main(string[] args)
    {
        CreateHostBuilder(args).Build().Run();
    }

    public static IHostBuilder CreateHostBuilder(string[] args) =>
        Host.CreateDefaultBuilder(args)
            .ConfigureWebHostDefaults(webBuilder =>
            {
                webBuilder.UseStartup<Startup>();
            });
}
```

hang on, what did it look like

```
   public static IWebHostBuilder CreateWebHostBuilder(string[] args) =>
      WebHost.CreateDefaultBuilder(args)
         .UseStartup<Startup>();
```

At first glance this looks like a minimal change and on the surface it is, you change ```IWebHostBuilder``` to ```IHostBuilder```.

In addition we notice that we can easily enable and disable HTTP support  by using the ```ConfigurableWebHost```, treating this adapter in a modular fashion like any other Adapter :)



# Conclusion

This is great for me as I can configure my host is a consistent way.

the workaround can be removed in favor of this new way.

I also hope this helps other 3rd party HTTP frameworks an easier way to work with .NET Core moving forward.

ps. thanks to the .NET team for making this change.