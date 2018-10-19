---
layout: page
title: TestFlask Assistant
permalink: /test-flask-assistant/
order: 5
---

TestFlask Assistant is a nuget package that provides an ASP.NET MVC Area for your ASP.NET MVC apps that consume your service weaved with TestFlask. Using the mini view that is embedded to the bottom right of any page, you can create a scenario, start recording or overwrite existing steps inside a scenario. 

Here is an example of TestFlask Assistant running inside a web app page.

![TestFlask Assistant Sample](https://github.com/FatihSahin/test-flask-sample/raw/master/Docs/assistant_1.png)

### How to install it inside an ASP.NET Web App

[TestFlask.Assistant.Mvc](https://www.nuget.org/packages/TestFlask.Assistant.Mvc/) package must be installed into your web application project. 

You can install the package using nuget package manager inside Visual Studio or from the package manager console. 

> Install-Package TestFlask.Assistant.Mvc -Version 1.0.0

After installation, following changes will occur

A TestFlask assistant section will be added to your web.config

```xml
  <testFlaskAssistant enabled="true">
    <api url="http://localhost:12079" />
    <manager url="http://localhost:4200" />
    <project key="QuickPayDemo" />
  </testFlaskAssistant>
```
You need to configure your running TestFlask API Host url along with the TestFlask Manager url. You also need to set the TestFlask project key to create scenarios and record steps inside that project.

TestFlask.Assistant package will also append an MVC Area to your web projects.

![TestFlask Area](/public/images/testFlaskAssistant_area.PNG)

Inside the ```_Layout.cshtml```, you should add ```testFlask.css``` into html head tag.

```html
<head>
    ...
    <link rel="stylesheet" type="text/css" href="@Url.Content("~/Content/testFlask.css")">
</head>
```

And lastly you need to append testFlask area view at the bottom of the body.

```html
<body>
    <!-- content here -->
    @{ Html.RenderPartial("~/Areas/TestFlask/Views/TestFlaskAssistant/Index.cshtml", TestFlask.Assistant.Mvc.Models.AssistantSessionContext.Current); }
    <!-- maybe some script registrations here -->
</body>
```

There you go, try to run your web app and you can now record your scenarios. You can even record multiple steps in different pages on to a single scenario.

> TestFlask stores scenario context inside the ASP.NET Session between page loads.