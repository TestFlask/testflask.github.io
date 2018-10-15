---
layout: page
title: TestFlask Manager
permalink: /test-flask-manager/
order: 3
---

TestFlask Manager is a SPA Web Application that is used to manage your TestFlask scenarios.
Source code is available on [GitHub](https://github.com/FatihSahin/test-flask-web). It is written in Angular4. Feel free to contribute to it.

Here are some views that are most important.

* This is the scenario list view for QuickPayDemo TestFlask project

    ![TestFlask Scenario List](/public/images/testFlaskManager_scenarioListView.PNG)

* This is scenario details view for MoneyTransferSuccess scenario

    ![TestFlask Scenario View](/public/images/testFlaskManager_scenarioView.PNG)

    You can rename a scenario or give different labels to categorize your scenarios. Labels are also useful when auto generating unit tests with TestFlask.CLI to filter for which scenarios you want to create unit tests. You need to save the scenario from the Save button at the top of the page. It only saves scenario header modifications. If you need to update step details, use step detail part for the selected step below the scenario details and save step modifications using Save button for the step at the bottom of the page.

* Here is the step details view for the selected step inside scenario details view 

    ![TestFlask Step View](/public/images/testFlaskManager_stepView.PNG)

    Inside the step details, on the left there is the invocation tree that is recorded and on the right invocation details for the selected invocation on the tree.

    You can edit request and response objects here to manipulate the test scenario. You can also toggle Replay checkbox here. It tells TestFlask to use the recorded invocation response or not. TestFlask will call the original method if Replay is disabled on the invocation. Therefore, it is possible for TestFlask to mock your method call or totally ignore the mocking and behave as the original. This behavior is very interesting because it provides a lot of flexibility when testing a scenario. For some invocations you can use mock response, but for some you may really call real implementation. So a test can be run with partial unit test behavior and partial integration test behaviour. That is something I call a *hybrid test*. It is actually very useful while developing a certain feature while mocking unrelated integration points to bypass or mock them.

    You need to save the step after modifying any invocation details.

* You can also update or view Raw Http request for the root invocation of the step using the Raw Request button at the bottom. After modification, you again need to save the step.

    Here is the raw request viewer/editor

    ![TestFlask Request View](/public/images/testFlaskManager_requestView.PNG)

### Match Strategy

TestFlask uses a match strategy for matching a response for a given request. There are 6 different match strategies

Strategy        | Description
----------------|----------------
Inherit         | This tells TestFlask to inherit strategy from one level up. It goes up like this Step level => Scenario level => Project level. So if you want to use same strategy inside the entire project, you just need to set match strategy on that level. As Inherit is the default strategy for all scenarios and steps, they will inherti from project
Signature       | When selected, TestFlask returns first found response in the invocation tree for all the methods with same signature even if the input parameters (request object) is different.
Request         | TestFlask uses method signature plus the request identifier key resolved from an ```IRequestIdentifer``` during recording. This is the most commonly used and recommended strategy as it generally covers most of the use cases.
Depth           | In addition to Request (signature+requestIdentifierKey), TestFlask can differentiate different invocation with different depths inside the invocation tree.
Sibling         | In addition to Depth (signature+requestIdentifierKey+depth), TestFlask returns differentiate different siblings from a different parent invocatio even if the siblings have the same depth.
Exact           | TestFlask tries to match exact position of the (same depth, same parent, same invocation index if there are same siblings) invocation to return a matchnig response. As test scenarios and business rules always change, a recorded invocation tree can be very different from an invocation runtime tree in replay, this strategy is very fragile and can easily break.

### Configuring and Hosting a TestFlask Manager Instance

*   TestFlask Manager is just a web client for TestFlask.API. Therefore, you just need to properly set a TestFlask API host url.

*   You can configure TestFlask API host url inside appSettings/appSettings.ts file
    ```typescript
    export class AppSettings {
        public static API_ENDPOINT='http://localhost:12079/api/';
    }
    ```