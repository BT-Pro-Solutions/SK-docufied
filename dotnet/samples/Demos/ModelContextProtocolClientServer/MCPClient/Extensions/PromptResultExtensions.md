# Explanation
This C# code defines extension methods for converting prompt results and messages into chat message contents compatible with the Semantic Kernel framework. Specifically, it provides conversions from `GetPromptResult` objects to a list of `ChatMessageContent` objects, and from individual `PromptMessage` instances into `ChatMessageContent`. This enables easy integration between the custom prompt message types used in the ModelContextProtocol and the Microsoft Semantic Kernel's chat completion structures.

```csharp
// Copyright (c) Microsoft. All rights reserved.

using System.Collections.Generic;
using System.Linq;
using Microsoft.SemanticKernel;
using Microsoft.SemanticKernel.ChatCompletion;
using ModelContextProtocol.Protocol.Types;

namespace MCPClient;

/// <summary>
/// Extension methods for <see cref="GetPromptResult"/>.
/// </summary>
internal static class PromptResultExtensions
{
    /// <summary>
    /// Converts a <see cref="GetPromptResult"/> to chat message contents.
    /// </summary>
    /// <param name="result">The prompt result to convert.</param>
    /// <returns>The corresponding <see cref="ChatHistory"/>.</returns>
    public static IList<ChatMessageContent> ToChatMessageContents(this GetPromptResult result)
    {
        return [.. result.Messages.Select(ToChatMessageContent)];
    }

    /// <summary>
    /// Converts a <see cref="PromptMessage"/> to a <see cref="ChatMessageContent"/>.
    /// </summary>
    /// <param name="message">The <see cref="PromptMessage"/> to convert.</param>
    /// <returns>The corresponding <see cref="ChatMessageContent"/>.</returns>
    public static ChatMessageContent ToChatMessageContent(this PromptMessage message)
    {
        return new ChatMessageContent(role: message.Role.ToAuthorRole(), items: [message.Content.ToKernelContent()]);
    }
}
```