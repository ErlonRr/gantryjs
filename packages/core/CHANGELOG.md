# @gantry/core

## 2.0.0

### Major Changes

- 639ad08: Accessible default palette (WCAG 2.2 AA): bar labels reach 4.5:1 and bars, markers and arrows 3:1. Labels inside bars switch to a readable colour on custom `barColor`s (`contrastRatio()`, `readableOn()`). The previous colours are exported as `LegacyLightTheme` / `LegacyDarkTheme`.
- 1cedb3f: **No more `dayjs`: zero runtime dependencies.** Dates are formatted by a built-in, `Intl`-based formatter
  with dayjs-compatible tokens (`YYYY YY M MM MMM MMMM Q D DD Do d dd ddd dddd H HH h hh m mm s ss A a W WW [text]`).
  `locale` is any BCP 47 tag (`'it'`, `'de-CH'`, `'pt-BR'`) and needs no `import 'dayjs/locale/…'`; unknown or malformed tags
  fall back to `'en'`, which is also the default — the dayjs global locale (`dayjs.locale('it')`) is no longer read, so set
  `locale` explicitly. Names come from CLDR, so a few abbreviations change (German `Mär`, Spanish `sept` / `a. m.`,
  lowercase Catalan months, Russian `июн.`), and a short month after a day number uses the format-context form (`5 мар.`).
  New exports: `dateNames(locale, kind)`, `resolveFormatLocale`, `isoWeek`, `isoWeekYear`, `quarterOf`, `dayOfYear`, `ISO_FORMAT`.
  
  **Shared logic in `@gantry/core`**, used by both renderers instead of their own copies:
  virtual rows (`computeVirtualWindow` with pinned rows, `isIndexMounted`, `virtualIndexes`), horizontal windowing
  (`computeHorizontalWindow`, `tiersInRange`, `bandsInRange`, `linesInRange`, `sameHorizontalWindow`), selection
  (`selectionClick`, `selectAllRows`, `selectionOf`, `emptySelection`, `shouldEmitSelection`), view and panel width
  (`viewFromOptions`, `viewFromState`, `resolvePanelWidth`, `clampPanelWidth`), versioned state persistence
  (`GANTT_STATE_VERSION`, `parseState`, `serializeState`, `normalizeState`, `readStoredState`, `writeStoredState`),
  tooltip and legend models (`buildTooltipModel`, `buildLegend`), keyboard (`resolveKey`, `navigateGrid`,
  `KeyboardEditSession`), screen-reader text (`taskAccessibleName`, `announce*`).
  
  Breaking: stored state is written as version 2 (`{"v":2,…}`; v1 snapshots are migrated, invalid fields are dropped,
  snapshots from a newer version are ignored); `GanttLabels` has new required members for accessible names
  (`ariaProgress`, `ariaCritical`, `ariaBaseline`, `ariaDateRange`, `ariaDateFormatShort`, `ariaDateFormatLong`,
  `ariaTimeFormat`, `ariaListSeparator`), announcements (`announce*`), tooltip rows (`ttCritical`, `ttCriticalValue`,
  `ttFloat`, `ttFloatDays`, `ttSuccessors`) and the legend (`legendCritical`, `legendBaseline`) — `options.labels` stays
  partial; `GanttBarContext` fields are required; `DragPreview` is `{ delta, unit }` (`daysDelta` is deprecated);
  `criticalIds` is a `ReadonlySet`; the legacy helpers `PX_PER_DAY`, `HEADER_HEIGHTS`, `getTimelineRange`,
  `generateHeaderRows`, `generateGridLines`, `zoomIn`, `zoomOut` are removed (use `buildTimeline` and the zoom levels).
  
  Fixed: the Hour view drew overlapping, out-of-order header cells in time zones west or east of UTC (an hour boundary
  on UTC midnight was read as a date-only value); holidays are looked up in O(1) while painting.
  
  Renderers (`<gantry-element>` and `<gantry>` behave the same):
  
  - Bars' accessible names come from `taskAccessibleName`: name, type, dates (with the time of day when a task
    doesn't start or end at midnight; formats `labels.ariaDateFormatShort` / `ariaDateFormatLong` / `ariaTimeFormat`),
    progress (rounded, clamped), "critical" and "planned baseline shown" (not for milestones).
  - The tooltip shows critical path, float, baseline start / end / variance, predecessors and successors when that data
    exists; milestones show one date row (Angular's "◆ Milestone" badge is gone); progress is clamped to 0–100 and its
    fill follows the critical colour. Angular's tooltip now has `role="tooltip"` and the shared `gantt-tooltip` class.
  - The legend honours `showToday` and lists the critical path (bar) and baseline (strip) when those options are on; the
    swatch geometry comes from `@gantry/core` (`legendSwatch`, `LEGEND_SWATCH_SIZE`).
  - Keyboard: `Ctrl/⌘+Shift+A` also selects all; shortcuts no longer fire with `Alt` / `AltGr`; unhandled keys
    (`F`, `Ctrl+Z`, `Ctrl+Y`) keep their browser default.
  - `selection-change` / `selectionChange` is no longer emitted when clearing an already empty selection.
  - `setState()` drops invalid fields (e.g. `panelWidth: 0`); an explicit `panel.collapsed: false` beats `panelVisible: false`.
  - `@gantry/vanilla`: the element's selection and tooltip internals that implement this are private in 2.0 (see the
    `wc-engine` changeset); the `virtual-scroll` entry no longer has its own `computeVirtualWindow` (use the core one).
  - `ngx-gantry`: the selection service, task list, tooltip and canvas these changes touched are internal in
    `ngx-gantry` 2.0 (see its changeset); use `<gantry>`'s `selectionChange`, `getSelectedTasks()` and
    `setSelectedTasks()`.
- 19d6e62: Gantry is now available for every framework: `@gantry/core` (framework-free logic), `@gantry/vanilla` (`<gantry-element>` Web Component), and `@gantry/react`, `@gantry/vue`, `@gantry/svelte` bindings. `ngx-gantry` now builds on `@gantry/core` and gains column sorting (built-in columns are sortable).
- c8b61c7: **Theming through CSS variables.** Every colour of the render model is now a `Paint`: a CSS colour
  value that is either a theme token (`var(--gantt-*)`) or an explicit task / option colour passed
  through as is. Both renderers apply paints as CSS properties, and the theme reaches the pixels only
  through custom properties. The resolved config no longer carries the theme.
  
  ### `@gantry/core` (breaking)
  
  - `resolveConfig(options, theme, i18n)` becomes `resolveConfig(options?, i18n?, metrics?)`, where
    `metrics` is `Partial<ThemeMetrics>` (pass `themeMetrics(theme)`; only `barRadius` is read, as the
    default `bar.radius`). `ResolvedGanttConfig.theme` is removed.
  - The resolved-colour fields are replaced by paints:
  
    | Removed                                             | Replacement                                                                  |
    | --------------------------------------------------- | ---------------------------------------------------------------------------- |
    | `BarVisual.fill`, `.label.color`, `.progress.color` | `BarVisual.paint.{body,label,progress}`, plus `BarVisual.tone`               |
    | `ArrowVisual.color`                                 | `ArrowVisual.paint`                                                          |
    | `TimeBand.color`                                    | `TimeBand.paint` (bands merge by kind and paint)                             |
    | `ResolvedLine.color`                                | `ResolvedLine.paint`                                                         |
    | `LegendItem.color`                                  | `LegendItem.paint`                                                           |
    | `CellVisual.dotColor`                               | `CellVisual.dotPaint`                                                        |
    | `statusColor(status, theme)`                        | `statusPaint(status)`                                                        |
    | `TooltipModel.colors`, `TooltipColors`              | `TooltipModel.critical` + `TooltipModel.paint.progress` (chrome: CSS tokens) |
  
  - No more theme inputs: `buildTooltipModel`'s `config` has no `theme`, `buildLegend` takes no
    `theme`, `computeCellVisual(col, row, ctx)` drops its fourth argument, and `computeTimeBands`
    reads no `cfg.theme`. `computeBarVisual` takes `fontPx` and `barTextColor` (the label colour on an
    explicit `task.barColor`, default `'#FFFFFF'`) from `themeMetrics(theme)`.
  - Bar labels on theme fills use the contrast tokens `--gantt-bar-text-on-bar`,
    `--gantt-bar-text-on-milestone` and `--gantt-bar-text-on-critical`
    (`readableOn(fill, barTextColor)`). A page-CSS override of `--gantt-bar`, `--gantt-milestone` or
    `--gantt-critical` must also set the matching token, or use `setTheme()`.
  - Custom renderers: get a concrete colour with `resolvePaint(paint, theme)` (it accepts the bar
    context's `ctx.theme`).
  - `annotationPaint()` gives `isToday` annotations the today colour (the Web Component used to ignore
    `isToday`; Angular already drew them in the today colour), so the line, its region and its legend
    swatch agree in both renderers.
  - New label `dragDelta(amount, unit)` (`'+3d'` / `'-2h'`): the movement shown above the drag ghost is
    translatable. Like the other new `GanttLabels` members, only objects typed as a full `GanttLabels`
    need it; `options.labels` stays partial.
  - New exports: `Paint`, `ThemeColorKey`, `themeVar`, `isThemeVar`, `resolvePaint`, `PAINTS`,
    `rowPaint`, `annotationPaint`, `statusPaint`, `themeToCss`, `CONTRAST_CSS_VARS`, `ThemeMetrics`,
    `themeMetrics`, `sameThemeMetrics`, `BarTone`, `BarPaint`, `TooltipPaint`, `headerCellBorder`,
    `buildArrowIndex` (`ArrowIndex`, `ArrowLink`), `computeDragGhostVisual` (`GhostVisual`,
    `GhostLabel`), `DRAG_SOURCE_OPACITY`, `quantizeNow`, `todayX`, `periodNext`,
    `VIEWPORT_FALLBACK_HEIGHT`.
  
  ```ts
  // before
  const cfg = resolveConfig(options, theme, i18n);
  bar.setAttribute('fill', visual.fill);
  // after
  const cfg = resolveConfig(options, i18n, themeMetrics(theme));
  bar.style.setProperty('fill', visual.paint.body); // or resolvePaint(visual.paint.body, theme)
  ```
  
  ### `@gantry/vanilla`
  
  - The theme variables live in a `:host` stylesheet inside the shadow root instead of the host's
    inline style: page CSS on `gantry-element` overrides them, and a string `style` attribute (for
    example Svelte's `style=`) or `setAttribute('style', …)` no longer wipes them.
  - With `darkMode: 'auto'`, an OS colour-scheme flip re-renders the chart at once.
  - Colours are set through the CSSOM, never as presentation attributes or style text, so a task
    colour such as `'red;background:url(x)'` can no longer inject CSS.
  - Custom `renderBar`: `ctx.visual.fill`, `ctx.visual.label.color` and `ctx.visual.progress.color`
    are gone; `ctx.visual.paint.{body,label,progress}` may be `var()` strings — apply them as style
    properties or resolve them with `resolvePaint(paint, ctx.theme)`. `ctx.theme` is the element's
    resolved theme.
  - The built-in bar a custom `renderBar` gets from `renderDefault()` paints with the `--gantt-*`
    variables, so it needs them in scope (inside the element or on an ancestor). The low-level painters
    (`paintBar`, `GanttCanvasProps`, `GanttTaskListProps`, …) are no longer exported (see the engine
    changeset).
  - `@gantry/react`, `@gantry/vue` and `@gantry/svelte` follow in the same release (fixed version
    group); no API change there.
  
  ### `ngx-gantry` (breaking)
  
  `ngx-gantry` re-exports `@gantry/core`, so every core change above reaches Angular users. In
  particular, **custom bars** registered with `provideGanttBar()` that read `ctx.visual.fill`,
  `ctx.visual.label.color` or `ctx.visual.progress.color` must switch to `ctx.visual.paint.body` /
  `.label` / `.progress`: bind them as styles (`[style.fill]="ctx.visual.paint.body"`) or resolve them
  to colours with `resolvePaint(paint, ctx.theme)`. Code calling `resolveConfig`, `statusColor` or
  reading `TooltipModel.colors` migrates as described in the `@gantry/core` section.
  
  - Paints are bound through `[style.fill]` / `[style.stroke]` / `[style.background-color]`. Page CSS
    that targeted SVG `fill` / `stroke` presentation attributes no longer applies; restyle through the
    `--gantt-*` tokens.
  - The theme service also writes the contrast tokens (`themeToCss`).
  - Header cell borders come from core (`headerCellBorder`).
  - The chart's resolved theme reaches custom bars as `ctx.theme`; the `resolveConfig` call site
    passes the theme metrics, so a colour-only theme change re-renders no geometry.
  - The task list, load-more footer and drag preview are painted with the chart's `--gantt-*` tokens
    (their `theme` inputs are gone in `ngx-gantry` 2.0, where these components are internal).

### Minor Changes

- 2cf00e4: Fully configurable chart, rendered from one shared core model in every binding: N-tier header scales with custom formats (ISO / Sunday weeks, "W52", "4–11 Mar"…), zoom presets and an Hour view; period delimiters through header and body; weekends, holidays, working and intensive hours; bar / milestone / group / arrow sizing and styling; a resizable, collapsible task-list panel (30% by default, remembers its last width) with horizontal scroll, a split / table / chart view switch and persisted UI state (`stateKey`, `state-change`). Fixes time-zone and daylight-saving misalignment, header/row misalignment, label overflow and many smaller rendering bugs.
- 7bf8c1b: **Readable current-period label, themable progress overlay, shared binding helpers.**
  
  - **`--gantt-today-label`** (fourth derived token of `themeToCss()`, `CONTRAST_CSS_VARS.todayLabel`): the colour of
    the current-period header label, `todayLabelColor(theme)` = `todayColor` when it reaches 4.5:1 on the 14 % today
    tint over `headerBackground`, else `todayColor` mixed towards `textPrimary` in 10 % steps until it does. The light
    theme's label changes from `#B45309` (3.97:1, a WCAG AA failure) to `#964B13` (5.01:1); the dark theme keeps
    `#FBBF24`. `<gantry-element>`'s header uses it (`.g-cell--current .g-cell-label`); a page-CSS override of
    `--gantt-today` should set it too. New helper `readableOnTint(fg, bg, alpha, toward, min = 4.5)`;
    `resolvePaint()` resolves the new token.
  - **`progressOverlay`** theme key (`--gantt-progress-overlay`, `#000000` in every built-in palette, `PAINTS.progressOverlay`):
    bars without `task.progressColor` / `bar.progress.color` paint their progress with `var(--gantt-progress-overlay)`
    instead of the literal `#000000` (same pixels, now themable). A custom renderer reading
    `ctx.visual.paint.progress` gets the token: apply it as a style property or `resolvePaint()` it.
  - `themeToCss()` returns 42 custom properties (was 40); `<gantry-element>` lists both new ones as `@cssprop`.
  - Helpers both bindings share, moved from the Web Component into `@gantry/core`: `columnsGridTemplate(cols)` (the
    task-list `grid-template-columns`), `sameVirtualWindow(a, b)`, `HORIZONTAL_WINDOW_QUANTUM` (256),
    `announceLoadState(prev, next, labels)` with `LoadStateSnapshot` / `LoadStateLabels`, `TOOLTIP_SHOW_DELAY` (320 ms)
    and `TOOLTIP_HIDE_DELAY` (120 ms).
  - More shared rules and constants both bindings now read from `@gantry/core` instead of their own copies:
    `seedCollapsed(tasks, seen?)` (`task.collapsed` seeds only the ids a chart has never seen; returns the ids to
    collapse and the ids seen so far, type `CollapseSeed`), `clampColumnWidth(width, col)` with `COLUMN_MIN_WIDTH` (40)
    and `COLUMN_MAX_WIDTH` (800), `VIRTUAL_OVERSCAN_ROWS` (4, `computeVirtualWindow`'s default), `panelWidthRange(bounds,
  available)` (the limits `clampPanelWidth` clamps to, type `PanelWidthRange`), `splitterValue(input)` (the splitter
    separator's `aria-valuenow` / `aria-valuemin` / `aria-valuemax`, type `SplitterValue`), `SPLITTER_KEY_STEP` (24 px
    per Arrow key), `SPLITTER_TOGGLE_OFFSET` (24 px below the header), `STATE_SAVE_DELAY` (150 ms debounce of the state
    report and `stateKey` write) and `CLOCK_INTERVAL` (60 s).
- 6522a87: Data sources share one framework-free loader (`PagedLoader` in `@gantry/core`): failed pages retry automatically with exponential backoff (`retries`, `retryDelay`), then show an error with a Retry button, emit `loadError` / `load-error` and pause until `retry()` — no more request storm. `reset()` replaces instead of duplicating, `initialPage` works (`0` = wait for `loadNext()`), and Angular's `GanttDataSource` gains `error()`, `nextPage()` and `retry()` and no longer needs an injection context. New labels `loadError` and `retry`.
- 6522a87: `@gantry/core/demo` adds `createDemoData()` and `anchorToToday(template, { now, at })`: they move a plan by whole weeks so today sits inside it (about 40% in by default), keeping weekdays and date-only values. `DEMO_TASKS` / `DEMO_ANNOTATIONS` stay the fixed 2026 reference plan.
- 6522a87: New `initialScroll` option: `'auto'` (default) centres today only when it is near the tasks and otherwise shows the earliest task, so the first view always has bars; also `'today'`, `'start'`, a `Date` or `false`. Zooming now keeps the date at the viewport centre in place, an earlier range start no longer shifts the view, and smooth scrolling honours `prefers-reduced-motion`.
- cdcf5cd: `ViewMode`, `DependencyType` and `TaskType` are now `as const` objects with a same-name union type (no TS enums): `ViewMode.Week` reads the same and plain strings are accepted. Optional properties of the public types accept `undefined` explicitly, so projects using `exactOptionalPropertyTypes` can pass `undefined`. `ngx-gantry` declares `"type": "module"`. Task-list rows can be focused and selected with Enter / Space.
  
  New `GanttThemeOverrides` type for `options.theme`, `setTheme()` and `patchTheme()`: any token may be `undefined` (keeps the built-in value).
- 6522a87: Split panel vs table view: the panel next to the timeline shows `name`, `start`, `end`, `progress`; the table view shows type, a derived status (with a coloured dot), duration, predecessors and, with `baseline`, baseline start/end and variance. `panel.columns` / `table.columns` pick columns by key (built-ins need no definition), `setColumnVisible(key, visible, view)` works per view, sorting reorders siblings at every tree level, and custom columns get `sortValue` and a `ColumnContext`. Visible changes: the default panel drops Duration; Duration reads the drawn span (one day longer with `endDateMode: 'inclusive'`); code building a full `GanttLabels` object needs the new keys.
- c8b61c7: **Accessibility of `<gantry-element>`.** The host is a landmark region instead of a tab stop, and keyboard focus moves
  through the controls inside it.
  
  ### `@gantry/vanilla` (breaking)
  
  - **No `tabindex` on the host.** The element no longer makes itself focusable: the shadow root delegates focus, so
    `el.focus()` focuses the first internal tab stop. The tab stops are,
    in order: the toolbar buttons, the task list (one row at a time: the focused row, else the row focused last while it
    is mounted, else the first row in view; the other rows have `tabindex="-1"`), the splitter, its collapse toggle and the
    timeline viewport (`role="group"`, `tabindex="0"`, named by `labels.ariaChart`; arrow keys scroll it). Each shows a
    focus ring in `--gantt-sel-border` on keyboard focus. A page that set `tabindex` on the element should remove it.
  - **No `data-theme` on the host.** Use the `dark` custom state instead: `gantry-element:state(dark) { … }` replaces
    `gantry-element[data-theme="dark"]` (Chromium 90–124 use the legacy `:--dark`). The host also exposes `:state(dragging)`
    while a bar is dragged or resized and `:state(loading)` while a data-source page loads.
  - **`role="region"` through `ElementInternals`.** The host is a region named by `labels.ariaChart` (kept in sync when the
    labels change) without adding attributes; a `role` or `aria-label` the page sets on the element still wins. Engines
    without ARIA on `ElementInternals` get `role="region"` and `aria-label` host attributes when the element connects.
  - **The splitter's separator is a child of `.gantry-splitter`.** `role="separator"`, `tabindex="0"` and the new
    `aria-valuenow` (the panel width in px), `aria-valuemin` (0 when the panel is collapsible, else `panel.minWidth`) and
    `aria-valuemax` (the widest panel the splitter allows; core `splitterValue()`) moved to `.gantry-splitter-handle`,
    which covers the splitter box; the collapse toggle is its sibling, no longer nested inside the separator (whose
    children are presentational). `::part(splitter)` is still the box. Keyboard handling is unchanged.
  - **Live region.** A polite `.gantt-sr-live` (`role="status"`) announces changes of the number of selected tasks
    (`labels.announceSelection`) and the data source's load state (`labels.loading`, then `labels.announceLoaded(n)` or
    `labels.allLoaded(n)`); a failed page is still announced once in `.gantt-sr-alert`.
  - When the focused control disappears (its row was removed), focus goes to the timeline viewport (in the table view, to
    the task list's tab-stop row) instead of the host. Starting a bar drag focuses the timeline viewport.
  - The toolbar is named by `labels.ariaToolbar`.
  
  ### `@gantry/core`
  
  New labels in `GanttLabels` / `DEFAULT_LABELS`: `ariaToolbar` (`'Chart toolbar'`, the toolbar's accessible name) and
  `announceLoaded(count)` (`'20 tasks loaded'`, read when a data-source page loads and more remain). They are required
  members of `GanttLabels`: only objects typed as a full `GanttLabels` need them, `options.labels` stays partial.
  
  ### `ngx-gantry`
  
  The toolbar is named by `labels.ariaToolbar`, like the Web Component's.

### Patch Changes

- 6522a87: Angular: bars follow their rows after collapse / sort, `selectionChange` is emitted for every selection (task list, keyboard, API), Ctrl/Cmd+A selects all visible rows and honours `enableSelection` / `multiSelect`. Shortcuts are ignored while typing in inputs or `contenteditable` cells. The resolved config gains `selection: { enabled, multi }`.
- d2e5d4c: Bug fixes: tasks with a `null`, empty, unknown or cyclic `parentId` no longer disappear and a task without `end` no longer breaks the date columns (new `normalizeTasks()`); the critical path reads dates like the bars (`endDateMode: 'inclusive'` counts the last day, hour-level tasks use fractional days); column sorting is linear-time.
