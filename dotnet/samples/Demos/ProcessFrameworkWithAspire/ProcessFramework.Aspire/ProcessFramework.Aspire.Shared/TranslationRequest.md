# Explanation
This C# class defines a simple data model named `TranslationRequest` within the `ProcessFramework.Aspire.Shared` namespace. The model represents a request to translate a given text and contains a single property, `TextToTranslate`, which stores the text that needs to be translated. This property is initialized to an empty string by default.

```csharp
// Copyright (c) Microsoft. All rights reserved.

namespace ProcessFramework.Aspire.Shared;

/// <summary>
/// Represents a request to translate a given text.
/// </summary>
public class TranslationRequest
{
    /// <summary>
    /// Gets or sets the text to be translated.
    /// </summary>
    public string TextToTranslate { get; set; } = string.Empty;
}
```