# Explanation
This C# code defines a filter class `MeteorSummarizationEvaluationFilter` that integrates into a function invocation pipeline to evaluate the quality of text summarization results. It uses the METEOR metric, which measures the similarity between the original source text and the generated summary. The METEOR score ranges from 0 to 1, with higher values indicating better summary quality. After a function invocation completes, the filter extracts the source text and the generated summary, sends them to an evaluation service, logs the score, and throws an exception if the score falls below a specified threshold. This ensures only summaries of sufficient quality are accepted.

```csharp
// Copyright (c) Microsoft. All rights reserved.

using Microsoft.Extensions.Logging;
using Microsoft.SemanticKernel;
using QualityCheckWithFilters.Models;
using QualityCheckWithFilters.Services;

namespace QualityCheckWithFilters.Filters;

/// <summary>
/// Filter which performs text summarization evaluation using METEOR metric: https://huggingface.co/spaces/evaluate-metric/meteor.
/// METEOR score ranges from 0 to 1, where higher values indicate better similarity between original text and generated summary.
/// </summary>
internal sealed class MeteorSummarizationEvaluationFilter(
    EvaluationService evaluationService,
    ILogger logger,
    double threshold) : IFunctionInvocationFilter
{
    public async Task OnFunctionInvocationAsync(FunctionInvocationContext context, Func<FunctionInvocationContext, Task> next)
    {
        await next(context);

        var sourceText = context.Result.RenderedPrompt!;
        var summary = context.Result.ToString();

        var request = new SummarizationEvaluationRequest { Sources = [sourceText], Summaries = [summary] };
        var response = await evaluationService.EvaluateAsync<SummarizationEvaluationRequest, MeteorSummarizationEvaluationResponse>(request);

        var score = Math.Round(response.Score, 4);

        logger.LogInformation("[METEOR] Score: {Score}", score);

        if (score < threshold)
        {
            throw new KernelException($"METEOR summary evaluation score ({score}) is lower than threshold ({threshold})");
        }
    }
}
```