# Explanation
The `SelectedServiceFilter` class is an implementation of the `IPromptRenderFilter` interface in the `AIModelRouter` namespace. Its purpose is to log (in yellow text to the console) the identifier of the service being used to render a prompt, whenever a prompt is processed. After logging, it resets the console color to white and displays an "Assistant > " prompt. This filter can help with debugging and monitoring which service is selected in a multi-service AI scenario, particularly when using Microsoft Semantic Kernel.

```csharp
// Copyright (c) Microsoft. All rights reserved.

using Microsoft.SemanticKernel;

#pragma warning disable SKEXP0001
#pragma warning disable SKEXP0010
#pragma warning disable CA2249 // Consider using 'string.Contains' instead of 'string.IndexOf'

namespace AIModelRouter;

/// <summary>
/// Using a filter to log the service being used for the prompt.
/// </summary>
internal sealed class SelectedServiceFilter : IPromptRenderFilter
{
    /// <inheritdoc/>
    public Task OnPromptRenderAsync(PromptRenderContext context, Func<PromptRenderContext, Task> next)
    {
        Console.ForegroundColor = ConsoleColor.Yellow;
        Console.WriteLine($"Selected service id: '{context.Arguments.ExecutionSettings?.FirstOrDefault().Key}'");

        Console.ForegroundColor = ConsoleColor.White;
        Console.Write("Assistant > ");
        return next(context);
    }
}
```