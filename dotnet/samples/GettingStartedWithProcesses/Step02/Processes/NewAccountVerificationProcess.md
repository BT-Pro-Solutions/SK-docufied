# Explanation
This C# code defines a process called `NewAccountVerificationProcess` using a pipeline-style approach with the Semantic Kernel framework. It demonstrates creating a `KernelProcess` composed of multiple verification steps that are triggered based on specific events related to new account opening. The process includes steps for checking a customer's credit score (`CreditScoreCheckStep`) and performing fraud detection (`FraudDetectionStep`). The interactions between these steps are coordinated such that when a new customer form is completed, both credit score checking and fraud detection are triggered. Moreover, if the credit score check results in approval, it triggers the fraud detection step again to continue validation. The code references a visual diagram for this process flow in the Semantic Kernel GitHub repository.

```csharp
// Copyright (c) Microsoft. All rights reserved.

using Microsoft.SemanticKernel;
using Step02.Models;
using Step02.Steps;

namespace Step02.Processes;

/// <summary>
/// Demonstrate creation of <see cref="KernelProcess"/> and
/// eliciting its response to five explicit user messages.<br/>
/// For each test there is a different set of user messages that will cause different steps to be triggered using the same pipeline.<br/>
/// For visual reference of the process check the <see href="https://github.com/microsoft/semantic-kernel/tree/main/dotnet/samples/GettingStartedWithProcesses/README.md#step02b_accountOpening" >diagram</see>.
/// </summary>
public static class NewAccountVerificationProcess
{
    public static ProcessBuilder CreateProcess()
    {
        ProcessBuilder process = new("AccountVerificationProcess");

        var customerCreditCheckStep = process.AddStepFromType<CreditScoreCheckStep>();
        var fraudDetectionCheckStep = process.AddStepFromType<FraudDetectionStep>();

        // When the newCustomerForm is completed...
        process
            .OnInputEvent(AccountOpeningEvents.NewCustomerFormCompleted)
            // The information gets passed to the core system record creation step
            .SendEventTo(new ProcessFunctionTargetBuilder(customerCreditCheckStep, functionName: CreditScoreCheckStep.Functions.DetermineCreditScore, parameterName: "customerDetails"))
            // The information gets passed to the fraud detection step for validation
            .SendEventTo(new ProcessFunctionTargetBuilder(fraudDetectionCheckStep, functionName: FraudDetectionStep.Functions.FraudDetectionCheck, parameterName: "customerDetails"));

        // When the creditScoreCheck step results in Approval, the information gets to the fraudDetection step to kickstart this step
        customerCreditCheckStep
            .OnEvent(AccountOpeningEvents.CreditScoreCheckApproved)
            .SendEventTo(new ProcessFunctionTargetBuilder(fraudDetectionCheckStep, functionName: FraudDetectionStep.Functions.FraudDetectionCheck, parameterName: "previousCheckSucceeded"));

        return process;
    }
}
```