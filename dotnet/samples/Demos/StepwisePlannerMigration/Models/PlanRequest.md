# Explanation
This C# code defines a simple request model named `PlanRequest` within the `StepwisePlannerMigration.Models` namespace. The class is intended for use with planning endpoints and contains a single property, `Goal`, which represents the objective to be achieved after the execution of a plan. The property is a string with public getter and setter, and XML documentation comments are included for clarity.

```csharp
// Copyright (c) Microsoft. All rights reserved.

namespace StepwisePlannerMigration.Models;

/// <summary>
/// Request model for planning endpoints.
/// </summary>
public class PlanRequest
{
    /// <summary>
    /// A goal which should be achieved after plan execution.
    /// </summary>
    public string Goal { get; set; }
}
```