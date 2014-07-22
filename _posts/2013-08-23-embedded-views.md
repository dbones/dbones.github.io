---
layout: post
description: "Using Embedded views, the Razor ViewEngine, with Nancy self host."
tags: [nancy]
comments:true
published: true
---
[Back Post - made available on 2014-07-22]

So far I have returned "Hello world" from the HelloWorldModule, and added a quick performance counter to the trace log. My running example is using Nancy's self hosting.

In this post, I wanted to start using the Razor ViewEngine.

#Register the engine
Add a reference to the Razor package
> Nancy.Viewengines.Razor

Now we support razor, well kind of (we will get to that). For now if you look at the server information the view engine is listed.

#Add a simple view
I wanted to start off with a simple view (*helloworld* go figure)

Alter the module to return a view, not using a model yet.

{% highlight csharp %}
public class HelloModule : NancyModule
{
    public HelloModule()
    {
        Get["/"] = parameters =>
        {
            return View["index"]; 
        };
    }
}
{% endhighlight %}
    
*side note - the module acts remarkably like a Controller (what you would find in ASP.NET MVC)*

Add a Index.cshtml file, in a folder call Views -> Home, with the following contents

    Hello world

I was not kidding, just wanted to see if I can get this to work.

overview of the folder structure:

    TestNancy 
    +   Modules
    |   + HelloModule.cs
    +   Views
        + Home
            + Index.cshtml        

Run this and go to the web page, and it will show a simple error page, this is because the web site route directory is in the bin directory

>Currently available view engine extensions: sshtml,html,htm,cshtml,vbhtml
Locations inspected: views/Hello/index-en-GB,views/Hello/index,Hello/index-en-GB,Hello/index,views/index-en-GB,views/index,index-en-GB,index
Root path: D:\Dev\vs default\TestNancy\TestNancy\bin\Debug

The last bit provides the context, so this makes sense. and for me I want to get the embedded views to work. so i will solve this issue with embedded views.

#ResourceViewLocationProvider to the rescue
This bit took longer than i would like to mention, the main bit to note

* ensure you override the correct methods in the bootstrapper
* ensure you add the right assembly
* ensure you add the folder which contains the view
* ensure you have set the cshtml file as an embedded resource (i did not forget this one at least, win for me)

if you do not do this then you may get the following error

>Only one view was found in assembly TestNancy, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null, but no rootnamespace had been registered.

Ok now we cleared that up here is the code which you belongs to the **BootStrap** class

{% highlight csharp %}
protected override void ConfigureApplicationContainer(TinyIoCContainer container)
{
    base.ConfigureApplicationContainer(container);
    //ensure you put the correct assembly in here!
    ResourceViewLocationProvider.RootNamespaces.Add(typeof(HelloModule).Assembly, "TestNancy.Views.Home");
}

protected override NancyInternalConfiguration InternalConfiguration
{
    get
    {
        return NancyInternalConfiguration.WithOverrides(
            cfg =>
            {
                cfg.ViewLocationProvider = typeof (ResourceViewLocationProvider);
            });
    }
}
{% endhighlight %}

If we run our sample now we can now browse our site and see *Hello World* again. Check the trace logs, this provides a little more information. The process time has gone up a little, as you would expect 111ms from cold and 3ms when warm (this is running on my dev laptop, I may write a post on that later, but for now i7 4HGz, 32GB RAM and 7200RPM HDD)

#Whats missing still
I still need to look into

* static resources, jpg, css etc
* using a Model

I am curious on how we will get the folder location, IE Server.MapPath, sure this has been considered.

*side note, im using version 0.18.0, the Embedded Content support is still being worked on [https://github.com/NancyFx/Nancy/issues/732](https://github.com/NancyFx/Nancy/issues/732)*