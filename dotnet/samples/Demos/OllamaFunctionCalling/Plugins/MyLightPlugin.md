# Explanation
This C# class, `MyLightPlugin`, represents a controllable light bulb that can be turned on or off. It is designed for use as a plugin, specifically for integration with the Microsoft Semantic Kernel framework (as indicated by the `KernelFunction` attributes). The class exposes:
- A method to check whether the light is currently on (`IsTurnedOn`).
- Methods to turn the light on (`TurnOn`) or off (`TurnOff`).
The initial state can be set through the constructor. All API functions are decorated with appropriate descriptions for discoverability and automated documentation.

```csharp
// Copyright (c) Microsoft. All rights reserved.

using System.ComponentModel;
using Microsoft.SemanticKernel;

namespace OllamaFunctionCalling;

/// <summary>
/// Class that represents a controllable light bulb.
/// </summary>
[Description("Represents a light bulb")]
public class MyLightPlugin(bool turnedOn = false)
{
    private bool _turnedOn = turnedOn;

    [KernelFunction, Description("Returns whether this light is on")]
    public bool IsTurnedOn() => _turnedOn;

    [KernelFunction, Description("Turn on this light")]
    public void TurnOn() => _turnedOn = true;

    [KernelFunction, Description("Turn off this light")]
    public void TurnOff() => _turnedOn = false;
}
```