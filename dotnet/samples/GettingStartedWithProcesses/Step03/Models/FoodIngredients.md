# Explanation
This C# code defines an enumeration `FoodIngredients` representing various food items that can be used in cooking steps like gathering ingredients, cutting food, and frying food. It includes items such as Potatoes, Fish, Buns, Sauce, Condiments, and a None option. Additionally, the code provides an extension class `FoodIngredientsExtensions` that maps each enum value to a friendlier string representation, allowing easy conversion from enum values to readable strings.

```csharp
// Copyright (c) Microsoft. All rights reserved.

namespace Step03.Models;

/// <summary>
/// Food Ingredients used in steps such GatherIngredientStep, CutFoodStep, FryFoodStep
/// </summary>
public enum FoodIngredients
{
    Pototoes,
    Fish,
    Buns,
    Sauce,
    Condiments,
    None
}

/// <summary>
/// Extensions to have access to friendly string names for <see cref="FoodIngredients"/>
/// </summary>
public static class FoodIngredientsExtensions
{
    private static readonly Dictionary<FoodIngredients, string> s_foodIngredientsStrings = new()
    {
        { FoodIngredients.Pototoes, "Potatoes" },
        { FoodIngredients.Fish, "Fish" },
        { FoodIngredients.Buns, "Buns" },
        { FoodIngredients.Sauce, "Sauce" },
        { FoodIngredients.Condiments, "Condiments" },
        { FoodIngredients.None, "None" }
    };

    public static string ToFriendlyString(this FoodIngredients ingredient)
    {
        return s_foodIngredientsStrings[ingredient];
    }
}
```