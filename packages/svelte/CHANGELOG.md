# @gantry/svelte

## 2.0.0

### Major Changes

- 19d6e62: Gantry is now available for every framework: `@gantry/core` (framework-free logic), `@gantry/vanilla` (`<gantry-element>` Web Component), and `@gantry/react`, `@gantry/vue`, `@gantry/svelte` bindings. `ngx-gantry` now builds on `@gantry/core` and gains column sorting (built-in columns are sortable).
- 019ac69: **`@gantry/svelte` 2.0: a one-level copy instead of a deep clone, nullable props, no lost first events.**
  
  - **Data props are copied one level, never deep-cloned (breaking).** `tasks`, `columns`, `annotations` and
    `options` reach `<gantry-element>` as a plain array (or object) holding your own items: no `$state` proxy at the
    top and no `$state.snapshot` of every task on each change. Assigning a new array, and on `$state` `push` /
    `splice` / `tasks[i] = …` / `options.viewMode = …`, update the chart; a field changed in place inside an item
    (`tasks[i].end = …`) is no longer seen — replace the item, or keep large plans in `$state.raw` and assign a new
    array.
  - **Clearing a prop restores the default.** A prop that goes back to `undefined` is forwarded (it used to be
    ignored); a prop never passed leaves the element's own value (its default, an attribute such as `dark-mode`, or a
    property set through `bind:element`).
  - **First events arrive.** The listeners are attached before the first property is set, so an event fired while
    the chart starts (`onViewModeChange` for `options.viewMode`) reaches its callback; callbacks are read when the
    event fires.
  - Each property is re-assigned only when it changes, after the listeners and before the element's first frame (no
    flash of defaults); unmounting removes the listeners; server rendering (`svelte/server`) emits
    `<gantry-element>` without touching the DOM. The prop and callback lists are checked against `GANTRY_PROPERTIES`
    and `GANTRY_EVENTS`.

### Minor Changes

- 2cf00e4: Fully configurable chart, rendered from one shared core model in every binding: N-tier header scales with custom formats (ISO / Sunday weeks, "W52", "4–11 Mar"…), zoom presets and an Hour view; period delimiters through header and body; weekends, holidays, working and intensive hours; bar / milestone / group / arrow sizing and styling; a resizable, collapsible task-list panel (30% by default, remembers its last width) with horizontal scroll, a split / table / chart view switch and persisted UI state (`stateKey`, `state-change`). Fixes time-zone and daylight-saving misalignment, header/row misalignment, label overflow and many smaller rendering bugs.
- 6522a87: Split panel vs table view: the panel next to the timeline shows `name`, `start`, `end`, `progress`; the table view shows type, a derived status (with a coloured dot), duration, predecessors and, with `baseline`, baseline start/end and variance. `panel.columns` / `table.columns` pick columns by key (built-ins need no definition), `setColumnVisible(key, visible, view)` works per view, sorting reorders siblings at every tree level, and custom columns get `sortValue` and a `ColumnContext`. Visible changes: the default panel drops Duration; Duration reads the drawn span (one day longer with `endDateMode: 'inclusive'`); code building a full `GanttLabels` object needs the new keys.

### Patch Changes

- Updated dependencies [639ad08]
- Updated dependencies [6522a87]
- Updated dependencies [2cf00e4]
- Updated dependencies [7bf8c1b]
- Updated dependencies [1cedb3f]
- Updated dependencies [6522a87]
- Updated dependencies [6522a87]
- Updated dependencies [6522a87]
- Updated dependencies [19d6e62]
- Updated dependencies [d2e5d4c]
- Updated dependencies [cdcf5cd]
- Updated dependencies [6522a87]
- Updated dependencies [7bf8c1b]
- Updated dependencies [c8b61c7]
- Updated dependencies [c8b61c7]
- Updated dependencies [c8b61c7]
- Updated dependencies [6522a87]
  - @gantry/core@2.0.0
  - @gantry/vanilla@2.0.0
