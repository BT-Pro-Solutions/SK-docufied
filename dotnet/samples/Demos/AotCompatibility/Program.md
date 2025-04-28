# Explanation

This C# code file, `Program.cs`, demonstrates how to use the Semantic Kernel within Ahead-Of-Time (AOT) compiled applications. The main functionality includes running a set of sample kernel functions and plugin operations, including ONNX chat completion services, by invoking corresponding static methods. These samples help illustrate creating kernel functions, importing and adding plugins, and using AI services in an AOT context. The application utilizes user secrets for configuration, runs the samples asynchronously, and returns a status code based on the execution success of all samples.

```csharp
// Copyright (c) Microsoft. All rights reserved.

using Microsoft.Extensions.Configuration;

namespace SemanticKernel.AotCompatibility;

/// <summary>
/// This application demonstrates how to use the Semantic Kernel in AOT applications.
/// </summary>
internal sealed class Program
{
    private static async Task<int> Main(string[] args)
    {
        var config = new ConfigurationBuilder().AddUserSecrets<Program>().Build();

        bool success = await RunAsync(s_samples, config);

        return success ? 1 : 0;
    }

    private static readonly Func<IConfigurationRoot, Task>[] s_samples =
    [
        // Samples showing how to create a kernel function and invoke it in AOT applications.
        KernelFunctionSamples.CreateFunctionFromLambda,

        // Samples showing how to create, import and add a kernel plugin and invoke its functions in AOT applications.
        KernelPluginSamples.CreatePluginFromType,
        KernelPluginSamples.ImportPluginFromType,
        KernelPluginSamples.AddPluginFromType,

        // Samples showing how to use ONNX chat completion service in AOT applications.
        OnnxChatCompletionSamples.GetChatMessageContent,
        OnnxChatCompletionSamples.GetStreamingChatMessageContents
    ];

    private static async Task<bool> RunAsync(IEnumerable<Func<IConfigurationRoot, Task>> functionsToRun, IConfigurationRoot config)
    {
        bool failed = false;

        foreach (var function in functionsToRun)
        {
            Console.Write($"Running - {function.Method.DeclaringType?.Name}.{function.Method.Name}");

            try
            {
                await function(config);
            }
            catch (Exception)
            {
                failed = true;
            }
        }

        return failed;
    }
}
```