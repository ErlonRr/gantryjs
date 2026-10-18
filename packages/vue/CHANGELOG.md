# @gantry/vue

## 2.0.0

### Major Changes

- 19d6e62: Gantry is now available for every framework: `@gantry/core` (framework-free logic), `@gantry/vanilla` (`<gantry-element>` Web Component), and `@gantry/react`, `@gantry/vue`, `@gantry/svelte` bindings. `ngx-gantry` now builds on `@gantry/core` and gains column sorting (built-in columns are sortable).
- 019ac69: `<Gantry>` for Vue sets its listeners and props on `<gantry-element>` before Vue inserts it, so the chart connects
  and paints its first frame with your props (no flash of the defaults), and events fired by the first assignment or
  the first connect (`view-mode-change` from `options.viewMode`, `load-error` of a data source that already failed)
  reach the emits.
  
  - A prop that becomes `undefined` (or is dropped) restores the element's default; it used to keep the last value.
  - Your objects reach the element as they are: Vue proxies are unwrapped with `toRaw`, so `element.tasks` is the
    array you passed (an array that holds proxies, as `tasks.value.map(…)` returns, becomes an array of the raw items).
    Arrays are no longer copied on every assignment, only after an in-place change (`push`, `splice`, `list[i] = …`),
    because the element compares arrays by identity.
  - The template ref exposes only `element` (`expose({ element })`); the rest of the component instance is private.
  - The listeners come off after the element is unmounted, so a `state-change` it fires while disconnecting (a
    pending save) still reaches `@state-change`.
  - The props list is explicit and checked against `GANTRY_PROPERTIES`; server rendering (`renderToString`) emits
    `<gantry-element>` without touching the DOM. The README documents `@state-change` and `@load-error`.
  
  Breaking: the template ref (`ref="chart"`) exposes only `element` (`chart.value.element`); it used to be the whole
  component instance, including the undocumented `setElement` and the props.
  
  A reactive `options` object is watched one level deep and reaches the element as a one-level copy, so an in-place
  change of a top-level field (`options.viewMode = 'Week'`) is applied; replace a nested object (`options.panel = {…}`)
  to change it. Plain (non-reactive) objects are passed through unchanged.

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
