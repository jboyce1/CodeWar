---
layout: default
title: CS Essential Skills
---

# Essential C# Concepts for OpenRA Trait Modifications

This guide provides an in-depth understanding of the essential C# concepts needed to modify traits in OpenRA. Each concept is explained with multiple examples taken directly from OpenRA's `.cs` files, along with robust explanations of their importance.
```

## **1. Variables and Types**
### **Explanation**
Variables store data that can be used and manipulated in the code. Understanding data types (`int`, `bool`, `string`, etc.) is critical for defining traits, tracking states, and customizing behaviors.

### **Examples**
```csharp
// AmmoPool.cs
public readonly int Ammo = 1;
public readonly string Name = "primary";

// Parachutable.cs
public readonly bool KilledOnImpassableTerrain = true;

// AutoTarget.cs
public readonly int MinimumScanTimeInterval = 3;
```
### **Why Important**
- **Customization**: Variables like `Ammo` allow developers to control gameplay elements such as how much ammo a unit has.
- **Dynamic Behavior**: Variables like `KilledOnImpassableTerrain` determine conditional actions based on game state.
- **Efficiency**: Proper use of types ensures memory efficiency and reduces bugs.

---

## **2. Methods**
### **Explanation**
Methods define the actions or logic of a class. They are used to perform specific tasks, interact with other objects, and modify data.

### **Examples**
```csharp
// AutoTarget.cs
public void SetStance(Actor self, UnitStance value)
{
    if (Stance == value)
        return;

    var oldStance = Stance;
    Stance = value;
    ApplyStanceCondition(self);
}

// Cargo.cs
public bool CanUnload(BlockedByActor check = BlockedByActor.None)
{
    return !IsEmpty() && CurrentAdjacentCells.Any();
}

// Armament.cs
protected virtual void FireBarrel(Actor self, IFacing facing, in Target target, Barrel barrel)
{
    foreach (var na in notifyAttacks)
        na.PreparingAttack(self, target, this, barrel);
}
```
### **Why Important**
- **Logic and Behavior**: Methods like `SetStance` define how traits behave.
- **Reusability**: They encapsulate logic that can be reused throughout the codebase.
- **Scalability**: Methods allow for easy extension of functionality by overriding or adding new ones.

---

## **3. Basic Classes**
### **Explanation**
Classes are the blueprint for objects in object-oriented programming. In OpenRA, traits are implemented as classes that define specific behaviors for units or actors.

### **Examples**
```csharp
// Parachutable.cs
public class ParachutableInfo : TraitInfo
{
    public readonly bool KilledOnImpassableTerrain = true;
    public override object Create(ActorInitializer init)
    {
        return new Parachutable(init.Self, this);
    }
}

// AmmoPool.cs
public class AmmoPoolInfo : TraitInfo
{
    public readonly int Ammo = 1;
    public override object Create(ActorInitializer init)
    {
        return new AmmoPool(this);
    }
}

// Cargo.cs
public class CargoInfo : TraitInfo
{
    public readonly int MaxWeight = 0;
}
```
### **Why Important**
- **Modularity**: Classes encapsulate related data and methods, making code easier to manage.
- **Trait Definitions**: Every trait in OpenRA is represented by a class.
- **Extensibility**: New behaviors can be added by creating new classes.

---

## **4. Class Variables**
### **Explanation**
Class variables store the state of a class and define its behavior. These are often `readonly` for constant values or mutable for dynamic states.

### **Examples**
```csharp
// Armament.cs
public readonly WeaponInfo Weapon;
public int FireDelay { get; protected set; }

// AmmoPool.cs
public int CurrentAmmoCount { get; private set; }

// Cargo.cs
readonly List<Actor> cargo = new();
```
### **Why Important**
- **State Management**: Variables like `FireDelay` control behavior (e.g., weapon cooldowns).
- **Encapsulation**: Variables like `cargo` store a unit's passengers, encapsulating the transport's state.
- **Control**: Using `readonly` ensures important data cannot be accidentally modified.

---

## **5. Class Properties**
### **Explanation**
Properties provide controlled access to class variables, often using logic to manage how data is retrieved or modified.

### **Examples**
```csharp
// AutoTarget.cs
public UnitStance Stance { get; private set; }
public bool AllowMove => allowMovement && Stance > UnitStance.Defend;

// Armament.cs
public virtual WDist MaxRange()
{
    return new WDist(Util.ApplyPercentageModifiers(Weapon.Range.Length, rangeModifiers.ToArray()));
}

// AmmoPool.cs
public bool HasFullAmmo => CurrentAmmoCount == Info.Ammo;
```
### **Why Important**
- **Encapsulation**: Properties like `Stance` control how the stance variable is accessed and updated.
- **Derived Values**: Properties like `HasFullAmmo` compute values dynamically based on internal state.
- **Flexibility**: They allow adding additional logic without exposing implementation details.

---

## **6. Conditionals**
### **Explanation**
Conditionals are used to control the flow of logic, determining how code executes under specific conditions.

### **Examples**
```csharp
// AmmoPool.cs
if (CurrentAmmoCount <= 0 || count < 0)
    return false;

// AutoTarget.cs
if (IsTraitDisabled || !Info.ScanOnIdle || Stance < UnitStance.Defend)
    return;

// Cargo.cs
if (reservedWeight != 0)
    return;
```
### **Why Important**
- **Behavioral Logic**: Determines how traits react to changes in game state.
- **Validation**: Ensures that actions only occur when conditions are met (e.g., ammo usage).
- **Error Prevention**: Prevents invalid states or actions in the game.

---

## **7. Lists**
### **Explanation**
Lists store collections of items that can be dynamically modified. They are heavily used for managing units, weapons, and behaviors.

### **Examples**
```csharp
// Cargo.cs
readonly List<Actor> cargo = new();

// AutoTarget.cs
public readonly IEnumerable<AttackBase> ActiveAttackBases;

// Armament.cs
var barrels = new List<Barrel>();
```
### **Why Important**
- **Dynamic Storage**: Lists allow flexible storage of items like passengers, weapons, or traits.
- **Iteration**: Enables easy traversal and modification of elements.
- **Utility**: Commonly used for grouping related objects in OpenRA.

---

## **8. Dictionaries**
### **Explanation**
Dictionaries store key-value pairs for quick lookups, enabling complex mappings such as conditions or traits.

### **Examples**
```csharp
// Cargo.cs
public readonly Dictionary<string, string> PassengerConditions = new();

// AutoTarget.cs
[FieldLoader.Ignore]
public readonly Dictionary<UnitStance, string> ConditionByStance = new();

// Parachutable.cs
public readonly HashSet<string> WaterTerrainTypes = new() { "Water" };
```
### **Why Important**
- **Mappings**: Quickly map keys to values, such as unit types to conditions.
- **Customization**: Allows highly configurable and extensible traits.
- **Efficiency**: Provides O(1) access time for lookups.

---

## **9. Type Conversion**
### **Explanation**
Type conversion is essential when working with diverse data types, ensuring compatibility between operations.

### **Examples**
```csharp
// Armament.cs
ModifiedRange = new WDist(Util.ApplyPercentageModifiers(Weapon.Range.Length, rangeModifiers.ToArray()));

// AmmoPool.cs
CurrentAmmoCount = (CurrentAmmoCount + count).Clamp(0, Info.Ammo);

// AutoTarget.cs
nextScanTime = self.World.SharedRandom.Next(Info.MinimumScanTimeInterval, Info.MaximumScanTimeInterval);
```
### **Why Important**
- **Interoperability**: Ensures different types work together seamlessly.
- **Data Manipulation**: Allows transformations like clamping ammo values.
- **Precision**: Avoids errors caused by mismatched types.
