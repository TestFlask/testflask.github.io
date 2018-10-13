---
layout: page
title: TestFlask Weaver
permalink: /test-flask/
order: 1
---

TestFlask Weaver is made up of two nuget packages.

* [TestFlaskAddin.Fody](https://www.nuget.org/packages/TestFlaskAddin.Fody/)

    This is a [Fody](https://github.com/Fody/Fody) addin for TestFlask to plug in to MS Build. This package needs to be referenced in order to enable TestFlask to be able to weave your methods.

    When you install this package, a FodyWeaver.xml will be included in your project. You can enable by uncommenting or disable TestFlask by commenting TestFlaskAddin line using this xml file.

    ```xml
    <?xml version="1.0" encoding="utf-8"?>
    <Weavers>
        <TestFlaskAddin />
    </Weavers>
    ```

* [TestFlask.Aspects](https://www.nuget.org/packages/TestFlask.Aspects/)

    This package contains core TestFlask objects including the ```[Playback]``` attribute that is used to mark your methods that are to be weaved.

    You mark your methods as below.

    ```csharp
    [Playback]
    public Customer GetCustomer(int customerId)
    {
        using (var customerClient = new CustomerServiceClient())
        {
            return customerClient.GetCustomer(customerId);
        }
    }
    ```

    or using a ```RequestIdentifier```.

    ```csharp
    [Playback(typeof(AccountRequestIdentifier))]
    public Account GetAccount(string accountNumber)
    {
        using (var accountClient = new ExternalAccountServices.AccountServiceClient())
        {
            return accountClient.GetAccount(accountNumber);
        }
    }
    ```
    A ```RequestIdentifier``` is class that implements ```IRequestIdentifier``` to help TestFlask which recorded response to return for a given set of parameters of a method that is to be replayed.

    An example implementation of ```AccountRequestIdentifier``` is like this

    ```csharp
    public class AccountRequestIdentifier : IRequestIdentifier<string>
    {
        public string ResolveDisplayInfo(string accountNumber)
        {
            return $"Account ID => {accountNumber}";
        }

        public string ResolveIdentifierKey(string accountNumber)
        {
            return accountNumber; 
        }
    }
    ```

    ```ResolveIdentifierKey``` method is the important one in this class that returns a unique string identifier value that corresponds to a set of parameter values. A hash value created out of parameter values is generally suitable for multi param or object type param methods. In this example however, ```accountNumber``` is the only parameter for ```GetAccount``` method and is already a unique value that distinguishes every ```GetAccount``` request from each other.

    When TestFlask weaves your methods, it does not touch or alter your source code. It only alters your output assembly (exe or dll) in build time so that you can enable or disable TestFlask without altering your code. 

    Here, C# code for ```GetAccount``` is decompiled from weaved IL code to demonstrate what TestFlask does with these marked methods behind the scenes.

    ```csharp
    public Account GetAccount(string accountNumber)
    {
      FuncPlayer<string, Account> funcPlayer = new FuncPlayer<string, Account>("Demo.QuickPay.Biz.ExternalAccountServices.Account Demo.QuickPay.Biz.Integrators.AccountIntegrator::GetAccount(System.String)", (IRequestIdentifier<string>) new AccountRequestIdentifier(), (IResponseIdentifier<Account>) null);
      funcPlayer.BeginInvocation(accountNumber);
      switch (funcPlayer.DetermineTestMode(accountNumber))
      {
        case TestModes.NoMock:
          return funcPlayer.CallOriginal(accountNumber, new Func<string, Account>(this.GetAccount__Original));
        case TestModes.Record:
          return funcPlayer.Record(accountNumber, new Func<string, Account>(this.GetAccount__Original));
        case TestModes.Play:
          return funcPlayer.Play(accountNumber);
        default:
          throw new Exception("Invalid TestFlask test mode detected!");
      }
    }

    public Account GetAccount__Original(string accountNumber)
    {
      using (AccountServiceClient accountServiceClient = new AccountServiceClient())
        return accountServiceClient.GetAccount(accountNumber);
    }
    ```

    This method may seem complex, however this should not be a concern as code is auto generated. 
    What it does is more important. First thing to notice is that the original method is renamed as ```GetAccount__Original``` and the original method ```GetAccount``` has now some additional capabilities.

    It reacts to three main ```TestModes```.

    * ```TestModes.NoMock```

        This mode is actually calling the original method inside a wrapper and your method behaves the same as the unweaved original version. It is useful when you want to ignore or bypassTestFlask in your weaved method.

    * ```TestModes.Record```

        In this mode, TestFlask again invokes the original method. However this time, it records all the input parameters and output values of the method inside a ```TestFlaskContext```. When all the weaved methods finish working and the root method returns a output, TestFlask persists this intercepted ```TestFlaskContext``` through HTTP/Rest via TestFlask.API and store the context inside a scenario.

    * ```TestModes.Play```

        If you have a previously recorded scenario, then during playback, ```Play``` mode is used and a proper recorded response from a pre-loaded ```TestFlaskContext``` is returned from the method instead of actually calling it. If you have a ```RequestIdentifier``` for the current method, TestFlask uses the identifier key to match and then return the proper output for that method invocation entry inside the context.

TestFlask does not intercept or replay your mutated parameter objects or values that are expected to be modified inside the method. It is a subtle but a very important behavior. TestFlask is almost always suitable for mocking your pure functions or methods which generally does work of encoding/decoding data over IO. Please take notice that TestFlask is protocol independent (*tcp, http or udp. It really does not matter for TestFlask*). Such examples are RPC or REST service calls, database query calls and network operations that serializes some object or args over the wire and deserializes a response object back. 

Parameter mutation is not something TestFlask considers to replay or provide on replay, because it makes everything utterly complex. Thus, be careful to use TestFlask on your pure functions with immutable parameter behavior. Consider using TestFlask to mock your integration entrance methods for external code/systems. TestFlask is not suitable for weaving the methods that contains your core business logic or CPU bound complex instructions. It is not meant to weave/mock those type of functionality.*

### How does TestFlask understand TestMode?

TestFlask uses custom TestFlask Http Headers to determine what to do with an incoming request.

Http Header         | Description
--------------------| -------------------------------------------------------
TestFlask-Mode      | See below
TestFlask-ProjectKey| This is the key that you created for your backend service to categorize scenarios
TestFlask-ScenarioNo| This is the number for your test scenario. It is the main test data encapsulation identifier.
TestFlask-StepNo    | This is optional, if you provide a step no, it will override that step. If you do not, TestFlask will create an auto step under that scenario.

```TestFlask-Mode``` header can take 5 actual values but the most important ones are the first three.

TestFlask-Mode      | Description
--------------------| -------------------------------------------------------
Record              | Calls the original method and then persists request and response objects through TestFlask.API into a mongoDB database.
Play                | Calls TestFlask.API to look for a recorded response for the current request and returns that response.
NoMock              | Calls the original method with no mocking.
Assert              | Same as Play, however in this case TestFlask stores last response as an assertion result to assert later on.
IntelliRecord       | Just like record, however in this case if a matching invocation already exists it just replays it, but patches missing ones on to existing invocation tree.

Here are some illustrations of the way TestFlask works.

* In the first one, there is no TestFlask involved. Web clients call the TransferMoney endpoint on the service and sequentially it invokes GetAccount operations which fetches the data from an external service, returns the response to TransferMoney. Finally, TransferMoney does its own operations and returns the final response to the client.

    ![No TestFlask](/public/images/testflaskNoMock.png "No TestFlask")

* The second example is the Record mode. Here everything works exactly like the first time, however TestFlask intercepts these calls and puts them in TestFlask context. Just before the root invocation (*which is ```TransferMoney``` in this example*) returns its response to the web clients, TestFlask records the step through TestFlask API which stores this data inside the database.

    ![TestFlask Record](/public/images/testflaskRecord.png "TestFlask Record")

* Third one is the Play mode. This is where TestFlask intervenes with the normal flow. It first intercepts incoming call to the root invocation (*via ```HttpModule``` registered in the web config*), and immediately loads the given scenario and step data for the given TestFlask-StepNo in the custom Http headers using TestFlask.API. Every call from now on will be matched from TestFlask context that is loaded and no actual or external calls will be made to these matching methods.
![TestFlask Play](/public/images/testflaskPlay.png "TestFlask Play")