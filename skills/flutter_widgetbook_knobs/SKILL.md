---
name: flutter_widgetbook_knobs
description: Add Widgetbook knobs (context.knobs.*) to a widgetbook use case so widget properties (bool, int, double, String, Duration, DateTime, Color, iterables, enums/objects) can be tweaked live in the Widgetbook UI. Use when writing or updating a @widgetbook.UseCase and the widget under test needs configurable/interactive parameters instead of hard-coded values.
metadata:
  url: https://docs.widgetbook.io/knobs/overview
  last_modified: 15-07-2026
---

# flutter_widgetbook_knobs

Instructions for the agent to follow when this skill is activated.

## When to use

Use this skill when:
- Writing a new `@widgetbook.UseCase` function and a widget property should be adjustable live in the Widgetbook UI instead of hard-coded.
- Converting a static use case (fixed values) into an interactive one.
- The widget property is optional (`nullable`) and the use case should let a user toggle it to `null`.
- Setting up multi-snapshot generation for Widgetbook Cloud, where a single use case needs to render with several predefined knob value combinations.

## Instructions

1. **Access knobs via `context.knobs`** inside the use case builder function. The `BuildContext` is provided as the use case function's parameter.

2. **Every knob requires a unique `label`** within its `WidgetbookUseCase`. Optionally provide `description` (shown as inline docs in the Widgetbook UI) and `initialValue` (the value when the use case first loads; defaults to `null` for nullable knobs).

3. **Choose the correct knob type for the property**, using the regular (non-null) or `OrNull` variant depending on whether the widget property is nullable:

   | Dart type | Regular knob | Nullable knob |
   | --- | --- | --- |
   | `bool` | `context.knobs.boolean(...)` | `context.knobs.booleanOrNull(...)` |
   | `int` | `context.knobs.int.input(...)` or `context.knobs.int.slider(...)` | `context.knobs.intOrNull.input(...)` / `.slider(...)` |
   | `double` | `context.knobs.double.input(...)` or `context.knobs.double.slider(...)` | `context.knobs.doubleOrNull.input(...)` / `.slider(...)` |
   | `String` | `context.knobs.string(...)` | `context.knobs.stringOrNull(...)` |
   | `Duration` | `context.knobs.duration(...)` | `context.knobs.durationOrNull(...)` |
   | `DateTime` | `context.knobs.dateTime(...)` | `context.knobs.dateTimeOrNull(...)` |
   | `Color` | `context.knobs.color(...)` | `context.knobs.colorOrNull(...)` |
   | `T` (iterable of options, segmented control) | `context.knobs.iterable.segmented(...)` | `context.knobs.iterableOrNull.segmented(...)` |
   | `T` (enum/object, dropdown) | `context.knobs.object.dropdown(...)` | `context.knobs.objectOrNull.dropdown(...)` |
   | `T` (enum/object, segmented control) | `context.knobs.object.segmented(...)` | `context.knobs.objectOrNull.segmented(...)` |

   - Prefer `.slider` for numeric ranges with a known min/max where visually dragging is useful; prefer `.input` for free-form numeric entry.
   - Prefer `object.dropdown` for enums/objects with many options; prefer `object.segmented` (or `iterable.segmented`) when there are few options (e.g. 2–4) and a segmented button reads better than a dropdown.

4. **For `object`/`iterable` knobs, supply `options`** (the list of selectable values — commonly `EnumName.values`) and an optional `labelBuilder` to format each option's display text (e.g. `(value) => value.name`).

5. **Do not hard-code values that the user wants to vary** — replace the literal with the appropriate `context.knobs.*` call, keeping the surrounding widget construction unchanged.

6. **For multi-snapshot / Widgetbook Cloud support**, add a `cloudKnobsConfigs` map to the `@UseCase` annotation, keyed by a scenario name, with a list of the matching `*KnobConfig` objects (e.g. `BooleanKnobConfig('value', true)`, `NullKnobConfig('value')`) — one entry per knob whose value should be overridden for that snapshot. This is additive; the use case body itself does not change.

7. **After adding/editing knobs**, run `dart analyze` (or `dart-run-static-analysis`) and, if a Widgetbook app is running, use hot reload to confirm the new knob appears correctly in the Widgetbook UI's knobs panel.

## Examples

### Boolean knob (regular and nullable)
```dart
@UseCase(type: Checkbox, name: 'Duo state')
Widget buildCheckboxUseCase(BuildContext context) {
  return Checkbox(
    value: context.knobs.boolean(label: 'value'),
    onChanged: (value) {},
  );
}

@UseCase(type: Checkbox, name: 'Tristate')
Widget buildTristateCheckboxUseCase(BuildContext context) {
  return Checkbox(
    value: context.knobs.booleanOrNull(label: 'value'),
    tristate: true,
    onChanged: (value) {},
  );
}
```

### String, int slider, and Color knobs
```dart
@widgetbook.UseCase(name: 'with different title', type: Container)
Widget myWidget(BuildContext context) {
  return MyHomePage(
    title: context.knobs.string(
      label: 'Title Label',
      initialValue: 'HomePage',
    ),
  );
}

@UseCase(type: MyButton, name: 'Default')
Widget buildMyButtonUseCase(BuildContext context) {
  return MyButton(
    elevation: context.knobs.double.slider(
      label: 'elevation',
      initialValue: 2,
      min: 0,
      max: 16,
    ),
    padding: context.knobs.int.slider(
      label: 'padding',
      initialValue: 8,
      min: 0,
      max: 32,
    ),
    color: context.knobs.color(
      label: 'color',
      initialValue: Colors.blue,
    ),
  );
}
```

### Enum property via object dropdown/segmented knob
```dart
enum OnlineStatusType { online, offline, busy }

@UseCase(type: MyWidget, name: 'Default')
Widget buildUseCase(BuildContext context) {
  return MyWidget(
    status: context.knobs.object.dropdown(
      label: 'status',
      labelBuilder: (value) => value!.name,
      options: OnlineStatusType.values,
    ),
  );
}

// Few options: prefer a segmented control over a dropdown.
@UseCase(type: MyWidget, name: 'Segmented status')
Widget buildSegmentedUseCase(BuildContext context) {
  return MyWidget(
    status: context.knobs.object.segmented(
      label: 'status',
      labelBuilder: (value) => value!.name,
      options: OnlineStatusType.values,
    ),
  );
}
```

### Multi-snapshot config for Widgetbook Cloud
```dart
@UseCase(
  type: Checkbox,
  name: 'Tristate',
  cloudKnobsConfigs: {
    'Mixed': [NullKnobConfig('value')],
    'Checked': [BooleanKnobConfig('value', true)],
    'Unchecked': [BooleanKnobConfig('value', false)],
  },
)
Widget buildCheckboxUseCase(BuildContext context) {
  return Checkbox(
    value: context.knobs.booleanOrNull(label: 'value'),
    tristate: true,
    onChanged: (value) {},
  );
}
```
