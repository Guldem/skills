---
name: flutter_slivers
description: Use Flutter's sliver widgets (CustomScrollView, SliverAppBar, SliverList, SliverGrid, SliverPersistentHeader, etc.) to achieve fancy, high-performance scrolling effects — collapsing/floating app bars, mixed list+grid layouts, pinned headers, and parallax. Use when a single ListView/GridView isn't enough, e.g. combining multiple scrollable sections in one scroll view or animating an app bar with scroll position.
metadata:
  url: https://docs.flutter.dev/ui/layout/scrolling/slivers
  last_modified: 14-07-2026
---

# flutter_slivers

Instructions for the agent to follow when this skill is activated.

## When to use

Use this skill when:
- The user needs a collapsing, floating, snapping, or pinned app bar that reacts to scroll (`SliverAppBar`).
- Multiple distinct scrollable sections (e.g. a header, a grid, then a list) need to scroll together as **one continuous scroll view** rather than nested independent scrollables.
- The user wants scroll-driven effects: parallax images, elastic/stretchy headers, sticky section headers.
- A single `ListView`/`GridView` can't express the layout because it mixes different item shapes/sizes in one scroll.
- Performance matters for a long list/grid mixed with other content — slivers lazily build only visible children, same as `ListView.builder`.

Do **not** reach for slivers if a plain `ListView`, `GridView`, or `SingleChildScrollView` + `Column` already satisfies the layout — slivers add complexity and should only be used when `CustomScrollView` is actually needed to combine multiple scrolling regions.

## Instructions

1. **Always wrap slivers in a `CustomScrollView`.** Slivers are not standalone widgets — they must be children of the `slivers:` list of a `CustomScrollView` (or another sliver-aware container like `NestedScrollView`).

2. **Pick the right sliver for the job:**
   - `SliverAppBar` — app bar that can expand/collapse/float/pin/snap with scroll. Use `flexibleSpace` for a background image/parallax effect, `pinned: true` to keep the collapsed bar visible, `floating: true` to reappear immediately on scroll-up, `snap: true` (with `floating: true`) to fully animate in/out, and `stretch: true` for an overscroll/elastic effect.
   - `SliverList` — a non-lazy-by-default sliver; use `SliverList.list` for small fixed collections. For large/lazy lists prefer `SliverList(delegate: SliverChildBuilderDelegate(...))` or the shorthand `SliverList.builder(itemBuilder: ..., itemCount: ...)`.
   - `SliverGrid` — grid version of the above; use `SliverGrid.builder` with `gridDelegate: SliverGridDelegateWithFixedCrossAxisCount(...)` or `SliverGridDelegateWithMaxCrossAxisExtent(...)`.
   - `SliverToBoxAdapter` — wrap a single ordinary (non-sliver) widget so it can sit inside a `CustomScrollView` alongside other slivers.
   - `SliverPadding` — apply padding around a sliver (regular `Padding` does not work directly on slivers).
   - `SliverFillRemaining` — fill any remaining viewport space, useful for a footer or an empty-state widget at the end of a scroll view.
   - `SliverPersistentHeader` — build custom pinned/sticky section headers with a `SliverPersistentHeaderDelegate` (`minExtent`, `maxExtent`, `shrinkOffset`-aware `build`).
   - `SliverStickyHeader` (community package `sticky_headers`/`sliver_tools`) if built-in `SliverPersistentHeader` boilerplate is excessive for simple sticky-section-header use cases — mention this as an alternative but implement with core SDK widgets by default.

3. **Combine multiple scrollable sections as siblings in one `slivers` list** instead of nesting `ListView`s inside a `Column` inside a `SingleChildScrollView` — nesting causes janky/broken scroll physics and loses lazy building.

4. **For scroll views inside a `TabBarView`/nested scrolling UI** (e.g. a collapsing header shared across tabs), use `NestedScrollView` with `headerSliverBuilder` for the shared slivers (typically a `SliverAppBar`) and `body` for the per-tab scrollable content.

5. **Respect `SliverChildBuilderDelegate` best practices:**
   - Always provide `itemCount` (via the `.builder` constructors) when the number of items is known, so Flutter can compute scrollbar/semantics info correctly.
   - Keep `itemBuilder` cheap and side-effect free — it's called lazily as items scroll into view, just like `ListView.builder`.
   - Use `addAutomaticKeepAlives`/`addRepaintBoundaries` defaults unless you have a specific reason to change them.

6. **For `SliverAppBar` flexible space with a parallax image:**
   - Wrap the image in a `FlexibleSpaceBar` (its `background` slot) inside `flexibleSpace`.
   - Set `expandedHeight` on the `SliverAppBar` to control the max height.
   - Use `FlexibleSpaceBar.title` for a title that fades/moves as the bar collapses.

7. **Verify on-device/simulator.** After building or editing a sliver-based scroll view, use the Flutter tools (hot reload, widget inspector) to confirm scroll behavior — check for jank, incorrect `itemExtent`/`itemCount`, or missing `key`s causing state loss during scroll.

## Examples

### Basic CustomScrollView: collapsing app bar + list
```dart
CustomScrollView(
  slivers: [
    SliverAppBar(
      expandedHeight: 200,
      pinned: true,
      flexibleSpace: FlexibleSpaceBar(
        title: const Text('Fancy Scrolling'),
        background: Image.network('https://example.com/header.jpg', fit: BoxFit.cover),
      ),
    ),
    SliverList.builder(
      itemCount: items.length,
      itemBuilder: (context, index) => ListTile(title: Text(items[index])),
    ),
  ],
)
```

### Mixing a grid, a single widget, and a list in one scroll view
```dart
CustomScrollView(
  slivers: [
    const SliverToBoxAdapter(child: SectionHeader('Featured')),
    SliverPadding(
      padding: const EdgeInsets.all(16),
      sliver: SliverGrid.builder(
        gridDelegate: const SliverGridDelegateWithFixedCrossAxisCount(
          crossAxisCount: 2,
          mainAxisSpacing: 12,
          crossAxisSpacing: 12,
        ),
        itemCount: featured.length,
        itemBuilder: (context, index) => FeaturedCard(featured[index]),
      ),
    ),
    const SliverToBoxAdapter(child: SectionHeader('All items')),
    SliverList.builder(
      itemCount: allItems.length,
      itemBuilder: (context, index) => ListTile(title: Text(allItems[index].name)),
    ),
    const SliverFillRemaining(
      hasScrollBody: false,
      child: Center(child: Text('You reached the end')),
    ),
  ],
)
```

### Floating + snapping app bar
```dart
SliverAppBar(
  floating: true,
  snap: true,
  title: const Text('Reappears immediately on scroll-up'),
)
```

### Custom sticky section header via SliverPersistentHeader
```dart
class _SectionHeaderDelegate extends SliverPersistentHeaderDelegate {
  _SectionHeaderDelegate(this.title);
  final String title;

  @override
  double get minExtent => 48;
  @override
  double get maxExtent => 48;

  @override
  Widget build(BuildContext context, double shrinkOffset, bool overlapsContent) {
    return Container(
      color: Theme.of(context).colorScheme.surface,
      alignment: Alignment.centerLeft,
      padding: const EdgeInsets.symmetric(horizontal: 16),
      child: Text(title, style: Theme.of(context).textTheme.titleMedium),
    );
  }

  @override
  bool shouldRebuild(covariant _SectionHeaderDelegate oldDelegate) =>
      oldDelegate.title != title;
}

// Usage inside CustomScrollView.slivers:
SliverPersistentHeader(
  pinned: true,
  delegate: _SectionHeaderDelegate('Section A'),
),
```

### Shared collapsing header across tabs with NestedScrollView
```dart
NestedScrollView(
  headerSliverBuilder: (context, innerBoxIsScrolled) => [
    SliverAppBar(
      expandedHeight: 200,
      pinned: true,
      flexibleSpace: const FlexibleSpaceBar(title: Text('Profile')),
      bottom: const TabBar(tabs: [Tab(text: 'Posts'), Tab(text: 'About')]),
    ),
  ],
  body: TabBarView(
    children: [
      ListView.builder(itemCount: posts.length, itemBuilder: (_, i) => PostTile(posts[i])),
      const AboutSection(),
    ],
  ),
)
```
