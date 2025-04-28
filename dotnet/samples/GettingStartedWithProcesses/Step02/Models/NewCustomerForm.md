# Explanation
This C# class defines a `NewCustomerForm` model representing the data structure used to capture details of a new customer. It includes properties for personal information such as first and last names, date of birth, state, phone number, user ID, and email address. Each property is decorated with `JsonPropertyName` attributes to specify their JSON serialization names. The class also provides methods to create a copy of the form with default values for any empty fields (`CopyWithDefaultValues`) and to check whether all required form fields have been completed (`IsFormCompleted`). This class is used in samples named `Step02a_AccountOpening` and `Step02b_AccountOpening`.

```csharp
// Copyright (c) Microsoft. All rights reserved.

using System.Reflection;
using System.Text.Json.Serialization;

namespace Step02.Models;

/// <summary>
/// Represents the data structure for a form capturing details of a new customer, including personal information and contact details.<br/>
/// Class used in <see cref="Step02a_AccountOpening"/>, <see cref="Step02b_AccountOpening"/> samples
/// </summary>
public class NewCustomerForm
{
    [JsonPropertyName("userFirstName")]
    public string UserFirstName { get; set; } = string.Empty;

    [JsonPropertyName("userLastName")]
    public string UserLastName { get; set; } = string.Empty;

    [JsonPropertyName("userDateOfBirth")]
    public string UserDateOfBirth { get; set; } = string.Empty;

    [JsonPropertyName("userState")]
    public string UserState { get; set; } = string.Empty;

    [JsonPropertyName("userPhoneNumber")]
    public string UserPhoneNumber { get; set; } = string.Empty;

    [JsonPropertyName("userId")]
    public string UserId { get; set; } = string.Empty;

    [JsonPropertyName("userEmail")]
    public string UserEmail { get; set; } = string.Empty;

    public NewCustomerForm CopyWithDefaultValues(string defaultStringValue = "Unanswered")
    {
        NewCustomerForm copy = new();
        PropertyInfo[] properties = typeof(NewCustomerForm).GetProperties();

        foreach (PropertyInfo property in properties)
        {
            // Get the value of the property  
            string? value = property.GetValue(this) as string;

            // Check if the value is an empty string  
            if (string.IsNullOrEmpty(value))
            {
                property.SetValue(copy, defaultStringValue);
            }
            else
            {
                property.SetValue(copy, value);
            }
        }

        return copy;
    }

    public bool IsFormCompleted()
    {
        return !string.IsNullOrEmpty(UserFirstName) &&
            !string.IsNullOrEmpty(UserLastName) &&
            !string.IsNullOrEmpty(UserId) &&
            !string.IsNullOrEmpty(UserDateOfBirth) &&
            !string.IsNullOrEmpty(UserState) &&
            !string.IsNullOrEmpty(UserEmail) &&
            !string.IsNullOrEmpty(UserPhoneNumber);
    }
}
```