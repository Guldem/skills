---
name: dart_enhanced_enums
description: Declare and use Dart enhanced enums — enums with fields, constructors, methods, getters, and interfaces. Use when a fixed set of constant values needs associated data or behavior (e.g. replacing a class hierarchy of singletons, or adding properties to a simple enum).
metadata:
  url: https://dart.dev/language/enums
  last_modified: 14-07-2026
---

# dart_enhanced_enums

Instructions for the agent to follow when this skill is activated.

## When to use

Use this skill when:
- The user wants an `enum` to carry data (fields) alongside each constant value.
- The user wants to add methods, getters, or `const` constructors to an enum.
- The user wants an enum to implement an interface or `Comparable`.
- Converting a "simple" enum (`enum Color { red, green, blue }`) into one with associated properties.
- The user's Dart SDK / language version is `2.17` or later (enhanced enums require this minimum).

## Instructions

1. **Check the language version.** Enhanced enums require language version 2.17+. If the project's `pubspec.yaml` `environment.sdk` constraint is lower, warn the user before proceeding.

2. **Declare instances first.** All enum instances must be listed at the very beginning of the declaration, separated by commas, and terminated with a semicolon. There must be at least one instance.

3. **Add a `const` constructor.** All generative constructors in an enhanced enum must be `const`. Use named parameters for clarity when there are multiple fields.

4. **Use `final` for all instance fields**, including any added via mixins. Non-final fields are not allowed.

5. **Add methods/getters as needed.** Instance methods and getters can reference `this` to access the current enum value's fields.

6. **Avoid the following (compile errors):**
   - Overriding `index`, `hashCode`, or `==`.
   - Declaring a member named `values` (conflicts with the auto-generated static `values` getter).
   - Extending any class other than the implicit `Enum` (an enum cannot `extends` anything).
   - Non-const generative constructors.
   - Factory constructors that don't return one of the fixed enum instances.

7. **Interfaces are allowed.** An enhanced enum can `implements` one or more interfaces (e.g. `Comparable<T>`) and override their members, since it does not need to extend another class.

8. **Prefer primary constructors (Dart 3.13+)** for a more concise declaration when the SDK supports it — see the alternate example below. Otherwise use a standard named-parameter `const` constructor.

9. **After writing the enum**, run `dart analyze` (or use the `dart-run-static-analysis` skill) to catch any of the restricted patterns above.

## Examples

### Simple enum (no fields)
```dart
enum Color { red, green, blue }
```

### Enhanced enum with fields, getters, and an implemented interface
```dart
enum Vehicle implements Comparable<Vehicle> {
  car(tires: 4, passengers: 5, carbonPerKilometer: 400),
  bus(tires: 6, passengers: 50, carbonPerKilometer: 800),
  bicycle(tires: 2, passengers: 1, carbonPerKilometer: 0);

  const Vehicle({
    required this.tires,
    required this.passengers,
    required this.carbonPerKilometer,
  });

  final int tires;
  final int passengers;
  final int carbonPerKilometer;

  int get carbonFootprint => (carbonPerKilometer / passengers).round();

  bool get isTwoWheeled => this == Vehicle.bicycle;

  @override
  int compareTo(Vehicle other) => carbonFootprint - other.carbonFootprint;
}
```

### Same enum using a primary constructor (Dart 3.13+)
```dart
enum Vehicle(
  final int tires,
  final int passengers,
  final int carbonPerKilometer,
) implements Comparable<Vehicle> {
  car(4, 5, 400),
  bus(6, 50, 800),
  bicycle(2, 1, 0);

  int get carbonFootprint => (carbonPerKilometer / passengers).round();

  bool get isTwoWheeled => this == Vehicle.bicycle;

  @override
  int compareTo(Vehicle other) => carbonFootprint - other.carbonFootprint;
}
```

### Using enums
```dart
// Access like a static variable.
final favoriteColor = Color.blue;
if (favoriteColor == Color.blue) {
  print('Your favorite color is blue!');
}

// index: zero-based position in the declaration.
assert(Color.red.index == 0);
assert(Color.green.index == 1);
assert(Color.blue.index == 2);

// values: list of all enumerated values.
List<Color> colors = Color.values;
assert(colors[2] == Color.blue);

// name: the identifier's string name.
print(Color.blue.name); // 'blue'

// Switch statements warn if not exhaustive (no `default`).
switch (favoriteColor) {
  case Color.red:
    print('Red as roses!');
  case Color.green:
    print('Green as grass!');
  case Color.blue:
    print('Blue as the sky!');
}

// Access members like a normal object.
print(Vehicle.car.carbonFootprint);
```
