---
# Feel free to add content and custom Front Matter to this file.
# To modify the layout, see https://jekyllrb.com/docs/themes/#overriding-theme-defaults

layout: home
title: Home
---
# TestFlask Introduction
TestFlask is an open source set of components that helps creating and replaying test scenarios for your .net backend services. It can be easily applied to any Asp.Net MVC REST API or WCF Services.
TestFlask nuget packages are available on [nuget.org](https://www.nuget.org/packages?q=TestFlask)

Source code is also available on [GitHub](https://github.com/FatihSahin/test-flask)

Here is an introduction video for a quickstart to TestFlask. 

<iframe width="560" height="315" src="https://www.youtube.com/embed/zLx35V4BPQg?rel=0" frameborder="0" allow="autoplay; encrypted-media" allowfullscreen></iframe>

> Demo code for the video is available on [GitHub](https://github.com/FatihSahin/test-flask-demo)

> There is no .Net Core support yet. However, it is on the roadmap and a high priority issue.

## TestFlask Components

TestFlask consists of following components. You can go to detail pages of each for further details.

> TestFlask Weaver, API, CLI and Assistant is available on [TestFlask GitHub Repo](https://github.com/FatihSahin/test-flask)

> TestFlask Manager is also available on [TestFlask Manager GitHub Repo](https://github.com/FatihSahin/test-flask-manager)


* ### [TestFlask Weaver](/test-flask)

    TestFlask Weaver transforms your marked methods with record, replay capabilities during MS Build. It is the provider package for core functionality of TestFlask.

* ### [TestFlask API](/test-flask-api)

    TestFlask API is a standalone REST API that your weaved services talk to while recording and replaying scenarios. It provides endpoints for retrieving, creating or updating scenario and step data. It persists your test data into a mongo db. 

* ### [TestFlask Manager](/test-flask-manager)

    TestFlask Manager is as SPA web application written in Angular4 to manage your TestFlask projects and scenarios that are recorded. It uses TestFlask API at the backend. You can clone, edit, delete or label your scenarios using the manager and do some additional operations as well. 

* ###  [TestFlask CLI](/test-flask-cli)

    TestFlask CLI is created to provide additional complementary features for your TestFlask scenarios. Currently it is used to generate unit test files out of your scenarios.

* ### [TestFlask Assistant](/test-flask-assistant)

    TestFlask Assistant is a nuget package that contains a ASP.NET MVC Area style views and some raw javascript. It is used as a plugin to your MVC applications that are clients of your weaved services. Using the assistant you can record scenarios on the fly while actually using your web app.

## Developer Notes

TestFlask is open to contributions. It is currently a side project for me, thus our team at my company uses TestFlask to do some regression testing for a large payment hub project. Therefore, I can say that it is already being used in a real environment. I wish to do additions as long as I have spare time. There are many pieces waiting to be enhanced. Check out issues on github repo for details. 
Moreover, the main missing pieces are currently .net core support and async-Task support. These two are the main parts that I'd like to add in the nearest feature. Any contributions are welcome with PRs.