# @gantry/react

## 2.0.0

### Major Changes

- 19d6e62: Gantry is now available for every framework: `@gantry/core` (framework-free logic), `@gantry/vanilla` (`<gantry-element>` Web Component), and `@gantry/react`, `@gantry/vue`, `@gantry/svelte` bindings. `ngx-gantry` now builds on `@gantry/core` and gains column sorting (built-in columns are sortable).
- 019ac69: `<Gantry>` applies its props and listeners in the commit (layout effects in the browser, nothing on the server), so the
  chart's first frame already shows the passed `tasks` and `options` instead of flashing its defaults.
  
  - A prop set back to `undefined` (or left out after having a value) restores the element's default; it used to keep
    the last value. A prop that was never set is not written.
  - Events fired while the element applies its first props are delivered (listeners are attached first): for example
    `onViewModeChange` for `options.viewMode`, or `onLoadError` for a data source that already failed.
  - Callbacks are read from a ref updated in the commit, never during render: a render React discards (a suspended
    transition) can no longer swap in its callbacks, and changing a callback never re-adds listeners.
  - Objects are passed through unchanged (`ref.current.tasks === tasks`), and the ref is attached once the props are set.
  - `className` is set as the `class` attribute, so it also styles the chart in React 18 (which wrote a `classname`
    attribute on custom elements).
  - Server rendering emits `<gantry-element>` without touching DOM APIs and without React 18's layout-effect warning.
  
  Breaking: a prop set back to `undefined` now restores the element default (it used to keep the last value).

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
