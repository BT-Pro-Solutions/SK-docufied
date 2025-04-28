# Explanation

This C# file defines an ASP.NET Core API controller called `AutoFunctionCallingController`, which demonstrates a modern approach to using AI planning capabilities by leveraging "Auto Function Calling" via the Microsoft Semantic Kernel framework. The controller exposes three POST endpoints:

1. **generate-plan**: Generates a plan based on a user-specified goal, populating and returning a chat history containing the plan.
2. **execute-new-plan**: Executes a new plan directly from a given goal, returning the result.
3. **execute-existing-plan**: Executes a previously generated plan (retrieved from storage), returning its outcome.

The controller imports sample plugins (e.g., time and weather), and utilizes services for chat completion and plan provision. It demonstrates how to configure function-calling behavior in prompt execution with OpenAI models via Semantic Kernel. This is intended as a demo of best practices for orchestrating multi-step functions in Large Language Model (LLM) applications with automatic function selection.

```csharp
// Copyright (c) Microsoft. All rights reserved.

#pragma warning disable IDE0005 // Using directive is unnecessary

using System.Threading.Tasks;
using Microsoft.AspNetCore.Mvc;
using Microsoft.SemanticKernel;
using Microsoft.SemanticKernel.ChatCompletion;
using Microsoft.SemanticKernel.Connectors.OpenAI;
using StepwisePlannerMigration.Models;
using StepwisePlannerMigration.Plugins;
using StepwisePlannerMigration.Services;

#pragma warning restore IDE0005 // Using directive is unnecessary

namespace StepwisePlannerMigration.Controllers;

/// <summary>
/// This controller shows a new recommended approach how to use planning capability by using Auto Function Calling.
/// </summary>
[ApiController]
[Route("auto-function-calling")]
public class AutoFunctionCallingController : ControllerBase
{
    private readonly Kernel _kernel;
    private readonly IChatCompletionService _chatCompletionService;
    private readonly IPlanProvider _planProvider;

    public AutoFunctionCallingController(
        Kernel kernel,
        IChatCompletionService chatCompletionService,
        IPlanProvider planProvider)
    {
        this._kernel = kernel;
        this._chatCompletionService = chatCompletionService;
        this._planProvider = planProvider;

        this._kernel.ImportPluginFromType<TimePlugin>();
        this._kernel.ImportPluginFromType<WeatherPlugin>();
    }

    /// <summary>
    /// Action to generate a plan. Generated plan will be populated in <see cref="ChatHistory"/> object.
    /// </summary>
    [HttpPost, Route("generate-plan")]
    public async Task<IActionResult> GeneratePlanAsync(PlanRequest request)
    {
        ChatHistory chatHistory = [];
        chatHistory.AddUserMessage(request.Goal);

        OpenAIPromptExecutionSettings executionSettings = new() { FunctionChoiceBehavior = FunctionChoiceBehavior.Auto() };

        await this._chatCompletionService.GetChatMessageContentAsync(chatHistory, executionSettings, this._kernel);

        return this.Ok(chatHistory);
    }

    /// <summary>
    /// Action to execute a new plan. When generated plan is not needed,
    /// planning result can be obtained directly with <see cref="Kernel"/> object.
    /// </summary>
    [HttpPost, Route("execute-new-plan")]
    public async Task<IActionResult> ExecuteNewPlanAsync(PlanRequest request)
    {
        OpenAIPromptExecutionSettings executionSettings = new() { FunctionChoiceBehavior = FunctionChoiceBehavior.Auto() };

        FunctionResult result = await this._kernel.InvokePromptAsync(request.Goal, new(executionSettings));

        return this.Ok(result.ToString());
    }

    /// <summary>
    /// Action to execute existing plan. Generated plans can be stored in permanent storage for reusability.
    /// In this demo application it is stored in file.
    /// </summary>
    [HttpPost, Route("execute-existing-plan")]
    public async Task<IActionResult> ExecuteExistingPlanAsync()
    {
        ChatHistory chatHistory = this._planProvider.GetPlan("auto-function-calling-plan.json");
        OpenAIPromptExecutionSettings executionSettings = new() { FunctionChoiceBehavior = FunctionChoiceBehavior.Auto() };

        ChatMessageContent result = await this._chatCompletionService.GetChatMessageContentAsync(chatHistory, executionSettings, this._kernel);

        return this.Ok(result.Content);
    }
}
```