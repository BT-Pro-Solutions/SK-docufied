# Explanation

This C# program demonstrates integrating Semantic Kernel—a Microsoft library for AI orchestration—with .NET's dependency injection framework using the Generic Host. The application is structured for home automation scenarios, allowing various plugins (e.g., for time, alarms, and lights) to be injected and used by Semantic Kernel-based AI agents.

The application's configuration is loaded from multiple sources, including JSON files, environment variables, user secrets (in development), and command-line arguments. It sets up services for OpenAI chat completion (or alternatively Azure OpenAI), registers custom plugins as singletons (or keyed singletons), and constructs a home automation AI kernel with these plugins. The main workflow is contained in a hosted Worker service (not shown here).

Key points:
- Uses configuration binding and validation to load API keys and settings.
- Registers plugin classes so they can be injected into the AI kernel.
- Demonstrates how to incorporate OpenAI and OpenAPI plugins (commented out).
- The DI (Dependency Injection) container sets up all dependencies required by the home automation kernel.

```csharp
/*
 Copyright (c) Microsoft. All rights reserved.

 Example that demonstrates how to use Semantic Kernel in conjunction with dependency injection.

 Loads app configuration from:
 - appsettings.json.
 - appsettings.{Environment}.json.
 - Secret Manager when the app runs in the "Development" environment (set through the DOTNET_ENVIRONMENT variable).
 - Environment variables.
 - Command-line arguments.
*/

using HomeAutomation.Options;
using HomeAutomation.Plugins;
using Microsoft.Extensions.Configuration;
using Microsoft.Extensions.DependencyInjection;
using Microsoft.Extensions.Hosting;
using Microsoft.Extensions.Options;
using Microsoft.SemanticKernel;
using Microsoft.SemanticKernel.ChatCompletion;
// For Azure OpenAI configuration
#pragma warning disable IDE0005 // Using directive is unnecessary.
using Microsoft.SemanticKernel.Connectors.AzureOpenAI;
using Microsoft.SemanticKernel.Connectors.OpenAI;

namespace HomeAutomation;

internal static class Program
{
    internal static async Task Main(string[] args)
    {
        HostApplicationBuilder builder = Host.CreateApplicationBuilder(args);
        builder.Configuration.AddUserSecrets<Worker>();

        // Actual code to execute is found in Worker class
        builder.Services.AddHostedService<Worker>();

        // Get configuration
        builder.Services.AddOptions<OpenAIOptions>()
                        .Bind(builder.Configuration.GetSection(OpenAIOptions.SectionName))
                        .ValidateDataAnnotations()
                        .ValidateOnStart();

        /* Alternatively, you can use plain, Azure OpenAI after loading AzureOpenAIOptions instead  of OpenAI
        
        builder.Services.AddOptions<AzureOpenAIOptions>()
                        .Bind(builder.Configuration.GetSection(AzureOpenAIOptions.SectionName))
                        .ValidateDataAnnotations()
                        .ValidateOnStart();
        */

        // Chat completion service that kernels will use
        builder.Services.AddSingleton<IChatCompletionService>(sp =>
        {
            OpenAIOptions openAIOptions = sp.GetRequiredService<IOptions<OpenAIOptions>>().Value;

            // A custom HttpClient can be provided to this constructor
            return new OpenAIChatCompletionService(openAIOptions.ChatModelId, openAIOptions.ApiKey);

            /* Alternatively, you can use plain, Azure OpenAI after loading AzureOpenAIOptions instead
               of OpenAI options with builder.Services.AddOptions:
            
            AzureOpenAIOptions azureOpenAIOptions  = sp.GetRequiredService<IOptions<AzureOpenAIOptions>>().Value;
            return new AzureOpenAIChatCompletionService(azureOpenAIOptions.ChatDeploymentName, azureOpenAIOptions.Endpoint, azureOpenAIOptions.ApiKey);

            */
        });

        // Add plugins that can be used by kernels
        // The plugins are added as singletons so that they can be used by multiple kernels
        builder.Services.AddSingleton<MyTimePlugin>();
        builder.Services.AddSingleton<MyAlarmPlugin>();
        builder.Services.AddKeyedSingleton<MyLightPlugin>("OfficeLight");
        builder.Services.AddKeyedSingleton<MyLightPlugin>("PorchLight", (sp, key) =>
        {
            return new MyLightPlugin(turnedOn: true);
        });

        /* To add an OpenAI or OpenAPI plugin, you need to be using Microsoft.SemanticKernel.Plugins.OpenApi.
           Then create a temporary kernel, use it to load the plugin and add it as keyed singleton.
        Kernel kernel = new();
        KernelPlugin openAIPlugin = await kernel.ImportPluginFromOpenAIAsync("<plugin name>", new Uri("<OpenAI-plugin>"));
        builder.Services.AddKeyedSingleton<KernelPlugin>("MyImportedOpenAIPlugin", openAIPlugin);

        KernelPlugin openApiPlugin = await kernel.ImportPluginFromOpenApiAsync("<plugin name>", new Uri("<OpenAPI-plugin>"));
        builder.Services.AddKeyedSingleton<KernelPlugin>("MyImportedOpenApiPlugin", openApiPlugin);*/

        // Add a home automation kernel to the dependency injection container
        builder.Services.AddKeyedTransient<Kernel>("HomeAutomationKernel", (sp, key) =>
        {
            // Create a collection of plugins that the kernel will use
            KernelPluginCollection pluginCollection = [];
            pluginCollection.AddFromObject(sp.GetRequiredService<MyTimePlugin>());
            pluginCollection.AddFromObject(sp.GetRequiredService<MyAlarmPlugin>());
            pluginCollection.AddFromObject(sp.GetRequiredKeyedService<MyLightPlugin>("OfficeLight"), "OfficeLight");
            pluginCollection.AddFromObject(sp.GetRequiredKeyedService<MyLightPlugin>("PorchLight"), "PorchLight");

            // When created by the dependency injection container, Semantic Kernel logging is included by default
            return new Kernel(sp, pluginCollection);
        });

        using IHost host = builder.Build();

        await host.RunAsync();
    }
}
```