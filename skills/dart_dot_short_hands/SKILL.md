---
name: dart_dot_shorthands
description: Use Dart's dot-shorthand syntax (`.foo`) to access enum values, static members, and constructors without repeating the type name when it's inferable from context. Use when simplifying enum assignments/switches, static factory calls, or repetitive field initializers, on Dart 3.10+.
metadata:
  url: https://dart.dev/language/dot-shorthands
  last_modified: 14-07-2026
---

# dart_dot_shorthands

Instructions for the agent to follow when this skill is activated.

## When to use

Use this skill when:
- The user wants to shorten `TypeName.value` to `.value` for enum values, especially in `switch` expressions/statements and assignments.
- The user wants to call a static method/getter or a named/unnamed constructor without repeating the class name (e.g. `int.parse(...)` -> `.parse(...)`, `Point.origin()` -> `.origin()`, `AnimationController(...)` -> `.new(...)`).
- Cleaning up repetitive class field initializers where the field's declared type already makes the class obvious.
- The project's Dart SDK / language version is `3.10` or later (dot shorthands require this minimum). If lower, warn the user before applying this skill.

## Instructions

1. **Check the language version.** Dot shorthands require language version 3.10+. Verify `pubspec.yaml`'s `environment.sdk` constraint before applying.

2. **Only shorten when the context type is unambiguous.** The compiler infers the target type from the surrounding context (variable's declared type, function return type, switch scrutinee type, argument's parameter type, etc.). Do not use `.foo` where the expected type can't be statically determined.

3. **Common places to apply this shorthand:**
   - Enum value assignments: `Status s = .running;` instead of `Status s = Status.running;`
   - `switch` cases over enums: `.debug => 'gray'` instead of `LogLevel.debug => 'gray'`
   - Static factory/parse calls: `int.parse('8080')` -> `.parse('8080')` when assigned to an `int`-typed target.
   - Named/factory constructors: `Point.origin()` -> `.origin()` when the target type is `Point`.
   - Unnamed constructors via `.new`: `ScrollController()` -> `.new()` for field initializers with an explicit declared type.
   - `const` contexts: enum values and `const` constructors both work with dot shorthand, e.g. `const Status defaultStatus = .running;`.

4. **Chaining rule.** You can chain method/property calls after a dot shorthand (e.g. `.fromCharCode(72).toLowerCase()`), but the *initial* dot shorthand is resolved against the context type — only the first segment gets the inferred type; subsequent chained calls resolve normally against whatever type comes back.

5. **Equality checks are asymmetric.** For `==`/`!=`, only put the dot shorthand on the **right-hand side** — the left-hand side's static type provides the context. Never write `.red == myColor`; always write `myColor == .red`. Also do not use dot shorthand against a complex expression (e.g. a ternary) on the right of `==`.

6. **Never start an expression statement with `.`.** `.log('Hello');` as a standalone statement is a compile error — there's no context type to infer from. Only use dot shorthand where a context type exists (assignment, argument, return, case, etc.).

7. **Nullable and `FutureOr` targets have limited support:**
   - For `T?`, you can access static members of `T`, not of `Null`.
   - For `FutureOr<T>`, you can access static members of `T`, not of `Future`.

8. **After applying**, run `dart analyze` (or the `dart-run-static-analysis` skill) to confirm no ambiguous-context or asymmetric-equality errors were introduced.

## Examples

### Enums
```dart
enum Status { none, running, stopped, paused }

Status currentStatus = .running; // Instead of Status.running

enum LogLevel { debug, info, warning, error }

String colorCode(LogLevel level) => switch (level) {
  .debug => 'gray',
  .info => 'blue',
  .warning => 'orange',
  .error => 'red',
};
```

### Static members
```dart
int port = .parse('8080');       // Instead of int.parse('8080')
BigInt bigIntZero = .zero;       // Instead of BigInt.zero
```

### Named, factory, and generic constructors
```dart
class Point {
  final double x, y;
  const Point(this.x, this.y);
  const Point.origin() : x = 0, y = 0;
  factory Point.fromList(List<double> list) => Point(list[0], list[1]);
}

Point origin = .origin();               // Instead of Point.origin()
Point p1 = .fromList([1.0, 2.0]);        // Instead of Point.fromList([1.0, 2.0])
List<int> intList = .filled(5, 0);       // Instead of List.filled(5, 0)
```

### Unnamed constructors (`.new`) for field initializers
```dart
class _PageState extends State<Page> {
  late final AnimationController _animationController = .new(vsync: this);
  final ScrollController _scrollController = .new();
  final GlobalKey<ScaffoldMessengerState> scaffoldKey = .new();
  Map<String, Map<String, bool>> properties = .new();
}
```

### Constant expressions
```dart
const Status defaultStatus = .running;
const Point myOrigin = .origin();
const List<Point> keyPoints = [.origin(), .new(1.0, 1.0)];
```

### Asymmetric equality — correct vs incorrect
```dart
enum Color { red, green, blue }

Color myColor = Color.red;

// Correct: dot shorthand on the right-hand side.
if (myColor == .green) { /* ... */ }
if (myColor != .blue) { /* ... */ }
Color inferredColor = true ? .green : .blue; // OK: context is the assignment target.

// Incorrect: will not compile.
// if (.red == myColor) { }
// if (myColor == (true ? .green : .blue)) { }
// if ((myColor as Object) == .green) { }
```
