# Explanation

The `ScriptedUserInputStep` class is designed for use within Microsoft's Semantic Kernel process orchestration framework. It provides a reusable process step that simulates scripted user input during process execution. This step manages a list of pre-defined user messages and emits them sequentially when invoked. It's especially useful for automating flows that typically require interactive user input, such as demos or sample processes. The class is intended to be extended or overridden, allowing developers to customize how user messages are supplied or events are emitted. The associated `UserInputState` record holds the list of user inputs and manages the current position within that list.

Key features:
- Manages scripted user input within process steps.
- Emits events to indicate user input received or process exit.
- Provides a virtual method (`PopulateUserInputs`) for populating user messages.
- Intended for use in sample processes like account opening or agent orchestration.

```csharp
// Copyright (c) Microsoft. All rights reserved.

using Events;
using Microsoft.SemanticKernel;

namespace SharedSteps;

/// <summary>
/// A step that elicits user input.
///
/// Step used in the Processes Samples:
/// - Step01_Processes.cs
/// - Step02_AccountOpening.cs
/// - Step04_AgentOrchestration
/// </summary>
public class ScriptedUserInputStep : KernelProcessStep<UserInputState>
{
    public static class Functions
    {
        public const string GetUserInput = nameof(GetUserInput);
    }

    protected bool SuppressOutput { get; init; }

    /// <summary>
    /// The state object for the user input step. This object holds the user inputs and the current input index.
    /// </summary>
    private UserInputState? _state;

    /// <summary>
    /// Method to be overridden by the user to populate with custom user messages
    /// </summary>
    /// <param name="state">The initialized state object for the step.</param>
    public virtual void PopulateUserInputs(UserInputState state)
    {
        return;
    }

    /// <summary>
    /// Activates the user input step by initializing the state object. This method is called when the process is started
    /// and before any of the KernelFunctions are invoked.
    /// </summary>
    /// <param name="state">The state object for the step.</param>
    /// <returns>A <see cref="ValueTask"/></returns>
    public override ValueTask ActivateAsync(KernelProcessStepState<UserInputState> state)
    {
        _state = state.State;

        PopulateUserInputs(_state!);

        return ValueTask.CompletedTask;
    }

    internal string GetNextUserMessage()
    {
        if (_state != null && _state.CurrentInputIndex >= 0 && _state.CurrentInputIndex < this._state.UserInputs.Count)
        {
            var userMessage = this._state!.UserInputs[_state.CurrentInputIndex];
            _state.CurrentInputIndex++;

            Console.ForegroundColor = ConsoleColor.Yellow;
            Console.WriteLine($"USER: {userMessage}");
            Console.ResetColor();

            return userMessage;
        }

        Console.WriteLine("SCRIPTED_USER_INPUT: No more scripted user messages defined, returning empty string as user message");
        return string.Empty;
    }

    /// <summary>
    /// Gets the user input.
    /// Could be overridden to customize the output events to be emitted
    /// </summary>
    /// <param name="context">An instance of <see cref="KernelProcessStepContext"/> which can be
    /// used to emit events from within a KernelFunction.</param>
    /// <returns>A <see cref="ValueTask"/></returns>
    [KernelFunction(Functions.GetUserInput)]
    public virtual async ValueTask GetUserInputAsync(KernelProcessStepContext context)
    {
        var userMessage = this.GetNextUserMessage();
        // Emit the user input
        if (string.IsNullOrEmpty(userMessage))
        {
            await context.EmitEventAsync(new() { Id = CommonEvents.Exit });
            return;
        }

        await context.EmitEventAsync(new() { Id = CommonEvents.UserInputReceived, Data = userMessage });
    }
}

/// <summary>
/// The state object for the <see cref="ScriptedUserInputStep"/>
/// </summary>
public record UserInputState
{
    public List<string> UserInputs { get; init; } = [];

    public int CurrentInputIndex { get; set; } = 0;
}
```