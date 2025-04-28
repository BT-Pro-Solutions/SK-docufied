# Explanation
This C# code defines a data model named `ChatModel` that represents the request structure for a chat endpoint in the `ContentSafety.Models` namespace. The class includes:
- A required string property `Message` to hold the chat message content.
- An optional list of strings `Documents` to provide additional context or references for the chat request.
The `[Required]` data annotation indicates that the `Message` property must be provided in any request using this model.

```csharp
// Copyright (c) Microsoft. All rights reserved.

using System.ComponentModel.DataAnnotations;

namespace ContentSafety.Models;

/// <summary>
/// Request model for chat endpoint.
/// </summary>
public class ChatModel
{
    [Required]
    public string Message { get; set; }

    public List<string>? Documents { get; set; }
}
```