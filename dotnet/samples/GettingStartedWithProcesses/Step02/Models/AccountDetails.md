# Explanation
This C# code defines a data model for capturing details of a new customer's account in a banking or financial application. The `AccountDetails` class inherits from `NewCustomerForm` and includes properties for a unique account identifier (`AccountId`) and the type of account (`AccountType`). The `AccountType` enumeration lists possible account categories such as `PrimeABC` and `Other`. This class is intended to be used in the `Step02a_AccountOpening` and `Step02b_AccountOpening` samples, providing a structured way to manage new account data.

```csharp
// Copyright (c) Microsoft. All rights reserved.

namespace Step02.Models;

/// <summary>
/// Represents the data structure for a form capturing details of a new customer, including personal information, contact details, account id and account type.<br/>
/// Class used in <see cref="Step02a_AccountOpening"/>, <see cref="Step02b_AccountOpening"/> samples
/// </summary>
public class AccountDetails : NewCustomerForm
{
    public Guid AccountId { get; set; }
    public AccountType AccountType { get; set; }
}

public enum AccountType
{
    PrimeABC,
    Other,
}
```