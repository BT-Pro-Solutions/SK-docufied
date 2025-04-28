# Explanation
This C# code file defines request models used for evaluation purposes within the `QualityCheckWithFilters.Models` namespace. The base class, `EvaluationRequest`, includes a list of source texts. Two specialized subclasses extend this base request: `SummarizationEvaluationRequest` adds a list of generated summaries, while `TranslationEvaluationRequest` adds a list of generated translations. These models are intended for JSON serialization/deserialization, with properties annotated for correct JSON property naming.

```csharp
// Copyright (c) Microsoft. All rights reserved.

using System.Text.Json.Serialization;

namespace QualityCheckWithFilters.Models;

/// <summary>Base request model with source texts.</summary>
internal class EvaluationRequest
{
    [JsonPropertyName("sources")]
    public List<string> Sources { get; set; }
}

/// <summary>Request model with generated summaries.</summary>
internal sealed class SummarizationEvaluationRequest : EvaluationRequest
{
    [JsonPropertyName("summaries")]
    public List<string> Summaries { get; set; }
}

/// <summary>Request model with generated translations.</summary>
internal sealed class TranslationEvaluationRequest : EvaluationRequest
{
    [JsonPropertyName("translations")]
    public List<string> Translations { get; set; }
}
```
