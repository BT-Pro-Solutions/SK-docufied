# Explanation
This C# code demonstrates how to use the Code Interpreter tool with an AzureAI Agent via the Microsoft Semantic Kernel. Within a unit test, it:

- Initializes an AzureAI agent with access to the Code Interpreter tool.
- Creates a conversation thread for agent interactions.
- Asks the agent to compute Fibonacci sequence values less than 101 using code.
- Displays the conversation messages between user and agent.
- Properly cleans up by deleting the agent and thread after use.

This example is useful for understanding how to integrate computational tools (like code interpreters) into AzureAI agent workflows for programmatically responding to user queries involving code execution.

```csharp
// Copyright (c) Microsoft. All rights reserved.
using Microsoft.SemanticKernel;
using Microsoft.SemanticKernel.Agents;
using Microsoft.SemanticKernel.Agents.AzureAI;
using Microsoft.SemanticKernel.ChatCompletion;

namespace GettingStarted.AzureAgents;

/// <summary>
/// Demonstrate using code-interpreter on <see cref="AzureAIAgent"/> .
/// </summary>
public class Step04_AzureAIAgent_CodeInterpreter(ITestOutputHelper output) : BaseAzureAgentTest(output)
{
    [Fact]
    public async Task UseCodeInterpreterToolWithAgent()
    {
        // Define the agent
        Azure.AI.Projects.Agent definition = await this.AgentsClient.CreateAgentAsync(
            TestConfiguration.AzureAI.ChatModelId,
            tools: [new Azure.AI.Projects.CodeInterpreterToolDefinition()]);
        AzureAIAgent agent = new(definition, this.AgentsClient);

        // Create a thread for the agent conversation.
        AgentThread thread = new AzureAIAgentThread(this.AgentsClient, metadata: SampleMetadata);

        // Respond to user input
        try
        {
            await InvokeAgentAsync("Use code to determine the values in the Fibonacci sequence that that are less then the value of 101?");
        }
        finally
        {
            await thread.DeleteAsync();
            await this.AgentsClient.DeleteAgentAsync(agent.Id);
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