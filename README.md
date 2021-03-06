# Azure Bus Depot

## Motivation

A compact, opinioned but extensible .Net Standard SDK for Azure Service Bus which aims to tackle the cross-cutting concerns of:
- **Deserialisation**: Extensible [IMessageSerialiser](./Src/AzureBusDepot/IMessageSerialiser.cs) to derive an instance of your *TMessage* from a Message with built-in JSON deserialisation using [JsonMessageSerialiser](./Src/AzureBusDepot/JsonMessageSerialiser.cs)
- **Handler dispatching**: Extensible [IMessageProcessorDispatcher](./Src/AzureBusDepot/IMessageProcessorDispatcher.cs) for dispatching your [IMessageHandler](./Src/AzureBusDepot/IMessageHandler.cs) implementation with built-in property based dispatching using [MessagePropertyBasedDispatcher](./Src/AzureBusDepot/MessagePropertyBasedDispatcher.cs)
- **Logging**: Uses [ILogger](https://docs.microsoft.com/en-us/dotnet/api/microsoft.extensions.logging.ilogger?view=aspnetcore-2.1) methods from [Microsoft.Extensions.Logging](https://docs.microsoft.com/en-us/dotnet/api/microsoft.extensions.logging?view=aspnetcore-2.1) with built-in Application Insights tracing
- **Telemetry**: Extensible [IInstrumentor](./Src/AzureBusDepot/IInstrumentor.cs) to measure elapsed handling with built-in Application Insights metric tracking
- **Exception handling**: Handles common problems with message handling by abandoning or dead-lettering messages with enriched properties explaining the outcome

## Getting Started
1. Install the package `AzureBusDepot` from nuget
2. Define implementations of [IMessageHandler&lt;TMessage&gt;](./Src/AzureBusDepot/IMessageHandler.cs) for your message types (see [sample](./Samples/NetCoreConsoleApp/SingleMessageType/MyEventHandler.cs))
3. Define implementations of [IEndpointHandlingConfig](./Samples/NetCoreConsoleApp/SingleMessageType/SingleMessageTypeEndpointHandlingConfig.cs) for every Azure Service Bus endpoint you want to to listen to. As a consumer it is a bit of work but these unique config types are required to play nice with dependency injection and reduces a whole lot of boilerplate bootstrapping code.

### Single Message Type expected from a Queue / Topic Subscription 
Add the following to `ConfigureServices((hostContext, services) =>{...})` of a [Generic HostBuilder](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/host/generic-host?view=aspnetcore-2.1). 
```c#
services
    .AddAzureBusDepot<JsonMessageSerialiser>()
    .ConfigureSingleMessageTypeListener<MyEndpointHandlingConfig, MyEvent, MyEventHandler>(endpointConfig)
    .AddApplicationInsights(appInsightsOptions);
```

### Multiple Message Types expected from a Queue / Topic Subscription 
Add the following to `ConfigureServices((hostContext, services) =>{...})` of a [Generic HostBuilder](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/host/generic-host?view=aspnetcore-2.1). 
```c#
services
    .AddAzureBusDepot<JsonMessageSerialiser>()
    .ConfigureMessageListenerWithPropertyBasedDispatcher(
        endpointConfig, "NServiceBus.EnclosedMessageType")
    .AddMessageHandler<MyEvent, MyEventHandler>()
    .AddMessageHandler<MyOtherEvent, MyOtherEventHandler>()
    .AddApplicationInsights(appInsightsOptions);
```
## Samples
Samples can be found under the [samples](./Samples) folder. 

## Contributing
Please see the [contributing](./CONTRIBUTING.md) document. 
