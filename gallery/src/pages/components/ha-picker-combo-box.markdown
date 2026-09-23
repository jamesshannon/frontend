---
title: Picker combo box
---

# Picker combo box `<ha-picker-combo-box>`

The searchable list used inside pickers. `ha-generic-picker` renders it in a
popover or dialog; entity, area, floor, label and device pickers all reach it
that way. It is shown directly here so the list and its keyboard model can be
inspected on their own.

It takes no `hass`. Items come from a `getItems` callback and locale comes from
`internationalizationContext`.

## Two render paths

The item count picks the renderer, and the two behave differently enough that
list changes need checking against both.

| Items                         | Renderer          | Rows in the DOM       |
| ----------------------------- | ----------------- | --------------------- |
| 12 or fewer, no sections      | `repeat()`        | all of them           |
| more than 12, or any sections | `lit-virtualizer` | only the visible ones |

The threshold is `MAX_PLAIN_LIST_ITEMS`. Filtering keeps the mode the list
opened with — only the full item set decides it. Every real entity picker is
over the threshold, so the virtualized path is the one users meet.

## Two highlights

The list has two distinct row backgrounds, and they mean different things.

- **`current-value`** — the row matching the `value` property. Set during
  render, and pinned into view when the list opens.
- **`selected`** — the keyboard cursor. Moved by <kbd>↑</kbd> <kbd>↓</kbd>
  <kbd>Home</kbd> <kbd>End</kbd>, and it is the row <kbd>Enter</kbd> picks.

`selected` is applied imperatively to the row node rather than rendered from
state, and under the virtualizer it is applied a frame after the row is
scrolled into view, because the row may not exist yet.

## Keyboard

| Key                                  | Behavior                                 |
| ------------------------------------ | ---------------------------------------- |
| <kbd>↑</kbd> <kbd>↓</kbd>            | Move the cursor, skipping section titles |
| <kbd>Home</kbd> <kbd>End</kbd>       | Jump to the first or last item           |
| <kbd>Enter</kbd>                     | Pick the row under the cursor            |
| <kbd>Ctrl/⌘</kbd> + <kbd>Enter</kbd> | Pick it, opening in a new tab            |

With one item in the list, <kbd>Enter</kbd> picks it whether or not the cursor
has moved.

## Implementation

### Example usage

`getItems` and `value` are set as properties, not attributes. The component
filters the returned list itself as the user types, so `getItems` can ignore its
search argument and return everything.

```ts
const getItems = (): PickerComboBoxItem[] => [
  { id: "light.desk", primary: "Desk", secondary: "Office" },
  { id: "light.porch", primary: "Porch", secondary: "Outside" },
];

// in render(), inside a lit template:
//   <ha-picker-combo-box
//     .getItems=[getItems]
//     .value=[this.value]
//     @value-changed=[this._valueChanged]
//   ></ha-picker-combo-box>
```

Gallery markdown is compiled into a lit template, so a literal dollar-brace
sequence in a fenced block becomes a live substitution. Square brackets stand in
for it above.

The host is `display: flex` with `flex: 1` and expects a parent with a bounded
height. In the app that bound comes from the popover; embedding it directly
means supplying one.

### Properties/Attributes

| Name                 | Type               | Default   | Description                                                                                                                 |
| -------------------- | ------------------ | --------- | --------------------------------------------------------------------------------------------------------------------------- |
| getItems             | Function           | -         | Required. Returns the items, optionally filtered by search string and section.                                              |
| getAdditionalItems   | Function           | -         | Extra items appended to search results, for example "add new".                                                              |
| value                | String             | -         | Id of the current item. Marks its row and pins it into view.                                                                |
| label                | String             | -         | Placeholder for the search field. Falls back to a localized "Search".                                                       |
| mode                 | "popover"/"dialog" | "popover" | Adjusts padding for the surface the list sits in.                                                                           |
| shown                | Boolean            | true      | Whether the surface finished animating. `ha-generic-picker` sets it so the virtualizer does not measure rows mid-animation. |
| sections             | Array              | -         | Filter chips. Section headers are plain strings returned by `getItems`.                                                     |
| sectionTitleFunction | Function           | -         | Builds the sticky section title from the visible range.                                                                     |
| searchKeys           | Array              | -         | Fuse weighted keys for fuzzy search.                                                                                        |
| searchFn             | Function           | -         | Post-processes filtered results.                                                                                            |
| rowRenderer          | Function           | -         | Replaces the default row template.                                                                                          |
| allowCustomValue     | Boolean            | false     | Offers the typed string as an item.                                                                                         |
| customValueLabel     | String             | -         | Label for that custom item.                                                                                                 |
| notFoundLabel        | String/Function    | -         | Shown when a search matches nothing.                                                                                        |
| emptyLabel           | String             | -         | Shown when there are no items at all.                                                                                       |
| noSort               | Boolean            | false     | Keeps the order `getItems` returned.                                                                                        |
| clearable            | Boolean            | false     | Adjusts search field padding for a clear affordance.                                                                        |
| disabled             | Boolean            | false     | Disables the list.                                                                                                          |
| required             | Boolean            | false     | Marks the field required.                                                                                                   |

### Events

| Event            | Detail              | Description                                                    |
| ---------------- | ------------------- | -------------------------------------------------------------- |
| `value-changed`  | `{ value }`         | An item was picked.                                            |
| `index-selected` | `{ index, newTab }` | Which row was picked, and whether it should open in a new tab. |
