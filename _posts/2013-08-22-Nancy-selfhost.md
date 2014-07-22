---
published: true
layout: post
description: Topshelf and Nancy selfhosting, a helloworld
tags: [helloworld, topshelf, nancy]
comments: true
---
In this post I will set-up Nancy as a windows service with the help of TopShelf.

## What is Nancy first?
Nancy as is a small http framework which allows .NET developers to deliver content such as Web pages and services (RESTful) really easily. 

# Create project
Create a console application (I called mine TestNancy), and then add the following nuget packages:

* TopShelf  (3.1.1)
* Nancy.Hosting.Self  (0.18.0)

# Set-up TopShelf 
TopShelf is my favoured way to handle windows services. The following code will create and set-up a windows service

    class Program
    {
        static void Main(string[] args)
        {
            HostFactory.Run(
                cfg =>
                {
                    cfg.Service<App>(
                        srv =>
                        {
                            srv.ConstructUsing(app => new App());
                            srv.WhenStarted(app => app.Start());
                            srv.WhenStopped(app => app.Stop());
                        });
                    
                    cfg.RunAsLocalSystem();
                    cfg.SetServiceName("NancyWebServer");
                    cfg.SetDisplayName("NancyWebServer");
                });
        }
    }

#Create the App class
The App class is a simple way for to encapsulate the application start and stop, at this point we will create/start the Nancy host, using Port 1234, and also close/dispose of it when we are done.

    public class App
    {
        NancyHost _nancyHost;

        public void Start()
        {
            _nancyHost = new NancyHost(new Uri("http://localhost:1234"));
            _nancyHost.Start();
        }

        public void Stop()
        {
            _nancyHost.Stop();
            _nancyHost.Dispose();
        }
        
    }

#Set-up a module
Nancy uses modules to set-up the server, in this example we are going to copy the code off the wiki.

    public class HelloModule : NancyModule
    {
        public HelloModule()
        {
            Get["/"] = parameters => "Hello World";
        }
    }

#Run
Run the application, **F5** should open a console application, open up you browser and navigate to **http://localhost:1234**, it should just display:

>Hello World


Thats it, we have created a fully working hello world.