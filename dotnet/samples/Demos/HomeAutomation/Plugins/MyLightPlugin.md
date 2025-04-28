# Explanation
This C# code defines a simple plugin class `MyLightPlugin` for a home automation system using Microsoft's Semantic Kernel framework. The class models a controllable light with an internal state indicating whether it is turned on or off. It provides three kernel functions: one to check if the light is on, one to turn the light on, and one to turn the light off. The class is decorated with attributes to describe the plugin and its functions, which can aid in integration and user documentation.

```csharp
// Copyright (c) Microsoft. All rights reserved.

using System.ComponentModel;
using Microsoft.SemanticKernel;

namespace HomeAutomation.Plugins;

/// <summary>
/// Class that represents a controllable light.
/// </summary>
[Description("Represents a light")]
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