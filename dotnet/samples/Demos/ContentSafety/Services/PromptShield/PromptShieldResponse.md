# Explanation
This C# class defines a data model `PromptShieldResponse` that represents the response from a service analyzing user prompts and documents for potential vulnerabilities or risks. This model contains analysis results for a user prompt and a list of analyses for multiple documents. It is designed for use with JSON serialization, matching the expected property names from the Content Safety API provided by Microsoft.

```csharp
// Copyright (c) Microsoft. All rights reserved.

using System.Text.Json.Serialization;

namespace ContentSafety.Services.PromptShield;

/// <summary>
/// Flags potential vulnerabilities within user prompt and documents.
/// More information here: https://learn.microsoft.com/en-us/azure/ai-services/content-safety/quickstart-jailbreak#interpret-the-api-response
/// </summary>
public class PromptShieldResponse
{
    /// <summary>
    /// Contains analysis results for the user prompt.
    /// </summary>
    [JsonPropertyName("userPromptAnalysis")]
    public PromptShieldAnalysis? UserPromptAnalysis { get; set; }

    /// <summary>
    /// Contains a list of analysis results for each document provided.
    /// </summary>
    [JsonPropertyName("documentsAnalysis")]
    public List<PromptShieldAnalysis>? DocumentsAnalysis { get; set; }
}
```