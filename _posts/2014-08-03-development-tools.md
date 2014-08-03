---
published: true
layout: post
title: Development tools
description: CI and the tools to do so
date: 2014-08-03
tags: [ci]
comments: true
---

There are a number of tools out there which help the development team deliver. In this post I will provide an over-view of flow which I have use successfully. 

These notes are a little dated, and do not include PaaS in the solution, however this is not to say that we could alter and append the tools in the correct place. (I wanted to post then up as they seem to be collecting dust on my harddrive.)

##Overview
The following diagram **overviews** the main points where each tool plays a part in a continuous integration scenario. This solution favours a package management component and a release management tool.

<figure>
	<a href="http://dbones.github.io/images/posts/2014/ci_overviewGeneric.jpg"><img src="http://dbones.github.io/images/posts/2014/ci_overviewGeneric.jpg"></a>
	<figcaption><a href="http://dbones.github.io/images/posts/2014/ci_overviewGeneric.jpg" title="CI Overview">Ci overview</a>.</figcaption>
</figure>

1.	Designs have reached a level of confidence, and are now ready to be developed. *The development can evolve the design, which needs to be updated*
2.	Import all 3rd party components required to develop the software
3.	Check-in code into the projects source repository
4.	The build server takes the latest check-in (source code files) and 3rd party packages. It then runs though and entire build process.
5.	(For web applications only) On a successful build of the project, a deployment package will be made available.
6.	(For web applications only) A staff member (product owner) can now release the software to Quality Assurance servers, to allow the start of acceptance testing
7.	(For web applications only) upon approval, a staff member can deploy the latest package to the official site.
8.	(For library packages) A staff member can deploy the latest version into the 3rd party package repository

###Build
Step 4 is a complex and very important step, the following is the overview of this step.

<figure>
	<a href="http://dbones.github.io/images/posts/2014/ci_build.png"><img src="http://dbones.github.io/images/posts/2014/ci_build.png"></a>
	<figcaption><a href="http://dbones.github.io/images/posts/2014/ci_build.png" title="CI Build">Ci Build</a>.</figcaption>
</figure>

* Get Latest – pulls the source files from the projects source repository.
* Get Dependencies – downloads all the correct versions of all the 3rd party decencies (Nuget packaged).
* Build – compiles the project’s .Net solution for “release”, this normally complies each .Net project in the .Net solution.
* Unit Tests – runs all unit tests (this includes BDD tests), produce reports (passed/failed tests and code coverage)
* Bundle – produces the output artifacts for the project (Zip, Nuget or Folder) ready for consumption.
* Test Deploy – (Web applications only) automatically deploys the application onto the build environment.
* Smoke Tests – executes automated tests which invoke actions on the UI ensuring the expected behaviour is observed.

##Components
above was a generic overview, the following has been revised with my personal favourites. (for a .NET setup, some tools will obviously change if you are a Java or Ruby developer)

<figure>
	<a href="http://dbones.github.io/images/posts/2014/ci_overviewTools.jpg"><img src="http://dbones.github.io/images/posts/2014/ci_overviewTools.jpg"></a>
	<figcaption><a href="http://dbones.github.io/images/posts/2014/ci_overviewTools.jpg" title="CI Overview tools">Ci overview with tools</a>.</figcaption>
</figure>

* Design - Enterprise Architect
* Develop - Visual Studio (Resharper, Ants profiler, SourceTree)
* Source Control - Stash
* Build - Team City
* Package Management - Nuget
* Deploy - Octopus Deploy (I really like the look of this one, but have not used it yet)

I do not mind using subversion, but I have found git with gitflow to be really nice to work with.

##Gated check-ins?
We all make mistakes, but you do not need gated check-ins. No.

The above will only package up and make available, code which passes automated checks. It also relies upon the developers being professional. (The release control should be done via senior members of the team.)

You can also use GitFlow's features to isolate code check-ins. When a developer is working on code then we do not want to lose it. :) let them check it in.

