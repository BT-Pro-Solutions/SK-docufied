# Explanation
This code defines an external step class `ExternalStep` used within a semantic kernel process. It is intended to be used in process samples related to food preparation. The step emits an external event asynchronously when executed, sending out an event with a specified name, data, and public visibility.

```csharp
// Copyright (c) Microsoft. All rights reserved.

using Microsoft.SemanticKernel;

namespace Step03.Steps;

/// <summary>
/// Step used in the Processes Samples:
/// - Step_03_FoodPreparation.cs
/// </summary>
public class ExternalStep(string externalEventName) : KernelProcessStep
{
    private readonly string _externalEventName = externalEventName;

    [KernelFunction]
    public async Task EmitExternalEventAsync(KernelProcessStepContext context, object data)
    {
        await context.EmitEventAsync(new() { Id = this._externalEventName, Data = data, Visibility = KernelProcessEventVisibility.Public });
    }
}
```