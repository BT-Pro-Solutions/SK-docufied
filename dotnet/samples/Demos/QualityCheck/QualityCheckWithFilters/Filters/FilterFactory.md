# Explanation
This C# code defines a `FilterFactory` class that acts as a factory for creating different types of function invocation filters based on an evaluation score type. The factory uses a dictionary to map `EvaluationScoreType` enum values to corresponding constructors of various evaluation filters (BERT, BLEU, METEOR, COMET). Each filter is created with an `EvaluationService`, a logger, and a threshold value. This pattern centralizes the creation logic for filters and makes the code more maintainable and extensible.

```csharp
// Copyright (c) Microsoft. All rights reserved.

using Microsoft.Extensions.Logging;
using Microsoft.SemanticKernel;
using QualityCheckWithFilters.Models;
using QualityCheckWithFilters.Services;

namespace QualityCheckWithFilters.Filters;

/// <summary>
/// Factory class for function invocation filters based on evaluation score type.
/// </summary>
internal sealed class FilterFactory
{
    private static readonly Dictionary<EvaluationScoreType, Func<EvaluationService, ILogger, double, IFunctionInvocationFilter>> s_filters = new()
    {
        [EvaluationScoreType.BERT] = (service, logger, threshold) => new BertSummarizationEvaluationFilter(service, logger, threshold),
        [EvaluationScoreType.BLEU] = (service, logger, threshold) => new BleuSummarizationEvaluationFilter(service, logger, threshold),
        [EvaluationScoreType.METEOR] = (service, logger, threshold) => new MeteorSummarizationEvaluationFilter(service, logger, threshold),
        [EvaluationScoreType.COMET] = (service, logger, threshold) => new CometTranslationEvaluationFilter(service, logger, threshold),
    };

    public static IFunctionInvocationFilter Create(EvaluationScoreType type, EvaluationService evaluationService, ILogger logger, double threshold)
        => s_filters[type].Invoke(evaluationService, logger, threshold);
}
```