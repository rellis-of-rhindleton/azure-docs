---
title: Diagnose with Live Metrics - Application Insights - Azure Monitor
description: Monitor your web app in real time with custom metrics, and diagnose issues with a live feed of failures, traces, and events.
ms.topic: conceptual
ms.date: 11/15/2022
ms.reviewer: sdash
ms.devlang: csharp
---

# Live Metrics: Monitor and diagnose with 1-second latency

Monitor your live, in-production web application by using Live Metrics (also known as QuickPulse) from [Application Insights](./app-insights-overview.md). You can select and filter metrics and performance counters to watch in real time, without any disturbance to your service. You can also inspect stack traces from sample failed requests and exceptions. Together with [Profiler](./profiler.md) and [Snapshot Debugger](./snapshot-debugger.md), Live Metrics provides a powerful and noninvasive diagnostic tool for your live website.

> [!NOTE]
> Live Metrics only supports TLS 1.2. For more information, see [Troubleshooting](#troubleshooting).

With Live Metrics, you can:

* Validate a fix while it's released by watching performance and failure counts.
* Watch the effect of test loads and diagnose issues live.
* Focus on particular test sessions or filter out known issues by selecting and filtering the metrics you want to watch.
* Get exception traces as they happen.
* Experiment with filters to find the most relevant KPIs.
* Monitor any Windows performance counter live.
* Easily identify a server that's having issues and filter all the KPI/live feed to just that server.

![Screenshot that shows the Live Metrics tab.](./media/live-stream/live-metric.png)

Live Metrics is currently supported for ASP.NET, ASP.NET Core, Azure Functions, Java, and Node.js apps.

> [!NOTE]
> The number of monitored server instances displayed by Live Metrics might be lower than the actual number of instances allocated for the application. This mismatch is because many modern web servers will unload applications that don't receive requests over a period of time to conserve resources. Because Live Metrics only counts servers that are currently running the application, servers that have already unloaded the process won't be included in that total.

## Get started

> [!IMPORTANT]
> Monitoring ASP.NET Core [LTS](https://dotnet.microsoft.com/platform/support/policy/dotnet-core) applications requires Application Insights version 2.8.0 or above. To enable Application Insights, ensure that it's activated in the Azure portal and that the Application Insights NuGet package is included. Without the NuGet package, some telemetry is sent to Application Insights, but that telemetry won't show in Live Metrics.

1. Follow language-specific guidelines to enable Live Metrics:
   * [ASP.NET](./asp-net.md): Live Metrics is enabled by default.
   * [ASP.NET Core](./asp-net-core.md): Live Metrics is enabled by default.
   * [.NET/.NET Core Console/Worker](./worker-service.md): Live Metrics is enabled by default.
   * [.NET Applications: Enable using code](#enable-live-metrics-by-using-code-for-any-net-application).
   * [Java](./java-in-process-agent.md): Live Metrics is enabled by default.
   * [Node.js](./nodejs.md#live-metrics)

1. In the [Azure portal](https://portal.azure.com), open the Application Insights resource for your app. Then open Live Stream.

1. [Secure the control channel](#secure-the-control-channel) if you might use sensitive data like customer names in your filters.

[!INCLUDE [azure-monitor-log-analytics-rebrand](../../../includes/azure-monitor-instrumentation-key-deprecation.md)]

### Enable Live Metrics by using code for any .NET application

> [!NOTE]
> Live Metrics is enabled by default when you onboard it by using the recommended instructions for .NET applications.

To manually set up Live Metrics:

1. Install the NuGet package [Microsoft.ApplicationInsights.PerfCounterCollector](https://www.nuget.org/packages/Microsoft.ApplicationInsights.PerfCounterCollector).
1. The following sample console app code shows setting up Live Metrics:

    ```csharp
    using Microsoft.ApplicationInsights;
    using Microsoft.ApplicationInsights.Extensibility;
    using Microsoft.ApplicationInsights.Extensibility.PerfCounterCollector.QuickPulse;
    using System;
    using System.Threading.Tasks;
    
    namespace LiveMetricsDemo
    {
        class Program
        {
            static void Main(string[] args)
            {
                // Create a TelemetryConfiguration instance.
                TelemetryConfiguration config = TelemetryConfiguration.CreateDefault();
                config.InstrumentationKey = "INSTRUMENTATION-KEY-HERE";
                QuickPulseTelemetryProcessor quickPulseProcessor = null;
                config.DefaultTelemetrySink.TelemetryProcessorChainBuilder
                    .Use((next) =>
                    {
                        quickPulseProcessor = new QuickPulseTelemetryProcessor(next);
                        return quickPulseProcessor;
                    })
                    .Build();
    
                var quickPulseModule = new QuickPulseTelemetryModule();
    
                // Secure the control channel.
                // This is optional, but recommended.
                quickPulseModule.AuthenticationApiKey = "YOUR-API-KEY-HERE";
                quickPulseModule.Initialize(config);
                quickPulseModule.RegisterTelemetryProcessor(quickPulseProcessor);
    
                // Create a TelemetryClient instance. It is important
                // to use the same TelemetryConfiguration here as the one
                // used to set up Live Metrics.
                TelemetryClient client = new TelemetryClient(config);
    
                // This sample runs indefinitely. Replace with actual application logic.
                while (true)
                {
                    // Send dependency and request telemetry.
                    // These will be shown in Live Metrics.
                    // CPU/Memory Performance counter is also shown
                    // automatically without any additional steps.
                    client.TrackDependency("My dependency", "target", "http://sample",
                        DateTimeOffset.Now, TimeSpan.FromMilliseconds(300), true);
                    client.TrackRequest("My Request", DateTimeOffset.Now,
                        TimeSpan.FromMilliseconds(230), "200", true);
                    Task.Delay(1000).Wait();
                }
            }
        }
    }
    ```

The preceding sample is for a console app, but the same code can be used in any .NET applications. If any other telemetry modules are enabled to autocollect telemetry, it's important to ensure that the same configuration used for initializing those modules is used for the Live Metrics module.

## How does Live Metrics differ from metrics explorer and Log Analytics?

| Capabilities |Live Stream | Metrics explorer and Log Analytics |
|---|---|---|
|Latency|Data displayed within one second.|Aggregated over minutes.|
|No retention|Data persists while it's on the chart and is then discarded.|[Data retained for 90 days.](./data-retention-privacy.md#how-long-is-the-data-kept)|
|On demand|Data is only streamed while the Live Metrics pane is open. |Data is sent whenever the SDK is installed and enabled.|
|Free|There's no charge for Live Stream data.|Subject to [pricing](../logs/cost-logs.md#application-insights-billing).
|Sampling|All selected metrics and counters are transmitted. Failures and stack traces are sampled. |Events can be [sampled](./api-filtering-sampling.md).|
|Control channel|Filter control signals are sent to the SDK. We recommend you secure this channel.|Communication is one way, to the portal.|

## Select and filter your metrics

These capabilities are available with ASP.NET, ASP.NET Core, and Azure Functions (v2).

You can monitor custom KPI live by applying arbitrary filters on any Application Insights telemetry from the portal. Select the filter control that shows when you mouse-over any of the charts. The following chart plots a custom **Request** count KPI with filters on **URL** and **Duration** attributes. Validate your filters with the stream preview section that shows a live feed of telemetry that matches the criteria you've specified at any point in time.

![Screenshot that shows the Filter request rate.](./media/live-stream/filter-request.png)

You can monitor a value different from **Count**. The options depend on the type of stream, which could be any Application Insights telemetry like requests, dependencies, exceptions, traces, events, or metrics. It can also be your own [custom measurement](./api-custom-events-metrics.md#properties).

![Screenshot that shows the Query Builder on Request Rate with a custom metric.](./media/live-stream/query-builder-request.png)

Along with Application Insights telemetry, you can also monitor any Windows performance counter. Select it from the stream options and provide the name of the performance counter.

Live Metrics are aggregated at two points: locally on each server and then across all servers. You can change the default at either one by selecting other options in the respective dropdown lists.

## Sample telemetry: Custom live diagnostic events
By default, the live feed of events shows samples of failed requests and dependency calls, exceptions, events, and traces. Select the filter icon to see the applied criteria at any point in time.

![Screenshot that shows the Filter button.](./media/live-stream/filter.png)

As with metrics, you can specify any arbitrary criteria to any of the Application Insights telemetry types. In this example, we're selecting specific request failures and events.

![Screenshot that shows the Query Builder.](./media/live-stream/query-builder.png)

> [!NOTE]
> Currently, for exception message-based criteria, use the outermost exception message. In the preceding example, to filter out the benign exception with an inner exception message (follows the "<--" delimiter) "The client disconnected," use a message not-contains "Error reading request content" criteria.

To see the details of an item in the live feed, select it. You can pause the feed either by selecting **Pause** or by scrolling down and selecting an item. Live feed resumes after you scroll back to the top, or when you select the counter of items collected while it was paused.

![Screenshot that shows the Sample telemetry window with an exception selected and the exception details displayed at the bottom of the window.](./media/live-stream/sample-telemetry.png)

## Filter by server instance

If you want to monitor a particular server role instance, you can filter by server. To filter, select the server name under **Servers**.

![Screenshot that shows the Sampled live failures.](./media/live-stream/filter-by-server.png)

## Secure the control channel

Live Metrics custom filters allow you to control which of your application's telemetry is streamed to the Live Metrics view in the Azure portal. The filters criteria is sent to the apps that are instrumented with the Application Insights SDK. The filter value could potentially contain sensitive information, such as the customer ID. To keep this value secured and prevent potential disclosure to unauthorized applications, you have two options:

- **Recommended:** Secure the Live Metrics channel by using [Azure Active Directory (Azure AD) authentication](./azure-ad-authentication.md#configuring-and-enabling-azure-ad-based-authentication).
- **Legacy (no longer recommended):** Set up an authenticated channel by configuring a secret API key as explained in the "Legacy option" section.

> [!NOTE]
> On September 30, 2025, API keys used to stream Live Metrics telemetry into Application Insights will be retired. After that date, applications that use API keys won't be able to send Live Metrics data to your Application Insights resource. Authenticated telemetry ingestion for Live Metrics streaming to Application Insights will need to be done with [Azure AD authentication for Application Insights](./azure-ad-authentication.md).

It's possible to try custom filters without having to set up an authenticated channel. Select any of the filter icons and authorize the connected servers. If you choose this option, you'll have to authorize the connected servers once every new session or whenever a new server comes online.

> [!WARNING]
> We strongly discourage the use of unsecured channels and will disable this option six months after you start using it. The **Authorize connected servers** dialog displays the date after which this option will be disabled.

:::image type="content" source="media/live-stream/live-stream-auth.png" alt-text="Screenshot that shows the Authorize connected servers dialog." lightbox="media/live-stream/live-stream-auth.png":::

### Legacy option: Create an API key

1. Select the **API Access** tab and then select **Create API key**.

   ![Screenshot that shows selecting the API Access tab and the Create API key button.](./media/live-stream/api-key.png)

1. Select the **Authenticate SDK control channel** checkbox and then select **Generate key**.

   ![Screenshot that shows the Create API key pane. Select Authenticate SDK control channel checkbox and then select Generate key.](./media/live-stream/create-api-key.png)

### Add an API key to configuration

You can add an API key to configuration for ASP.NET, ASP.NET Core, WorkerService, and Azure Functions apps.

#### ASP.NET

In the *applicationinsights.config* file, add `AuthenticationApiKey` to `QuickPulseTelemetryModule`:

```xml
<Add Type="Microsoft.ApplicationInsights.Extensibility.PerfCounterCollector.QuickPulse.QuickPulseTelemetryModule, Microsoft.AI.PerfCounterCollector">
      <AuthenticationApiKey>YOUR-API-KEY-HERE</AuthenticationApiKey>
</Add>
```

#### ASP.NET Core

For [ASP.NET Core](./asp-net-core.md) applications, follow these instructions.

Modify `ConfigureServices` of your *Startup.cs* file as shown.

Add the following namespace:

```csharp
using Microsoft.ApplicationInsights.Extensibility.PerfCounterCollector.QuickPulse;
```

Then modify the `ConfigureServices` method:

```csharp
public void ConfigureServices(IServiceCollection services)
{
    // Existing code which includes services.AddApplicationInsightsTelemetry() to enable Application Insights.
    services.ConfigureTelemetryModule<QuickPulseTelemetryModule> ((module, o) => module.AuthenticationApiKey = "YOUR-API-KEY-HERE");
}
```

For more information on how to configure ASP.NET Core applications, see [Configuring telemetry modules in ASP.NET Core](./asp-net-core.md#configuring-or-removing-default-telemetrymodules).

#### WorkerService

For [WorkerService](./worker-service.md) applications, follow these instructions.

Add the following namespace:

```csharp
using Microsoft.ApplicationInsights.Extensibility.PerfCounterCollector.QuickPulse;
```

Next, add the following line before the call `services.AddApplicationInsightsTelemetryWorkerService`:

```csharp
    services.ConfigureTelemetryModule<QuickPulseTelemetryModule> ((module, o) => module.AuthenticationApiKey = "YOUR-API-KEY-HERE");
```

For more information on how to configure WorkerService applications, see [Configuring telemetry modules in WorkerServices](./worker-service.md#configure-or-remove-default-telemetry-modules).

#### Azure Functions apps

For Azure Functions apps (v2), you can secure the channel with an API key by using an environment variable.

Create an API key from within your Application Insights resource and go to **Settings** > **Configuration** for your Azure Functions app. Select **New application setting**, enter a name of `APPINSIGHTS_QUICKPULSEAUTHAPIKEY`, and enter a value that corresponds to your API key.

## Supported features table

| Language                         | Basic metrics       | Performance metrics | Custom filtering    | Sample telemetry    | CPU split by process |
|----------------------------------|:--------------------|:--------------------|:--------------------|:--------------------|:---------------------|
| .NET Framework                   | Supported ([LTS](https://dotnet.microsoft.com/platform/support/policy/dotnet-core)) | Supported ([LTS](https://dotnet.microsoft.com/platform/support/policy/dotnet-core)) | Supported ([LTS](https://dotnet.microsoft.com/platform/support/policy/dotnet-core)) | Supported ([LTS](https://dotnet.microsoft.com/platform/support/policy/dotnet-core)) | Supported ([LTS](https://dotnet.microsoft.com/platform/support/policy/dotnet-core))  |
| .NET Core (target=.NET Framework)| Supported ([LTS](https://dotnet.microsoft.com/platform/support/policy/dotnet-core)) | Supported ([LTS](https://dotnet.microsoft.com/platform/support/policy/dotnet-core)) | Supported ([LTS](https://dotnet.microsoft.com/platform/support/policy/dotnet-core)) | Supported ([LTS](https://dotnet.microsoft.com/platform/support/policy/dotnet-core)) | Supported ([LTS](https://dotnet.microsoft.com/platform/support/policy/dotnet-core))  |
| .NET Core (target=.NET Core)     | Supported ([LTS](https://dotnet.microsoft.com/platform/support/policy/dotnet-core)) | Supported*          | Supported ([LTS](https://dotnet.microsoft.com/platform/support/policy/dotnet-core)) | Supported ([LTS](https://dotnet.microsoft.com/platform/support/policy/dotnet-core)) | **Not supported**    |
| Azure Functions v2               | Supported           | Supported           | Supported           | Supported           | **Not supported**    |
| Java                             | Supported (V2.0.0+) | Supported (V2.0.0+) | **Not supported**   | Supported (V3.2.0+) | **Not supported**    |
| Node.js                          | Supported (V1.3.0+) | Supported (V1.3.0+) | **Not supported**   | Supported (V1.3.0+) | **Not supported**    |

Basic metrics include request, dependency, and exception rate. Performance metrics (performance counters) include memory and CPU. Sample telemetry shows a stream of detailed information for failed requests and dependencies, exceptions, events, and traces.

 PerfCounters support varies slightly across versions of .NET Core that don't target the .NET Framework:

- PerfCounters metrics are supported when running in Azure App Service for Windows (ASP.NET Core SDK version 2.4.1 or higher).
- PerfCounters are supported when the app is running in *any* Windows machines, like VM, Azure Cloud Service, or on-premises (ASP.NET Core SDK version 2.7.1 or higher), but only for apps that target .NET Core [LTS](https://dotnet.microsoft.com/platform/support/policy/dotnet-core) or higher.
- PerfCounters are supported when the app is running *anywhere* (such as Linux, Windows, app service for Linux, or containers) in the latest versions, but only for apps that target .NET Core [LTS](https://dotnet.microsoft.com/platform/support/policy/dotnet-core) or higher.

## Troubleshooting

Live Metrics uses different IP addresses than other Application Insights telemetry. Make sure [those IP addresses](./ip-addresses.md) are open in your firewall. Also check that [outgoing ports for Live Metrics](./ip-addresses.md#outgoing-ports) are open in the firewall of your servers.

As described in the [Azure TLS 1.2 migration announcement](https://azure.microsoft.com/updates/azuretls12/), Live Metrics now only supports TLS 1.2. If you're using an older version of TLS, Live Metrics won't display any data. For applications based on .NET Framework 4.5.1, see [Enable Transport Layer Security (TLS) 1.2 on clients - Configuration Manager](/mem/configmgr/core/plan-design/security/enable-tls-1-2-client#bkmk_net) to support the newer TLS version.

### Missing configuration for .NET

1. Verify that you're using the latest version of the NuGet package [Microsoft.ApplicationInsights.PerfCounterCollector](https://www.nuget.org/packages/Microsoft.ApplicationInsights.PerfCounterCollector).
1. Edit the `ApplicationInsights.config` file:
    * Verify that the connection string points to the Application Insights resource you're using.
    * Locate the `QuickPulseTelemetryModule` configuration option. If it isn't there, add it.
    * Locate the `QuickPulseTelemetryProcessor` configuration option. If it isn't there, add it.
         
     ```xml
    <TelemetryModules>
    <Add Type="Microsoft.ApplicationInsights.Extensibility.PerfCounterCollector.
    QuickPulse.QuickPulseTelemetryModule, Microsoft.AI.PerfCounterCollector"/>
    </TelemetryModules>
    
    <TelemetryProcessors>
    <Add Type="Microsoft.ApplicationInsights.Extensibility.PerfCounterCollector.
    QuickPulse.QuickPulseTelemetryProcessor, Microsoft.AI.PerfCounterCollector"/>
    <TelemetryProcessors>
    ````
1. Restart the application.

## Next steps

* [Monitor usage with Application Insights](./usage-overview.md)
* [Use Diagnostic Search](./diagnostic-search.md)
* [Profiler](./profiler.md)
* [Snapshot Debugger](./snapshot-debugger.md)
