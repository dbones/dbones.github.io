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