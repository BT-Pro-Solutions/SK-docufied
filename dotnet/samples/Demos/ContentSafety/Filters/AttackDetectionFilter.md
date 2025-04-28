# Explanation
This C# code defines a filter called `AttackDetectionFilter` for Microsoft Semantic Kernel applications that utilize Azure AI's Content Safety Prompt Shield service. The purpose of the filter is to detect prompt injection attacks before further action is taken on rendered prompts. The filter operates as follows:

- After rendering a prompt, it retrieves the prompt text and any associated documents.
- It calls the Prompt Shield service to analyze both the user-provided prompt and the documents for possible attacks.
- If an attack is detected in either the prompt or the documents, it throws an `AttackDetectionException`, preventing further processing. 
- This mechanism helps defend against prompt attacks and jailbreak attempts, contributing to safer use of AI-backed semantic kernel features. The filter is intended to be used as an implementation of the `IPromptRenderFilter` interface.

```csharp
// Copyright (c) Microsoft. All rights reserved.

using ContentSafety.Exceptions;
using ContentSafety.Services.PromptShield;
using Microsoft.SemanticKernel;

namespace ContentSafety.Filters;

/// <summary>
/// This filter performs attack detection using Azure AI Content Safety - Prompt Shield service.
/// For more information: https://learn.microsoft.com/en-us/azure/ai-services/content-safety/quickstart-jailbreak
/// </summary>
public class AttackDetectionFilter(PromptShieldService promptShieldService) : IPromptRenderFilter
{
    private readonly PromptShieldService _promptShieldService = promptShieldService;

    public async Task OnPromptRenderAsync(PromptRenderContext context, Func<PromptRenderContext, Task> next)
    {
        // Running prompt rendering operation
        await next(context);

        // Getting rendered prompt
        var prompt = context.RenderedPrompt;

        // Getting documents data from kernel
        var documents = context.Arguments["documents"] as List<string>;

        // Calling Prompt Shield service for attack detection
        var response = await this._promptShieldService.DetectAttackAsync(new PromptShieldRequest
        {
            UserPrompt = prompt!,
            Documents = documents
        });

        var attackDetected =
            response.UserPromptAnalysis?.AttackDetected is true ||
            response.DocumentsAnalysis?.Any(l => l.AttackDetected) is true;

        if (attackDetected)
        {
            throw new AttackDetectionException("Attack detected. Operation is denied.")
            {
                UserPromptAnalysis = response.UserPromptAnalysis,
                DocumentsAnalysis = response.DocumentsAnalysis
            };
        }
    }
}
```
