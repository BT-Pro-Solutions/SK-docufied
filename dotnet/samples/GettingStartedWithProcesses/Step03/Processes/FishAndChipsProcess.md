# Explanation
This C# code defines a sample process called `FishAndChipsProcess` that demonstrates how to create a composite process with fan-in/fan-out behavior by combining multiple smaller processes and steps. It uses the Microsoft Semantic Kernel framework to build an event-driven workflow that prepares fried fish and potato fries in parallel, then adds condiments, and finally emits an event signaling that the fish and chips dish is ready.

Key points:
- The process includes sub-processes for making fried fish and potato fries.
- An `AddFishAndChipsCondimentsStep` step adds condiments after both food items are prepared.
- An external step emits a final event indicating the dish is ready.
- Two variants are provided: one using stateless steps, and another using stateful steps.
- The process architecture uses event-driven transitions and function calls to coordinate the preparation steps.

```csharp
// Copyright (c) Microsoft. All rights reserved.

using System.Text.Json;
using Microsoft.SemanticKernel;
using Step03.Models;
using Step03.Steps;

namespace Step03.Processes;

/// <summary>
/// Sample process that showcases how to create a process with a fan in/fan out behavior and use of existing processes as steps.<br/>
/// Visual reference of this process can be found in the <see href="https://github.com/microsoft/semantic-kernel/blob/main/dotnet/samples/GettingStartedWithProcesses/README.md#fish-and-chips-preparation-process" >diagram</see>
/// </summary>
public static class FishAndChipsProcess
{
    public static class ProcessEvents
    {
        public const string PrepareFishAndChips = nameof(PrepareFishAndChips);
        public const string FishAndChipsReady = nameof(FishAndChipsReady);
        public const string FishAndChipsIngredientOutOfStock = nameof(FishAndChipsIngredientOutOfStock);
    }

    public static ProcessBuilder CreateProcess(string processName = "FishAndChipsProcess")
    {
        var processBuilder = new ProcessBuilder(processName);
        var makeFriedFishStep = processBuilder.AddStepFromProcess(FriedFishProcess.CreateProcess());
        var makePotatoFriesStep = processBuilder.AddStepFromProcess(PotatoFriesProcess.CreateProcess());
        var addCondimentsStep = processBuilder.AddStepFromType<AddFishAndChipsCondimentsStep>();
        // An additional step that is the only one that emits an public event in a process can be added to maintain event names unique
        var externalStep = processBuilder.AddStepFromType<ExternalFishAndChipsStep>();

        processBuilder
            .OnInputEvent(ProcessEvents.PrepareFishAndChips)
            .SendEventTo(makeFriedFishStep.WhereInputEventIs(FriedFishProcess.ProcessEvents.PrepareFriedFish))
            .SendEventTo(makePotatoFriesStep.WhereInputEventIs(PotatoFriesProcess.ProcessEvents.PreparePotatoFries));

        makeFriedFishStep
            .OnEvent(FriedFishProcess.ProcessEvents.FriedFishReady)
            .SendEventTo(new ProcessFunctionTargetBuilder(addCondimentsStep, parameterName: "fishActions"));

        makePotatoFriesStep
            .OnEvent(PotatoFriesProcess.ProcessEvents.PotatoFriesReady)
            .SendEventTo(new ProcessFunctionTargetBuilder(addCondimentsStep, parameterName: "potatoActions"));

        addCondimentsStep
            .OnEvent(AddFishAndChipsCondimentsStep.OutputEvents.CondimentsAdded)
            .SendEventTo(new ProcessFunctionTargetBuilder(externalStep));

        return processBuilder;
    }

    public static ProcessBuilder CreateProcessWithStatefulSteps(string processName = "FishAndChipsWithStatefulStepsProcess")
    {
        var processBuilder = new ProcessBuilder(processName);
        var makeFriedFishStep = processBuilder.AddStepFromProcess(FriedFishProcess.CreateProcessWithStatefulStepsV1());
        var makePotatoFriesStep = processBuilder.AddStepFromProcess(PotatoFriesProcess.CreateProcessWithStatefulSteps());
        var addCondimentsStep = processBuilder.AddStepFromType<AddFishAndChipsCondimentsStep>();
        // An additional step that is the only one that emits an public event in a process can be added to maintain event names unique
        var externalStep = processBuilder.AddStepFromType<ExternalFishAndChipsStep>();

        processBuilder
            .OnInputEvent(ProcessEvents.PrepareFishAndChips)
            .SendEventTo(makeFriedFishStep.WhereInputEventIs(FriedFishProcess.ProcessEvents.PrepareFriedFish))
            .SendEventTo(makePotatoFriesStep.WhereInputEventIs(PotatoFriesProcess.ProcessEvents.PreparePotatoFries));

        makeFriedFishStep
            .OnEvent(FriedFishProcess.ProcessEvents.FriedFishReady)
            .SendEventTo(new ProcessFunctionTargetBuilder(addCondimentsStep, parameterName: "fishActions"));

        makePotatoFriesStep
            .OnEvent(PotatoFriesProcess.ProcessEvents.PotatoFriesReady)
            .SendEventTo(new ProcessFunctionTargetBuilder(addCondimentsStep, parameterName: "potatoActions"));

        addCondimentsStep
            .OnEvent(AddFishAndChipsCondimentsStep.OutputEvents.CondimentsAdded)
            .SendEventTo(new ProcessFunctionTargetBuilder(externalStep));

        return processBuilder;
    }

    private sealed class AddFishAndChipsCondimentsStep : KernelProcessStep
    {
        public static class Functions
        {
            public const string AddCondiments = nameof(AddCondiments);
        }

        public static class OutputEvents
        {
            public const string CondimentsAdded = nameof(CondimentsAdded);
        }

        [KernelFunction(Functions.AddCondiments)]
        public async Task AddCondimentsAsync(KernelProcessStepContext context, List<string> fishActions, List<string> potatoActions)
        {
            Console.WriteLine($"ADD_CONDIMENTS: Added condiments to Fish & Chips - Fish: {JsonSerializer.Serialize(fishActions)}, Potatoes: {JsonSerializer.Serialize(potatoActions)}");
            fishActions.AddRange(potatoActions);
            fishActions.Add(FoodIngredients.Condiments.ToFriendlyString());
            await context.EmitEventAsync(new() { Id = OutputEvents.CondimentsAdded, Data = fishActions });
        }
    }

    private sealed class ExternalFishAndChipsStep : ExternalStep
    {
        public ExternalFishAndChipsStep() : base(ProcessEvents.FishAndChipsReady) { }
    }
}
```