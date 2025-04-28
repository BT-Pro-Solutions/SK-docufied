# Explanation
This C# code defines a record type `MarketingNewEntryDetails` within the `Step02.Models` namespace. The record is designed to hold information for a new entry in a marketing database, including an account identifier (`AccountId`), a contact name (`Name`), a phone number (`PhoneNumber`), and an email address (`Email`). It is used in samples named `Step02a_AccountOpening` and `Step02b_AccountOpening`.

```csharp
// Copyright (c) Microsoft. All rights reserved.

namespace Step02.Models;

/// <summary>
/// Holds details for a new entry in a marketing database, including the account identifier, contact name, phone number, and email address.<br/>
/// Class used in <see cref="Step02a_AccountOpening"/>, <see cref="Step02b_AccountOpening"/> samples
/// </summary>
public record MarketingNewEntryDetails
{
    public Guid AccountId { get; set; }

    public string Name { get; set; }

    public string PhoneNumber { get; set; }

    public string Email { get; set; }
}
```