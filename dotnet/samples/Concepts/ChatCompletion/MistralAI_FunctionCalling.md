# Explanation

This C# file demonstrates how to use function calling with MistralAI using the Microsoft Semantic Kernel SDK. It provides several test methods showing different scenarios for invoking semantic functions with the MistralAI chat completion connector, including:

- **Auto-invoking kernel functions:** The model can automatically call C# functions (plugins) defined in the kernel, such as a sample WeatherPlugin.
- **Handling multiple function calls in the chat:** The model handles conversational context and function calling across multiple messages and turns.
- **Specifying required or no kernel functions:** Tests show enforcing required functions or excluding function calling altogether.
- **Using multiple plugins:** Demonstrates combining several plugins and handling structure types (with enums) for function inputs.
- **Plugin definitions:** Shows how to expose .NET methods as available functions to the AI using `KernelFunction` attributes and `Description` annotations.

The test infrastructure relies on Semantic Kernel's plugin pattern and execution settings to control tool call (function call) behavior. Helper classes and enums are included to illustrate how complex function arguments can be passed (e.g., color enums for widgets). The file also includes a helper method for setting up the kernel with the appropriate plugins and chat completion service.

---

```csharp
// Copyright (c) Microsoft. All rights reserved.

using System.ComponentModel;
using System.Text.Json.Serialization;
using Microsoft.OpenApi.Extensions;
using Microsoft.SemanticKernel;
using Microsoft.SemanticKernel.ChatCompletion;
using Microsoft.SemanticKernel.Connectors.MistralAI;

namespace ChatCompletion;

/// <summary>
/// Demonstrates the use of function calling with MistralAI.
/// </summary>
public sealed class MistralAI_FunctionCalling(ITestOutputHelper output) : BaseTest(output)
{
    [Fact]
    public async Task AutoInvokeKernelFunctionsAsync()
    {
        // Create a kernel with MistralAI chat completion and WeatherPlugin
        Kernel kernel = this.CreateKernelWithWeatherPlugin();

        // Invoke chat prompt with auto invocation of functions enabled
        const string ChatPrompt = """
            <message role="user">What is the weather like in Paris?</message>
        """;
        var executionSettings = new MistralAIPromptExecutionSettings { ToolCallBehavior = MistralAIToolCallBehavior.AutoInvokeKernelFunctions };
        var chatSemanticFunction = kernel.CreateFunctionFromPrompt(
            ChatPrompt, executionSettings);
        var chatPromptResult = await kernel.InvokeAsync(chatSemanticFunction);

        Console.WriteLine(chatPromptResult);
    }

    [Fact]
    public async Task AutoInvokeKernelFunctionsMultipleCallsAsync()
    {
        // Create a kernel with MistralAI chat completion and WeatherPlugin
        Kernel kernel = this.CreateKernelWithWeatherPlugin();
        var service = kernel.GetRequiredService<IChatCompletionService>();

        // Invoke chat prompt with auto invocation of functions enabled
        var chatHistory = new ChatHistory
        {
            new ChatMessageContent(AuthorRole.User, "What is the weather like in Paris?")
        };
        var executionSettings = new MistralAIPromptExecutionSettings { ToolCallBehavior = MistralAIToolCallBehavior.AutoInvokeKernelFunctions };
        var chatPromptResult1 = await service.GetChatMessageContentsAsync(chatHistory, executionSettings, kernel);
        chatHistory.AddRange(chatPromptResult1);

        chatHistory.Add(new ChatMessageContent(AuthorRole.User, "What is the weather like in Marseille?"));
        var chatPromptResult2 = await service.GetChatMessageContentsAsync(chatHistory, executionSettings, kernel);

        Console.WriteLine(chatPromptResult1[0].Content);
        Console.WriteLine(chatPromptResult2[0].Content);
    }

    [Fact]
    public async Task RequiredKernelFunctionsAsync()
    {
        // Create a kernel with MistralAI chat completion and WeatherPlugin
        Kernel kernel = this.CreateKernelWithWeatherPlugin();
        var plugin = kernel.Plugins.First();

        // Invoke chat prompt with auto invocation of functions enabled
        const string ChatPrompt = """
            <message role="user">What is the weather like in Paris?</message>
        """;
        var executionSettings = new MistralAIPromptExecutionSettings
        {
            ToolCallBehavior = MistralAIToolCallBehavior.RequiredFunctions(plugin, true)
        };
        var chatSemanticFunction = kernel.CreateFunctionFromPrompt(
            ChatPrompt, executionSettings);
        var chatPromptResult = await kernel.InvokeAsync(chatSemanticFunction);

        Console.WriteLine(chatPromptResult);
    }

    [Fact]
    public async Task NoKernelFunctionsAsync()
    {
        // Create a kernel with MistralAI chat completion and WeatherPlugin
        Kernel kernel = this.CreateKernelWithWeatherPlugin();

        // Invoke chat prompt with auto invocation of functions enabled
        const string ChatPrompt = """
            <message role="user">What is the weather like in Paris?</message>
        """;
        var executionSettings = new MistralAIPromptExecutionSettings
        {
            ToolCallBehavior = MistralAIToolCallBehavior.NoKernelFunctions
        };
        var chatSemanticFunction = kernel.CreateFunctionFromPrompt(
            ChatPrompt, executionSettings);
        var chatPromptResult = await kernel.InvokeAsync(chatSemanticFunction);

        Console.WriteLine(chatPromptResult);
    }

    [Fact]
    public async Task AutoInvokeKernelFunctionsMultiplePluginsAsync()
    {
        // Create a kernel with MistralAI chat completion and WeatherPlugin and WidgetPlugin
        Kernel kernel = this.CreateKernelWithWeatherPlugin();
        kernel.Plugins.AddFromType<WidgetPlugin>();

        // Invoke chat prompt with auto invocation of functions enabled
        const string ChatPrompt = """
            <message role="user">Create a lime and scarlet colored widget for me.</message>
        """;
        var executionSettings = new MistralAIPromptExecutionSettings { ToolCallBehavior = MistralAIToolCallBehavior.AutoInvokeKernelFunctions };
        var chatSemanticFunction = kernel.CreateFunctionFromPrompt(
            ChatPrompt, executionSettings);
        var chatPromptResult = await kernel.InvokeAsync(chatSemanticFunction);

        Console.WriteLine(chatPromptResult);
    }

    public sealed class WeatherPlugin
    {
        [KernelFunction]
        [Description("Get the current weather in a given location.")]
        public string GetWeather(
            [Description("The city and department, e.g. Marseille, 13")] string location
        ) => "12°C\nWind: 11 KMPH\nHumidity: 48%\nMostly cloudy";
    }

    public sealed class WidgetPlugin
    {
        [KernelFunction]
        [Description("Creates a new widget of the specified type and colors")]
        public string CreateWidget([Description("The colors of the widget to be created")] WidgetColor[] widgetColors)
        {
            var colors = string.Join('-', widgetColors.Select(c => c.GetDisplayName()).ToArray());
            return $"Widget created with colors: {colors}";
        }
    }

    [JsonConverter(typeof(JsonStringEnumConverter))]
    public enum WidgetColor
    {
        [Description("Use when creating a red item.")]
        Red,

        [Description("Use when creating a green item.")]
        Green,

        [Description("Use when creating a blue item.")]
        Blue
    }

    private Kernel CreateKernelWithWeatherPlugin()
    {
        // Create a logging handler to output HTTP requests and responses
        var handler = new LoggingHandler(new HttpClientHandler(), this.Output);
        HttpClient httpClient = new(handler);

        // Create a kernel with MistralAI chat completion and WeatherPlugin
        IKernelBuilder kernelBuilder = Kernel.CreateBuilder();
        kernelBuilder.AddMistralChatCompletion(
                modelId: TestConfiguration.MistralAI.ChatModelId!,
                apiKey: TestConfiguration.MistralAI.ApiKey!,
                httpClient: httpClient);
        kernelBuilder.Plugins.AddFromType<WeatherPlugin>();
        Kernel kernel = kernelBuilder.Build();
        return kernel;
    }
}
```