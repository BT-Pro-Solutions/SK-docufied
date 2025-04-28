# Explanation

This C# code demonstrates how to use the `OpenAIAssistantAgent` class from the Microsoft Semantic Kernel library to interact with a file-search capability via the OpenAI Assistants API. The example creates an OpenAI assistant with file search enabled, uploads a PDF file (a fictional employee table), stores it in a vector store for semantic search, and conducts a multi-turn agent conversation by asking various questions about the file's contents. The logic also includes cleanup steps to delete resources after usage. This demonstrates how to integrate file-based retrieval and conversational AI using OpenAI Assistants within the Semantic Kernel ecosystem.

```csharp
// Copyright (c) Microsoft. All rights reserved.

using Microsoft.SemanticKernel;
using Microsoft.SemanticKernel.Agents;
using Microsoft.SemanticKernel.Agents.OpenAI;
using Microsoft.SemanticKernel.ChatCompletion;
using OpenAI.Assistants;
using Resources;

namespace GettingStarted.OpenAIAssistants;

/// <summary>
/// Demonstrate using <see cref="OpenAIAssistantAgent"/> with file search.
/// </summary>
public class Step05_AssistantTool_FileSearch(ITestOutputHelper output) : BaseAssistantTest(output)
{
    [Fact]
    public async Task UseFileSearchToolWithAssistantAgent()
    {
        // Define the assistant
        Assistant assistant =
            await this.AssistantClient.CreateAssistantAsync(
                this.Model,
                enableFileSearch: true,
                metadata: SampleMetadata);

        // Create the agent
        OpenAIAssistantAgent agent = new(assistant, this.AssistantClient);

        // Upload file - Using a table of fictional employees.
        await using Stream stream = EmbeddedResource.ReadStream("employees.pdf")!;
        string fileId = await this.Client.UploadAssistantFileAsync(stream, "employees.pdf");

        // Create a vector-store
        string vectorStoreId =
            await this.Client.CreateVectorStoreAsync(
                [fileId],
                waitUntilCompleted: true,
                metadata: SampleMetadata);

        // Create a thread associated with a vector-store for the agent conversation.
        AgentThread thread = new OpenAIAssistantAgentThread(
            this.AssistantClient,
            vectorStoreId: vectorStoreId,
            metadata: SampleMetadata);

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
            await this.AssistantClient.DeleteAssistantAsync(agent.Id);
            await this.Client.DeleteVectorStoreAsync(vectorStoreId);
            await this.Client.DeleteFileAsync(fileId);
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