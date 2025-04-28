# Explanation
This C# class `PromptShieldAnalysis` represents the result of a security assessment on user inputs or documents to detect potential vulnerabilities or malicious attempts such as user prompt attacks or document attacks. It contains a single boolean property `AttackDetected` indicating whether such an attack has been identified. The class is designed for use with JSON serialization, mapping the property to the JSON field "attackDetected". This is relevant in the context of content safety services and prompt shield analysis, often used to interpret API responses for threat detection.

```csharp
// Copyright (c) Microsoft. All rights reserved.

using System.Text.Json.Serialization;

namespace ContentSafety.Services.PromptShield;

/// <summary>
/// Flags potential vulnerabilities within user input.
/// More information here: https://learn.microsoft.com/en-us/azure/ai-services/content-safety/quickstart-jailbreak#interpret-the-api-response
/// </summary>
public class PromptShieldAnalysis
{
    /// <summary>
    /// Indicates whether a User Prompt attack (for example, malicious input, security threat) has been detected in the user prompt or
    /// a Document attack (for example, commands, malicious input) has been detected in the document.
    /// </summary>
    [JsonPropertyName("attackDetected")]
    public bool AttackDetected { get; set; }
}
```