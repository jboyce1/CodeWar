# Essential C# Concepts for OpenRA Trait Modifications

This guide provides an in-depth understanding of the essential C# concepts needed to modify traits in OpenRA. 

## **1. Variables and Types**
### **Explanation**
Variables store data that can be used and manipulated in the code. Understanding data types (`int`, `bool`, `string`, etc.) is critical for defining traits, tracking states, and customizing behaviors.

### **Examples**
```csharp
// AmmoPool.cs
public readonly int Ammo = 1; // Defines the total ammunition for an actor.
public readonly string Name = "primary";
-Specifies the name of the ammo pool.

// Parachutable.cs
public readonly bool KilledOnImpassableTerrain = true;
-Determines if the actor dies on invalid terrain.

// AutoTarget.cs
public readonly int MinimumScanTimeInterval = 3;
-Sets the minimum interval between scan actions.
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
        return; // Prevents redundant changes if the stance is already set.

    var oldStance = Stance;
    Stance = value; // Updates the unit's stance.
    ApplyStanceCondition(self); // Applies any conditions based on the new stance.
}

// Cargo.cs
public bool CanUnload(BlockedByActor check = BlockedByActor.None)
{
    return !IsEmpty() && CurrentAdjacentCells.Any(); // Checks if unloading is possible based on conditions.
}

// Armament.cs
protected virtual void FireBarrel(Actor self, IFacing facing, in Target target, Barrel barrel)
{
    foreach (var na in notifyAttacks)
        na.PreparingAttack(self, target, this, barrel); // Prepares the attack logic for the weapon's barrel.
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
    public readonly bool KilledOnImpassableTerrain = true; // Determines if the actor dies on invalid terrain.
    public override object Create(ActorInitializer init)
    {
        return new Parachutable(init.Self, this); // Creates an instance of the Parachutable trait.
    }
}

// AmmoPool.cs
public class AmmoPoolInfo : TraitInfo
{
    public readonly int Ammo = 1; // Specifies the maximum ammunition in the pool.
    public override object Create(ActorInitializer init)
    {
        return new AmmoPool(this); // Creates an instance of the AmmoPool trait.
    }
}

// Cargo.cs
public class CargoInfo : TraitInfo
{
    public readonly int MaxWeight = 0; // Specifies the maximum weight capacity for cargo.
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
public readonly WeaponInfo Weapon; // Links to the weapon's data.
public int FireDelay { get; protected set; } // Tracks the cooldown time before the weapon can fire again.

// AmmoPool.cs
public int CurrentAmmoCount { get; private set; } // Tracks the current ammo count.

// Cargo.cs
readonly List<Actor> cargo = new(); // Stores the list of actors currently in the cargo.
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
public UnitStance Stance { get; private set; } // Gets or privately sets the unit's current stance.
public bool AllowMove => allowMovement && Stance > UnitStance.Defend; // Determines if movement is allowed.

// Armament.cs
public virtual WDist MaxRange()
{
    return new WDist(Util.ApplyPercentageModifiers(Weapon.Range.Length, rangeModifiers.ToArray())); // Calculates the maximum range with modifiers applied.
}

// AmmoPool.cs
public bool HasFullAmmo => CurrentAmmoCount == Info.Ammo; // Checks if the ammo pool is full.
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
    return false; // Ensures ammo can't be used if there is none or the count is invalid.

// AutoTarget.cs
if (IsTraitDisabled || !Info.ScanOnIdle || Stance < UnitStance.Defend)
    return; // Skips scanning if the trait is disabled or other conditions aren't met.

// Cargo.cs
if (reservedWeight != 0)
    return; // Ensures the transport unit does not unload if weight is reserved.
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
readonly List<Actor> cargo = new(); // Stores the actors loaded as cargo.

// AutoTarget.cs
public readonly IEnumerable<AttackBase> ActiveAttackBases; // Holds a collection of attack bases currently active for an actor.

// Armament.cs
var barrels = new List<Barrel>(); // Defines a list of weapon barrels for the armament.
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
public readonly Dictionary<string, string> PassengerConditions = new(); // Maps passenger names to their granted conditions.

// AutoTarget.cs
[FieldLoader.Ignore]
public readonly Dictionary<UnitStance, string> ConditionByStance = new(); // Maps unit stances to their corresponding conditions.

// Parachutable.cs
public readonly HashSet<string> WaterTerrainTypes = new() { "Water" }; // Defines valid terrain types for water corpses.
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
ModifiedRange = new WDist(Util.ApplyPercentageModifiers(Weapon.Range.Length, rangeModifiers.ToArray())); // Converts range modifiers to a new distance value.

// AmmoPool.cs
CurrentAmmoCount = (CurrentAmmoCount + count).Clamp(0, Info.Ammo); // Ensures ammo count stays within valid bounds.

// AutoTarget.cs
nextScanTime = self.World.SharedRandom.Next(Info.MinimumScanTimeInterval, Info.MaximumScanTimeInterval); // Converts random values into scan timing intervals.
```
### **Why Important**
- **Interoperability**: Ensures different types work together seamlessly.
- **Data Manipulation**: Allows transformations like clamping ammo values.
- **Precision**: Avoids errors caused by mismatched types.
