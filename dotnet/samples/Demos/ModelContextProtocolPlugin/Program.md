# Explanation

This C# program demonstrates how to connect to the Model Context Protocol (MCP) GitHub server and use its tools as plugins within the Microsoft Semantic Kernel framework. It authenticates with OpenAI to access chat models (such as `gpt-4o-mini`), retrieves the list of AI tools available from the GitHub MCP server, and dynamically adds them as kernel functions. The program also provides an example of using function calling: it asks the kernel (backed by OpenAI) to summarize the last four commits in a GitHub repository. Finally, it defines an agent (`ChatCompletionAgent`) capable of answering questions about GitHub repositories by leveraging these integrated tools and demonstrates an example interaction. The program is designed to be extensible and demonstrates best practices around dependency injection, logging, and function calling with OpenAI's function-calling capabilities.

```csharp
// Copyright (c) Microsoft. All rights reserved.

using Microsoft.Extensions.Configuration;
using Microsoft.Extensions.DependencyInjection;
using Microsoft.Extensions.Logging;
using Microsoft.SemanticKernel;
using Microsoft.SemanticKernel.Agents;
using Microsoft.SemanticKernel.Connectors.OpenAI;
using ModelContextProtocol.Client;
using ModelContextProtocol.Protocol.Transport;

var config = new ConfigurationBuilder()
    .AddUserSecrets<Program>()
    .AddEnvironmentVariables()
    .Build();

if (config["OpenAI:ApiKey"] is not { } apiKey)
{
    Console.Error.WriteLine("Please provide a valid OpenAI:ApiKey to run this sample. See the associated README.md for more details.");
    return;
}

// Create an MCPClient for the GitHub server
await using var mcpClient = await McpClientFactory.CreateAsync(new StdioClientTransport(new()
{
    Name = "MCPServer",
    Command = "npx",
    Arguments = ["-y", "@modelcontextprotocol/server-github"],
}));

// Retrieve the list of tools available on the GitHub server
var tools = await mcpClient.ListToolsAsync().ConfigureAwait(false);
foreach (var tool in tools)
{
    Console.WriteLine($"{tool.Name}: {tool.Description}");
}

// Prepare and build kernel with the MCP tools as Kernel functions
var builder = Kernel.CreateBuilder();
builder.Services
    .AddLogging(c => c.AddDebug().SetMinimumLevel(Microsoft.Extensions.Logging.LogLevel.Trace))
    .AddOpenAIChatCompletion(
        modelId: config["OpenAI:ChatModelId"] ?? "gpt-4o-mini",
        apiKey: apiKey);
Kernel kernel = builder.Build();
kernel.Plugins.AddFromFunctions("GitHub", tools.Select(aiFunction => aiFunction.AsKernelFunction()));

// Enable automatic function calling
OpenAIPromptExecutionSettings executionSettings = new()
{
    Temperature = 0,
    FunctionChoiceBehavior = FunctionChoiceBehavior.Auto(options: new() { RetainArgumentTypes = true })
};

// Test using GitHub tools
var prompt = "Summarize the last four commits to the microsoft/semantic-kernel repository?";
var result = await kernel.InvokePromptAsync(prompt, new(executionSettings)).ConfigureAwait(false);
Console.WriteLine($"\n\n{prompt}\n{result}");

// Define the agent
ChatCompletionAgent agent = new()
{
    Instructions = "Answer questions about GitHub repositories.",
    Name = "GitHubAgent",
    Kernel = kernel,
    Arguments = new KernelArguments(executionSettings),
};

// Respond to user input, invoking functions where appropriate.
ChatMessageContent response = await agent.InvokeAsync("Summarize the last four commits to the microsoft/semantic-kernel repository?").FirstAsync();
Console.WriteLine($"\n\nResponse from GitHubAgent:\n{response.Content}");
```