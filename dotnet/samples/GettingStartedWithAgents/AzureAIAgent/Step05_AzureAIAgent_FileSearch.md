# Explanation

This C# code demonstrates how to use an `AzureAIAgent` with file search capabilities in an Azure AI environment. It is structured as a test class that shows how to upload a file (e.g., "employees.pdf"), create a vector store for semantic search, and configure an agent with file search tools. The example sends several queries to the agent, which responds based on the content of the uploaded file. It also properly cleans up resources after the test. This example is relevant for developers integrating Azure AI Agents with document-based information retrieval.

```csharp
// Copyright (c) Microsoft. All rights reserved.
using Azure.AI.Projects;
using Microsoft.SemanticKernel;
using Microsoft.SemanticKernel.Agents.AzureAI;
using Microsoft.SemanticKernel.ChatCompletion;
using Resources;

namespace GettingStarted.AzureAgents;

/// <summary>
/// Demonstrate using <see cref="AzureAIAgent"/> with file search.
/// </summary>
public class Step05_AzureAIAgent_FileSearch(ITestOutputHelper output) : BaseAzureAgentTest(output)
{
    [Fact]
    public async Task UseFileSearchToolWithAgent()
    {
        // Define the agent
        await using Stream stream = EmbeddedResource.ReadStream("employees.pdf")!;

        AgentFile fileInfo = await this.AgentsClient.UploadFileAsync(stream, AgentFilePurpose.Agents, "employees.pdf");
        VectorStore fileStore =
            await this.AgentsClient.CreateVectorStoreAsync(
                [fileInfo.Id],
                metadata: new Dictionary<string, string>() { { SampleMetadataKey, bool.TrueString } });
        Agent agentModel = await this.AgentsClient.CreateAgentAsync(
            TestConfiguration.AzureAI.ChatModelId,
            tools: [new FileSearchToolDefinition()],
            toolResources: new()
            {
                FileSearch = new()
                {
                    VectorStoreIds = { fileStore.Id },
                }
            },
            metadata: new Dictionary<string, string>() { { SampleMetadataKey, bool.TrueString } });
        AzureAIAgent agent = new(agentModel, this.AgentsClient);

        // Create a thread associated for the agent conversation.
        Microsoft.SemanticKernel.Agents.AgentThread thread = new AzureAIAgentThread(this.AgentsClient, metadata: SampleMetadata);

        // Respond to user input
        try
        {
            await InvokeAgentAsync("Who is the youngest employee?");
            await InvokeAgentAsync("Who works in sales?");
            await InvokeAgentAsync("I have a customer request, who can help me?");
        }
        finally
        {
            await thread.DeleteAsync();
            await this.AgentsClient.DeleteAgentAsync(agent.Id);
            await this.AgentsClient.DeleteVectorStoreAsync(fileStore.Id);
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
            }
        }
    }
}
```