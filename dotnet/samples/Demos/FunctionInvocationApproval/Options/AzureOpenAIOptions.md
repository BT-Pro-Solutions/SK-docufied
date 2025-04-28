# Explanation
This C# class `AzureOpenAIOptions` defines configuration settings required to connect to the Azure OpenAI chat completion service. It holds properties for the deployment name, endpoint URL, and API key. Additionally, it includes a validation property `IsValid` that ensures all required configuration fields are non-empty. This class facilitates loading configuration from a settings section named `"AzureOpenAI"` and can be used to ensure that the Azure OpenAI connection details are properly provided before use.

```csharp
// Copyright (c) Microsoft. All rights reserved.

namespace FunctionInvocationApproval.Options;

/// <summary>
/// Configuration for Azure OpenAI chat completion service.
/// </summary>
public class AzureOpenAIOptions
{
    public const string SectionName = "AzureOpenAI";

    /// <summary>
    /// Azure OpenAI deployment name, see https://learn.microsoft.com/azure/cognitive-services/openai/how-to/create-resource
    /// </summary>
    public string ChatDeploymentName { get; set; }

    /// <summary>
    /// Azure OpenAI deployment URL, see https://learn.microsoft.com/azure/cognitive-services/openai/quickstart
    /// </summary>
    public string Endpoint { get; set; }

    /// <summary>
    /// Azure OpenAI API key, see https://learn.microsoft.com/azure/cognitive-services/openai/quickstart
    /// </summary>
    public string ApiKey { get; set; }

    public bool IsValid =>
        !string.IsNullOrWhiteSpace(this.ChatDeploymentName) &&
        !string.IsNullOrWhiteSpace(this.Endpoint) &&
        !string.IsNullOrWhiteSpace(this.ApiKey);
}
```