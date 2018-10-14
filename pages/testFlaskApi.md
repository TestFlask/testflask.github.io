---
layout: page
title: TestFlask API
permalink: /test-flask-api/
order: 2
---

TestFlask.API is a REST API that serves TestFlask ready services behind the scenes to create, update, load scenario and step data. It is powered by a NoSQL (*mongoDB*) database. 

Currently there is no cloud version of TestFlask.API, so you need to host your own API for your services. Source code is available on [TestFlask](https://github.com/FatihSahin/test-flask) github project. Just clone the repo, open up TestFlask solution and inside there is the TestFlask.API project. Configure it with your own mongodb instance, a desired port and you are ready to host.

Configuration for mongoDB is as follows
```xml 
  <appSettings>
    <add key="testFlaskMongoDbServer" value="mongodb://localhost" />
    <add key="testFlaskMongoDbName" value="test" />
  </appSettings>
```

> There is no security built-in to TestFlask.API at the moment, however if requested or heavily needed it is a considerable feature.

Not only the TestFlask ready services talk to the API, but also [TestFlask Manager](/test-flask-manager) connects to the API to manage projects and scenarios. TestFlask API is designed in a way to work as a multi-tenant way so that it may host many unrelated projects if you want to use it inside the company for different teams.