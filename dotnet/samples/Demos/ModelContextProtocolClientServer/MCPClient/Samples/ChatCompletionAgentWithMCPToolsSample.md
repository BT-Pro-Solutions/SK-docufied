# Explanation
This C# sample demonstrates how to use the `ChatCompletionAgent` alongside MCP (Model Context Protocol) tools, which are represented as Kernel functions within the Microsoft Semantic Kernel framework. The example shows how to:

1. Create an MCP client to connect to MCP servers.
2. Retrieve the list of available tools (functions) from the MCP server.
3. Register those MCP tools as Kernel functions within a Semantic Kernel instance.
4. Configure a chat completion agent with specific instructions, a name, the kernel, and execution settings that enable automatic function calling.
5. Invoke the chat completion agent with a user prompt.
6. The agent interacts with the AI model which calls the necessary MCP functions in sequence to answer the prompt.
7. The sample highlights how function calls are chained, such as retrieving current UTC time and then calling a weather API function with the retrieved time and city name.
8. Finally, the response is displayed, showing how the AI leverages external MCP tools for dynamic information retrieval in conversational scenarios.

This pattern facilitates integrating external APIs and dynamic data sources into conversational AI workflows by exposing them as functions callable by the chat model.

```csharp
// Copyright (c) Microsoft. All rights reserved.

using System;
using System.Collections.Generic;
using System.Linq;
using System.Threading.Tasks;
using Microsoft.SemanticKernel;
using Microsoft.SemanticKernel.Agents;
using Microsoft.SemanticKernel.Connectors.OpenAI;
using ModelContextProtocol.Client;

namespace MCPClient.Samples;

/// <summary>
/// Demonstrates how to use <see cref="ChatCompletionAgent"/> with MCP tools represented as Kernel functions.
/// </summary>
internal sealed class ChatCompletionAgentWithMCPToolsSample : BaseSample
{
    /// <summary>
    /// Demonstrates how to use <see cref="ChatCompletionAgent"/> with MCP tools represented as Kernel functions.
    /// The code in this method:
    /// 1. Creates an MCP client.
    /// 2. Retrieves the list of tools provided by the MCP server.
    /// 3. Creates a kernel and registers the MCP tools as Kernel functions.
    /// 4. Defines chat completion agent with instructions, name, kernel, and arguments.
    /// 5. Invokes the agent with a prompt.
    /// 6. The agent sends the prompt to the AI model, together with the MCP tools represented as Kernel functions.
    /// 7. The AI model calls DateTimeUtils-GetCurrentDateTimeInUtc function to get the current date time in UTC required as an argument for the next function.
    /// 8. The AI model calls WeatherUtils-GetWeatherForCity function with the current date time and the `Boston` arguments extracted from the prompt to get the weather information.
    /// 9. Having received the weather information from the function call, the AI model returns the answer to the agent and the agent returns the answer to the user.
    /// </summary>
    public static async Task RunAsync()
    {
        Console.WriteLine($"Running the {nameof(ChatCompletionAgentWithMCPToolsSample)} sample.");

        // Create an MCP client
        await using IMcpClient mcpClient = await CreateMcpClientAsync();

        // Retrieve and display the list provided by the MCP server
        IList<McpClientTool> tools = await mcpClient.ListToolsAsync();
        DisplayTools(tools);

        // Create a kernel and register the MCP tools as kernel functions
        Kernel kernel = CreateKernelWithChatCompletionService();
        kernel.Plugins.AddFromFunctions("Tools", tools.Select(aiFunction => aiFunction.AsKernelFunction()));

        // Enable automatic function calling
        OpenAIPromptExecutionSettings executionSettings = new()
        {
            FunctionChoiceBehavior = FunctionChoiceBehavior.Auto(options: new() { RetainArgumentTypes = true })
        };

        string prompt = "What is the likely color of the sky in Boston today?";
        Console.WriteLine(prompt);

        // Define the agent
        ChatCompletionAgent agent = new()
        {
            Instructions = "Answer questions about the weather.",
            Name = "WeatherAgent",
            Kernel = kernel,
            Arguments = new KernelArguments(executionSettings),
        };

        // Invokes agent with a prompt
        ChatMessageContent response = await agent.InvokeAsync(prompt).FirstAsync();

        Console.WriteLine(response);
        Console.WriteLine();

        // The expected output is: The sky in Boston today is likely gray due to rainy weather.
    }
}
```
