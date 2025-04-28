# Explanation
This C# code defines a sealed class `AgentCompletionRequest` which models a request for agent-based chat completion. It includes properties for the prompt text, the chat history, and a flag indicating whether the response should be streamed. This class is part of the `ChatWithAgent.ApiService` namespace and uses the `Microsoft.SemanticKernel.ChatCompletion` namespace for the `ChatHistory` type.

```csharp
// Copyright (c) Microsoft. All rights reserved.

using Microsoft.SemanticKernel.ChatCompletion;

namespace ChatWithAgent.ApiService;

/// <summary>
/// The agent completion request model.
/// </summary>
public sealed class AgentCompletionRequest
{
    /// <summary>
    /// Gets or sets the prompt.
    /// </summary>
    public required string Prompt { get; set; }

    /// <summary>
    /// Gets or sets the chat history.
    /// </summary>
    public required ChatHistory ChatHistory { get; set; }

    /// <summary>
    /// Gets or sets a value indicating whether streaming is requested.
    /// </summary>
    public bool IsStreaming { get; set; }
}
```