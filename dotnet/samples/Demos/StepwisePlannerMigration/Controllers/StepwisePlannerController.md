# Explanation

This code file defines the `StepwisePlannerController` in a C# ASP.NET Core Web API application. The controller demonstrates how to use the `FunctionCallingStepwisePlanner` class from Microsoft Semantic Kernel to generate and execute stepwise plans using plugins (such as time and weather). The controller has three main endpoints:
- **generate-plan**: Generates a new plan based on a provided goal, returning a chat history of the planning session.
- **execute-new-plan**: Executes a new stepwise plan based on a provided goal and returns the final answer.
- **execute-existing-plan**: Executes an existing, previously stored plan from a file, allowing for reusability.

The controller depends on `Kernel`, `FunctionCallingStepwisePlanner`, and `IPlanProvider` services for its operation. The code serves as an example of the older, explicit stepwise planning method, with a pointer to a newer approach in another controller.

```csharp
// Copyright (c) Microsoft. All rights reserved.

#pragma warning disable IDE0005 // Using directive is unnecessary

using System.Threading.Tasks;
using Microsoft.AspNetCore.Mvc;
using Microsoft.SemanticKernel;
using Microsoft.SemanticKernel.ChatCompletion;
using Microsoft.SemanticKernel.Planning;
using StepwisePlannerMigration.Models;
using StepwisePlannerMigration.Plugins;
using StepwisePlannerMigration.Services;

#pragma warning restore IDE0005 // Using directive is unnecessary

namespace StepwisePlannerMigration.Controllers;

/// <summary>
/// This controller shows the old way how to use planning capability by using <see cref="FunctionCallingStepwisePlanner"/>.
/// A new recommended approach is demonstrated in <see cref="AutoFunctionCallingController"/>.
/// </summary>
[ApiController]
[Route("stepwise-planner")]
public class StepwisePlannerController : ControllerBase
{
    private readonly Kernel _kernel;
    private readonly FunctionCallingStepwisePlanner _planner;
    private readonly IPlanProvider _planProvider;

    public StepwisePlannerController(
        Kernel kernel,
        FunctionCallingStepwisePlanner planner,
        IPlanProvider planProvider)
    {
        this._kernel = kernel;
        this._planner = planner;
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
        FunctionCallingStepwisePlannerResult result = await this._planner.ExecuteAsync(this._kernel, request.Goal);

        return this.Ok(result.ChatHistory);
    }

    /// <summary>
    /// Action to execute a new plan.
    /// </summary>
    [HttpPost, Route("execute-new-plan")]
    public async Task<IActionResult> ExecuteNewPlanAsync(PlanRequest request)
    {
        FunctionCallingStepwisePlannerResult result = await this._planner.ExecuteAsync(this._kernel, request.Goal);

        return this.Ok(result.FinalAnswer);
    }

    /// <summary>
    /// Action to execute existing plan. Generated plans can be stored in permanent storage for reusability.
    /// In this demo application it is stored in file.
    /// </summary>
    [HttpPost, Route("execute-existing-plan")]
    public async Task<IActionResult> ExecuteExistingPlanAsync(PlanRequest request)
    {
        ChatHistory chatHistory = this._planProvider.GetPlan("stepwise-plan.json");
        FunctionCallingStepwisePlannerResult result = await this._planner.ExecuteAsync(this._kernel, request.Goal, chatHistory);

        return this.Ok(result.FinalAnswer);
    }
}
```