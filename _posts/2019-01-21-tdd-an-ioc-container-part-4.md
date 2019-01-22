---
published: true
layout: post
title: TDD an IoC container - part 4 - Tools
description: using TDD to develop a simple IoC container.
date: 2019-01-24
tags: [dotnet, tdd]
comments: false
---

This post series looks into how I apply TDD in order to have **confidence** in what is developed.

In this post I wanted to look a little into the tools used to continuously run the tests, which ensured that we did not break the contract while we have been refactoring the code.

## catch-up

At this point we have

- confirmed how the tests should drive the code (at the moment i would say this is mostly followed)
- since the last post we have added support for IEnumberable, Lazy, more support for Threading. 
- the internal code may not be the cleanest (we are refactoring constantly now, improving existing functionality and adding new features)
- given the project a name: Bonsai

# Overview

We are coding in .NET Core (Standard), and we have started this project with a number of tests, which we have tried to use as the requirements.

With regards to tools there are a number of tools at our disposal, and we will look at the local development tools as well as the continuos integration tools.

# Testing Tools

- [Mspec](https://github.com/machine/machine.specifications) - Test Framework which has been used
- [Mspec dotnet adaptor](https://github.com/machine/machine.vstestadapter) - allows tests to be run with the dotnet cli.

# Local development

I tried 2 different set of tools during the development of this library

## VS Code - FREE option (and it works well)

![](https://raw.githubusercontent.com/dbones/dbones.github.io/master/images/posts/2019/bonsai/vsCode.png)

**Tools**

- VS Code => editor
- [Coverage Gutters](https://marketplace.visualstudio.com/items?itemName=ryanluker.vscode-coverage-gutters) => shows the lines of code which have been covered
- [Coverlet](https://github.com/tonerdo/coverlet/) - dotnet Cli plugin which runs code coverage
- [Test Explorer](https://marketplace.visualstudio.com/items?itemName=formulahendry.dotnet-test-explorer) - shows the runable tests - note this was able to run the tests, however the experience is not 100%. (at the time of writing)

Code supports .NET core really well, including some code refactoring. The best part to this setup was a small plugin which enabled code coverage to be seen against each line of code.

In this setup we take advantage of the dotnet test adaptor (which runs MSpec) and use its watch command.

```
dotnet watch --project ./src/Bonsai.Tests test /p:CollectCoverage=true /p:CoverletOutputFormat=lcov /p:CoverletOutput=./lcov.info
``` 

Now when we save a file, the tests are rerun and we get automatically updated in VS Code to what has been covered.


## Rider

![](https://raw.githubusercontent.com/dbones/dbones.github.io/master/images/posts/2019/bonsai/jetbrains.png)

**Tools**

- Rider => IDE (compile, test runner and code coverage)
- MSpec plugin => provides great integration

This experience was fully integrated, and offered the best experience with MSpec, allowing us to select a individual test as well as rerun all tests with code coverage on file saves.


Either tool I fund to be a great place to code.

# Continuous Integrations

here we have several tools which try to ensure that quality is met/retained, and all of them run silently in the background (to enable a smooth process)

- [Github](https://github.com/dbones/bonsai) - Source Control, somewhere to store the code, where others can hopefully contribute.
- [AppVeyor](https://ci.appveyor.com/project/dbones/bonsai) - Build service
- [CodeCov](https://codecov.io/gh/dbones/bonsai) - Code Coverage reports
- [Codacy](https://app.codacy.com/project/dbones/bonsai/dashboard) - Code quality reports
- [Nuget](https://www.nuget.org/packages/Bonsai.Ioc/) - Package manager

some notes on the above tools

##Testing

Code Coverage is supplied by the same library as before Coverlet, however we need to configure it to produce reports in the OpenCover format in-order for CodeCov to process.

![](https://raw.githubusercontent.com/dbones/dbones.github.io/master/images/posts/2019/bonsai/CodeCov.PNG)

In addition AppVeyor supports a set of known result formats, luckly dotnet test support a logger called trx, which can be uploaded to the mstest end point

![](https://raw.githubusercontent.com/dbones/dbones.github.io/master/images/posts/2019/bonsai/tests-on-ci.PNG)

The following shows how these 2 reports are generated and then uploaded to the correct service endpoint.

```
dotnet test --logger trx --results-directory ./results /p:CollectCoverage=true /p:CoverletOutputFormat=opencover /p:Threshold=70 ./Bonsai.Tests

cd Bonsai.Tests
Invoke-WebRequest -Uri 'https://codecov.io/bash' -OutFile codecov.sh
bash codecov.sh -f "coverage.opencover.xml"


$wc = New-Object 'System.Net.WebClient'
$results = Resolve-Path .\results\*.trx
$results = $results[0]
Write-Host $results
$wc.UploadFile("https://ci.appveyor.com/api/testresults/mstest/$($env:APPVEYOR_JOB_ID)", ($results))
```

The above took a little of digging around.


## Quality

I have focused mostly on correctness (passing tests) and then on speed (benchmarks), however I do care about quality, thus I have included CodeCov within the tools.

at this point we only receive a 'B', however the report includes a detailed output of where in the code the changes need to be made.

![](https://raw.githubusercontent.com/dbones/dbones.github.io/master/images/posts/2019/bonsai/quality.PNG)

at the moment I do not agree with a B score, I was hoping it to be more critical over the classes sizes and responsibilities. something to indicate if SRP may be broken. I will look more into this, to ensure that im not missing something.


# Wrap up

For an open source project there are so many tools we can get our hands on to help with the continuous integration of new changes.

All of them offer great insight, of which we can act on in due course.
