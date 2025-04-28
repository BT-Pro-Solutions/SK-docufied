# Explanation
This code defines a sample ASP.NET Core chat controller named `ChatController` for a web API. The controller receives chat messages via HTTP POST, constructs a prompt using the provided user message and optional documents, and invokes a Semantic Kernel to generate a friendly assistant response. The result is returned as an HTTP response. The controller depends on the `Kernel` object (from Microsoft.SemanticKernel) and uses a model called `ChatModel` for the request body.

```csharp
// Copyright (c) Microsoft. All rights reserved.

using ContentSafety.Models;
using Microsoft.AspNetCore.Mvc;
using Microsoft.SemanticKernel;

namespace ContentSafety.Controllers;

/// <summary>
/// Sample chat controller.
/// </summary>
[ApiController]
[Route("[controller]")]
public class ChatController(Kernel kernel) : ControllerBase
{
    private const string Prompt =
        """
        <message role="system">You are friendly assistant.</message>
        <message role="user">{{$userMessage}}</message>
        """;

    private readonly Kernel _kernel = kernel;

    [HttpPost]
    public async Task<IActionResult> PostAsync(ChatModel chat)
    {
        var arguments = new KernelArguments
        {
            ["userMessage"] = chat.Message,
            ["documents"] = chat.Documents
        };

        return this.Ok((await this._kernel.InvokePromptAsync(Prompt, arguments)).ToString());
    }
}
```