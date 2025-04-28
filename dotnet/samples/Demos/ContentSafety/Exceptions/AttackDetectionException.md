# Explanation

This C# code defines a custom exception class, `AttackDetectionException`, which is intended for use in applications employing content safety and prompt analysis, particularly to detect prompt injection or jailbreak attacks. The exception is thrown when attacks are detected in user prompts or associated documents. It includes properties to hold detailed analysis results (`PromptShieldAnalysis`) for both the user prompt and documents, and overrides the `Data` property to include these details for further inspection or logging. The code references Microsoft's Azure Content Safety API context, indicating relevant documentation for interpreting API responses.

```csharp
// Copyright (c) Microsoft. All rights reserved.

using System.Collections;
using ContentSafety.Services.PromptShield;

namespace ContentSafety.Exceptions;

/// <summary>
/// Exception which is thrown when attack is detected in user prompt or documents.
/// More information here: https://learn.microsoft.com/en-us/azure/ai-services/content-safety/quickstart-jailbreak#interpret-the-api-response
/// </summary>
public class AttackDetectionException : Exception
{
    /// <summary>
    /// Contains analysis result for the user prompt.
    /// </summary>
    public PromptShieldAnalysis? UserPromptAnalysis { get; init; }

    /// <summary>
    /// Contains a list of analysis results for each document provided.
    /// </summary>
    public IReadOnlyList<PromptShieldAnalysis>? DocumentsAnalysis { get; init; }

    /// <summary>
    /// Dictionary with additional details of exception.
    /// </summary>
    public override IDictionary Data => new Dictionary<string, object?>()
    {
        ["userPrompt"] = this.UserPromptAnalysis,
        ["documents"] = this.DocumentsAnalysis,
    };

    public AttackDetectionException()
    {
    }

    public AttackDetectionException(string? message) : base(message)
    {
    }

    public AttackDetectionException(string? message, Exception? innerException) : base(message, innerException)
    {
    }
}
```
