# Explanation

This C# code demonstrates how to use an Azure AI Agent, specifically an `AzureAIAgent` with the Code Interpreter tool, to manipulate and analyze CSV files. The example is structured as a unit test that uploads a CSV file, defines an agent with access to that file, interacts with the agent by asking data-related questions, and requests a report. The agent responds using the Code Interpreter capabilities, showing how to automate data analysis and file generation workflows. Finally, the code ensures cleanup by deleting created resources (threads, agents, and files) after execution. This is particularly useful for integrating AI-powered data analysis into applications using the Microsoft Semantic Kernel and Azure AI services.

```csharp
// Copyright (c) Microsoft. All rights reserved.
using Azure.AI.Projects;
using Microsoft.SemanticKernel;
using Microsoft.SemanticKernel.Agents.AzureAI;
using Microsoft.SemanticKernel.ChatCompletion;
using Resources;
using Agent = Azure.AI.Projects.Agent;

namespace Agents;

/// <summary>
/// Demonstrate using code-interpreter to manipulate and generate csv files with <see cref="AzureAIAgent"/> .
/// </summary>
public class AzureAIAgent_FileManipulation(ITestOutputHelper output) : BaseAzureAgentTest(output)
{
    [Fact]
    public async Task AnalyzeCSVFileUsingAzureAIAgentAsync()
    {
        await using Stream stream = EmbeddedResource.ReadStream("sales.csv")!;
        AgentFile fileInfo = await this.AgentsClient.UploadFileAsync(stream, AgentFilePurpose.Agents, "sales.csv");

        // Define the agent
        Agent definition = await this.AgentsClient.CreateAgentAsync(
            TestConfiguration.AzureAI.ChatModelId,
            tools: [new CodeInterpreterToolDefinition()],
            toolResources:
                new()
                {
                    CodeInterpreter = new()
                    {
                        FileIds = { fileInfo.Id },
                    }
                });
        AzureAIAgent agent = new(definition, this.AgentsClient);
        AzureAIAgentThread thread = new(this.AgentsClient);

        // Respond to user input
        try
        {
            await InvokeAgentAsync("Which segment had the most sales?");
            await InvokeAgentAsync("List the top 5 countries that generated the most profit.");
            await InvokeAgentAsync("Create a tab delimited file report of profit by each country per month.");
        }
        finally
        {
            await thread.DeleteAsync();
            await this.AgentsClient.DeleteAgentAsync(agent.Id);
            await this.AgentsClient.DeleteFileAsync(fileInfo.Id);
        }

        // Local function to invoke agent and display the conversation messages.
        async Task InvokeAgentAsync(string input)
        {
            ChatMessageContent message = new(AuthorRole.User, input);
            this.WriteAgentChatMessage(message);

            await foreach (ChatMessageContent response in agent.InvokeAsync(message, thread))
            {
                this.WriteAgentChatMessage(response);
                await this.DownloadContentAsync(response);
            }
        }
    }
}
```