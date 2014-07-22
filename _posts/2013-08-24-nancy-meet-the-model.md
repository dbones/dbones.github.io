---
published: true
layout: post
description: Passing a model to the view
tags: [nancy]
comments: true
---
[Back Post - made available on 2014-07-22]

Lets recap, I have looked into

* Hosting Nancy in a Windows Service and Console Application using Topshelf
* Adding the time taken for a request into the Trace Log
* How to Embed views into the DLL and get them to render

so this is all exciting stuff.

In this post I will look into a really important part of any web application, how to pass the model to the view. (which was easier than I thought it was going to be, bonus)

#Update the Module
updating the HelloModule:

{% highlight csharp %}
public class HelloModule : NancyModule
{
    public HelloModule()
    {
        Get["/"] = parameters =>
        {
            string name = "Dave"; // <- uber model 
            return View["index", name];
        };
    }
}
{% endhighlight %}

Ok this is epic simple, I have a model which is a string, set to Dave. Original I know, wait until you see the view.

Jokes aside note how we pass the model into the view, we are not using a method call (**View["index", name]**), this I think will catch me out sooner or later, but this is no big deal, and there is intelli sense for the newbies.

#Update the View
The view I found to be more interesting, here is the code:

{% highlight csharp %}
@inherits Nancy.ViewEngines.Razor.NancyRazorViewBase<string>
Hello @Model
{% endhighlight %}

note here that we inherit off the NancyRazorViewBase, in studio we will now have full intelli sense. nice touch by the Nancy team.

hit F5 and browse, and we will see "Hello Dave". epic