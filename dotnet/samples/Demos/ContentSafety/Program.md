# Explanation

This C# program sets up a minimal ASP.NET Core web application that integrates Microsoft's Semantic Kernel and Azure Content Safety services. The application:

- Loads configuration from multiple sources, including JSON files and user secrets.
- Configures OpenAI and Azure Content Safety options via custom extension methods and loads them into the service container.
- Registers ASP.NET Core services such as controllers, logging, and exception handling.
- Adds the Semantic Kernel for AI-powered functionalities and registers OpenAI chat completion.
- Incorporates prompt content safety filters to help moderate and detect attacks in prompts sent to AI models.
- Sets up the Azure AI Content Safety client using endpoint and key from the configuration for content moderation.
- Registers a custom `PromptShieldService` to provide additional content safety protections.
- Configures a custom exception handler for content safety-related exceptions.
- Sets up the HTTP request pipeline and maps controllers.

This boilerplate is useful for building secure AI-powered APIs that require input and output content moderation, leveraging both Semantic Kernel's extensibility and Azure's content safety capabilities.

```csharp
// Copyright (c) Microsoft. All rights reserved.

using Azure;
using Azure.AI.ContentSafety;
using ContentSafety.Extensions;
using ContentSafety.Filters;
using ContentSafety.Handlers;
using ContentSafety.Options;
using ContentSafety.Services.PromptShield;
using Microsoft.SemanticKernel;

var builder = WebApplication.CreateBuilder(args);

// Get configuration
var config = new ConfigurationBuilder()
    .SetBasePath(Directory.GetCurrentDirectory())
    .AddJsonFile("appsettings.json")
    .AddJsonFile("appsettings.Development.json", true)
    .AddUserSecrets<Program>()
    .Build();

var openAIOptions = config.GetValid<OpenAIOptions>(OpenAIOptions.SectionName);
var azureContentSafetyOptions = config.GetValid<AzureContentSafetyOptions>(AzureContentSafetyOptions.SectionName);

// Add services to the container.
builder.Services.AddControllers();
builder.Services.AddLogging(loggingBuilder => loggingBuilder.AddConsole());

// Add Semantic Kernel
builder.Services.AddKernel();
builder.Services.AddOpenAIChatCompletion(openAIOptions.ChatModelId, openAIOptions.ApiKey);

// Add Semantic Kernel prompt content safety filters
builder.Services.AddSingleton<IPromptRenderFilter, TextModerationFilter>();
builder.Services.AddSingleton<IPromptRenderFilter, AttackDetectionFilter>();

// Add Azure AI Content Safety services
builder.Services.AddSingleton<ContentSafetyClient>(_ =>
{
    return new ContentSafetyClient(
        new Uri(azureContentSafetyOptions.Endpoint),
        new AzureKeyCredential(azureContentSafetyOptions.ApiKey));
});

builder.Services.AddSingleton<PromptShieldService>(serviceProvider =>
{
    return new PromptShieldService(
        serviceProvider.GetRequiredService<ContentSafetyClient>(),
        azureContentSafetyOptions);
});

// Add exception handlers
builder.Services.AddExceptionHandler<ContentSafetyExceptionHandler>();
builder.Services.AddProblemDetails();

var app = builder.Build();

app.UseHttpsRedirection();
app.UseAuthorization();
app.UseExceptionHandler();

app.MapControllers();

app.Run();
```