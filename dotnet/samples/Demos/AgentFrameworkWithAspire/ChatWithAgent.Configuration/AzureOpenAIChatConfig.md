# Explanation
This C# class defines a configuration model for Azure OpenAI chat services. It includes properties for specifying the deployment name, model name, model version, and optional SKU details. The `DeploymentName` property is required, which typically corresponds to a specific deployment of the Azure OpenAI service. This class is designed to be used for strongly typed configuration binding within an application, facilitating the management of Azure OpenAI chat settings.

```csharp
// Copyright (c) Microsoft. All rights reserved.

using System.ComponentModel.DataAnnotations;

namespace ChatWithAgent.Configuration;

/// <summary>
/// Azure OpenAI chat configuration.
/// </summary>
public sealed class AzureOpenAIChatConfig
{
    /// <summary>
    /// Configuration section name.
    /// </summary>
    public const string ConfigSectionName = "AzureOpenAIChat";

    /// <summary>
    /// The name of the chat deployment.
    /// </summary>
    [Required]
    public string DeploymentName { get; set; } = string.Empty;

    /// <summary>
    /// The name of the chat model.
    /// </summary>
    public string ModelName { get; set; } = string.Empty;

    /// <summary>
    /// The chat model version.
    /// </summary>
    public string ModelVersion { get; set; } = string.Empty;

    /// <summary>
    /// The SKU name.
    /// </summary>
    public string? SkuName { get; set; }

    /// <summary>
    /// The SKU capacity
    /// </summary>
    public int? SkuCapacity { get; set; }
}
```