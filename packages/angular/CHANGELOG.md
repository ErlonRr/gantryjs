# ngx-gantry

## 2.0.0

### Major Changes

- 639ad08: Accessible default palette (WCAG 2.2 AA): bar labels reach 4.5:1 and bars, markers and arrows 3:1. Labels inside bars switch to a readable colour on custom `barColor`s (`contrastRatio()`, `readableOn()`). The previous colours are exported as `LegacyLightTheme` / `LegacyDarkTheme`.
- 6b74328: Requires Angular 22 (`@angular/core` and `@angular/common` `^22.0.0`) and TypeScript 6.0; Node `^22.12` or `>=24`. `dayjs` is no longer a dependency.
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
- 7bf8c1b: **`ngx-gantry` 2.0: the Angular chart at parity with `<gantry-element>`.** `<gantry>` now renders the same DOM, CSS
  classes, windows and accessibility tree as the Web Component, and is rebuilt on Angular's signal APIs: its state is
  `linkedSignal` / `model()` / `computed` (no syncing effects), browser work starts in `afterNextRender` and stops
  through `DestroyRef` (SSR-safe), custom bars resolve per chart through the injector hierarchy, and the public API is
  curated.
  
  ### Setup and providers (breaking)
  
  - `GanttModule`, `GanttModule.forRoot()`, `provideGanttLib()` and `BUILT_IN_BAR_DEFS` are removed. A `<gantry>` needs
    no provider (`imports: [GantryComponent]`): the built-in bars, the English labels and the light theme are there.
  - `provideGantt(config?, ...features)` (moved out of the theme service) returns `Provider[]` instead of
    `EnvironmentProviders` and works in application (`bootstrapApplication`), route and component `providers`. Each
    level merges its config over the level above: `theme`, `labels` (with `labels.viewModes`) and `formats` (with
    `formats.header`) per key, `darkMode` and `locale` replaced. The nearest level wins; of two `provideGantt` at the
    same level, the last one wins. A chart's `options` still win over all of them.
  - New `withBars(...defs)`: custom bars as a `provideGantt` feature (type `GanttFeature`).
  - `provideGanttBar(...defs)` takes any number of definitions, returns `Provider[]` and works at every level.
  - `GANTT_GLOBAL_CONFIG` and `GANTT_BAR_DEF` are no longer exported; `GANTT_BAR_DEF` has no default value.
  
  ### Custom bars (breaking)
  
  - `GanttBarBase` is an abstract `@Directive()` that declares `ctx = input.required<GanttBarContext>()`. Remove your
    own `ctx` input and the `context` getter. The chart keeps the same `ctx` object while the row's data is unchanged,
    so only the bars whose data changed re-render. `GanttBarInputs` is deleted.
  - The bar registry is per chart: `GanttBarRegistry` is internal and provided by each `<gantry>` (injecting it outside
    a chart throws), and its runtime `register()` is gone: provide the bar with `provideGanttBar` in the component or
    route that shows it. A type resolves to the nearest level that defines it (a level can replace a built-in type);
    an unknown `barType` draws the `task` bar. A bar provided for one page no longer shows up in the others.
  - Bars are direct children of `g.gantt-bars-layer`: no wrapper `<g>` per row and no `g[ganttBarHost]` level. The chart
    creates each bar's `<g>` host in the SVG namespace, stamps `data-task-id` on it (custom bars too), gives it the
    canvas' style scope attribute and dims the dragged bar with a CSS `opacity`. Built-in bars carry `role="listitem"`
    and an accessible name; a custom bar sets its own.
  
  ### Public API (breaking)
  
  - `ngx-gantry` exports `GantryComponent`, `GanttColumnDefDirective`, `provideGantt`, `withBars`, `GanttGlobalConfig`,
    `GanttFeature`, `provideGanttBar`, `GanttBarDef`, `GanttBarBase`, the built-in bars `GanttTaskBarComponent`,
    `GanttMilestoneComponent` and `GanttGroupBarComponent`, `GanttDataSource`, `createGanttDataSource`,
    `GanttDataSourceConfig`, `ColumnDef`, and everything in `@gantry/core`.
  - No longer exported (internal): `GanttArrowComponent`, `GanttHeaderComponent`, `GanttTaskListComponent`,
    `GanttCanvasComponent`, `GanttColumnService`, `GanttThemeService`, `GanttI18nService`, `GanttLayoutService`,
    `GanttSortState`, `GanttScrollService`, `GanttDragService`, `GanttDragPreviewComponent`, `DragState`,
    `GanttSelectionService`, `GanttKeyboardDirective`, `GanttLoadMoreComponent`, `GanttToolbarComponent`,
    `GanttBarRegistry`, `GANTT_BAR_DEF`, `GANTT_GLOBAL_CONFIG`. Drive the chart through `<gantry>`'s inputs, outputs and
    methods.
  - Deleted: `GanttVirtualScrollDirective`, `GanttAnnotationComponent`, `GanttTooltipDirective`, `GanttTooltipContext`,
    `GanttBarInputs`, and the `gantt-math` / `gantt-data` / `gantt-cpm` re-export modules. Their names still come from
    `@gantry/core` through `ngx-gantry`, now as real values: 1.x's type declarations exported 24 of them as type-only
    aliases, so `import { addDays } from 'ngx-gantry'` failed as a value. Fixed: `addDays`, `clamp`,
    `collectDescendants`, `computeCriticalPath`, `dateToPixel`, `diffDays`, `durationLabel`, `filterVisible`,
    `findRowById`, `flattenTree`, `formatDate`, `longestChain`, `pixelToDate`, `reindexVisible`, `snapToDay`,
    `sortRoots`, `SORT_BY_NAME`, `SORT_BY_PROGRESS`, `SORT_BY_START`, `DEFAULT_COLUMNS`, and the types `SortComparator`,
    `TimelineRange`, `CPMResult`, `ColumnCellContext`.
  - Deprecated members removed with them: the `theme` inputs of the task list, load-more footer, toolbar and drag
    preview; the task list's `panelWidthChange`; the drag preview's `barHeight`, `radius` and `unit`; the canvas'
    `selectionChange`; the keyboard directive's `visibleRows`; `GanttScrollService.scrollToDate()` / `scrollToToday()` /
    `scrollToOrigin()` / `visibleRowRange` (use `<gantry>`'s `scrollToDate()`, `scrollToToday()`, `scrollToTask()`);
    `GanttSelectionService.selectedCount` / `hasSelection` / `isSelectedSignal` / `select()` / `toggle()` /
    `getSelectedIds()` / `getSelectedRows()` (use `getSelectedTasks()`, `setSelectedTasks()`, `selectionChange`);
    `GanttLayoutService.layoutParams` / `setTasks()` (its `viewMode` is now `level`); `GanttColumnService.visibleColumns`
    / `gridTemplate` / `totalPanelWidth` / `registerColumn()` / `unregisterColumn()`; `GanttThemeService.init()` /
    `setDarkMode()` / `setOptionsTheme()` / `applyThemeToDom()`; `GanttI18nService.apply()`.
  - `GanttColumnDefDirective`: `<gantry>` reads its columns through a content query (a column inside `@if` or
    `<ng-container>` comes and goes with it; a `ganttColumn` outside `<gantry>`'s content is ignored). `getTemplate()`
    is removed and `cellTemplate` is a signal (`contentChild`).
  - `GanttDataSourceConfig.injector` is removed (the data source never needs an injection context).
  - The task list's own `[tlHeader]` and `[tlFooter]` slots are gone with the public `<gantt-tasklist>` (only a
    standalone task list could fill them). `<gantry>`'s slots are unchanged: `[ganttToolbar]`, `[tlFooter]` (below the
    task list) and `[ganttFooter]`.
  - `GantryComponent` and the inner components have no `ngOnInit` / `ngAfterViewInit` / `ngOnDestroy` methods.
  
  ### Inputs, outputs and methods
  
  - `viewMode` is a `model()`: `[(viewMode)]="zoom"`. `viewModeChange` still fires for the toolbar, the keyboard,
    `zoomIn()` / `zoomOut()` / `setViewMode()`, a restored state and an `options.viewMode` change, but not when the
    parent sets `[viewMode]` and not on the first render; it is typed `ViewMode | (string & {}) | undefined`. A zoom
    level restored from `options.stateKey` when the chart starts is a change too: it updates `[(viewMode)]` and fires
    `viewModeChange` once (as `<gantry-element>`'s `view-mode-change`). Between `[viewMode]` and `options.viewMode` the
    one that changed last wins (a new `options.viewMode` still overrides a user zoom).
  - `darkMode` defaults to `undefined`, so `provideGantt({ darkMode })` now applies (1.x's `false` default overrode it):
    `options.darkMode`, else `[darkMode]`, else `provideGantt({ darkMode })`, else light.
  - Boolean inputs (`showToolbar`, `enableDrag`, `enableResize`, `tooltipEnabled`, `showLegend`) accept attribute
    values: `<gantry showLegend>`, `showToolbar="false"`.
  - `<gantry [visibleRows]>` is removed: the keyboard reads the rows from the chart.
  - New, as on `<gantry-element>`: `view` (the current view, a read-only signal), `resetTheme()` (drops the
    `setTheme()` / `patchTheme()` overrides) and `getBarVisual(id)` (a bar's core geometry and paints, also for rows
    outside the rendered window).
  - A new `series` (or new data-source items) resets local drags as in 1.x. `task.collapsed` seeds only the ids the
    chart has never seen (`seedCollapsed` from `@gantry/core`, the Web Component's rule): an id seen before keeps the
    user's expand / collapse state, also across an empty series in between or a data-source reset, and an id removed
    and added again is not collapsed again.
  - A state change still waiting for its 150 ms debounce (`STATE_SAVE_DELAY`) is written under `options.stateKey` when
    the chart is destroyed, as `<gantry-element>` does when it disconnects (1.x dropped it). `stateChange` does not fire
    then: the chart's outputs are gone.
  - Column resizing captures the pointer (no `window` listeners) and reports a width only when it changed.
  
  ### Host element and theming (breaking)
  
  - The theme tokens are an inline `[style]` binding on the host (every `--gantt-*` token plus the derived contrast
    tokens), rendered on the server too, with no effect and no flash. A consumer `style` attribute is kept. To set a
    token from a template, use a `[style]` map (`[style]="{ '--gantt-bar': color }"`) or the `--global--` prefix
    (`[style.--global--gantt-bar]="color"`): Angular 22 namespaces a plain `[style.--gantt-bar]` binding, which then
    loses to the host map. Page CSS on `gantry` still needs `!important`. Not tested with Angular's
    `provideCssVarNamespacing()`.
  - No `data-theme` attribute: the host has the classes `gantry-dark` (dark theme on), `gantry-dragging` (a bar is
    dragged or resized) and `gantry-loading` (a data-source page is loading). Replace `gantry[data-theme='dark']` with
    `gantry.gantry-dark`.
  - The host's chrome (font, colours, layout) and the constant inline styles of inner elements moved into the
    component stylesheets.
  - The light theme's current-period header label uses `--gantt-today-label` (`#964B13`, 5.01:1; it was `#B45309` at
    3.97:1, below WCAG AA), and theme-coloured progress uses `--gantt-progress-overlay` (theme key `progressOverlay`);
    both tokens come from `@gantry/core`.
  
  ### Accessibility and focus (breaking)
  
  - The host is a `region` landmark named by `labels.ariaChart` (`role` and `aria-label` attributes) and no longer a tab
    stop: no `tabindex`, no host focus ring. The tab stops are the Web Component's: the toolbar buttons, the sortable
    column headers, one task-list row (roving `tabindex`: the focused row, else the row focused last while mounted,
    else the first row in view) with the collapse toggles, the splitter handle, its collapse toggle, and the timeline
    viewport (`.gantry-canvas-viewport`, `role="group"`, `tabindex="0"`, named by `labels.ariaChart`; arrow keys scroll
    it). Focus rings use `--gantt-sel-border`. A page that focused `<gantry>` itself should focus one of these.
  - The splitter's `role="separator"`, `tabindex="0"` and `aria-orientation` moved to its child
    `.gantry-splitter-handle`, which covers the splitter and gains `aria-valuenow` (the panel width in px),
    `aria-valuemin` (0 when the panel is collapsible, else `panel.minWidth`) and `aria-valuemax` (the widest panel the
    splitter allows), the Web Component's values from `splitterValue()` in `@gantry/core`; the collapse
    toggle is its sibling instead of a child of the separator.
  - A focused task-list row stays mounted while it is scrolled out of view (`.gantt-tl-row--pinned`), so focus survives
    scrolling. When the focused control disappears, focus moves to the timeline viewport (in the table view, to the task
    list's tab-stop row); pressing a draggable bar focuses the viewport, so `Escape` cancels the drag.
  - The task list's `aria-multiselectable` follows `enableSelection` and `multiSelect`.
  - Live regions: a polite `.gantt-sr-live` (`role="status"`) announces selection-count changes and the data source's
    load state (`labels.loading`, `labels.announceLoaded(n)`, `labels.allLoaded(n)`); a failed page is announced once in
    `.gantt-sr-alert` (`role="alert"`, `labels.loadError`) — the load-more footer no longer has a `role="alert"` of its
    own.
  
  ### Tooltip (breaking)
  
  - One tooltip per chart, inside `<gantry>` (it inherits the `--gantt-*` tokens) and created on the first hover,
    instead of one popover per bar in `<body>`. Its markup is the Web Component's:
    `div.gantt-tooltip[popover=manual][role=tooltip]` (`gantt-tooltip--critical` for critical tasks) >
    `div.gantt-tooltip-title` + `dl.gantt-tooltip-rows` > `div.gantt-tooltip-row` (`gantt-tooltip-row--progress`) >
    `dt.gantt-tooltip-label` + `dd.gantt-tooltip-value` (+ `dd.gantt-tooltip-track > div.gantt-tooltip-fill`). The
    `gantt-tooltip-popover` class and the inline styles are gone: restyle it with `gantry .gantt-tooltip …`.
  - It opens after 320 ms and closes after 120 ms (`TOOLTIP_SHOW_DELAY` / `TOOLTIP_HIDE_DELAY`), and hides when a drag
    starts or `tooltipEnabled` turns off.
  
  ### DOM and CSS classes (breaking)
  
  - The panes use the Web Component's classes; the old names are dropped: `.gantt-panel-scroll` → `.gantry-panel`,
    `.gantt-timeline-scroll` → `.gantry-canvas-viewport`, `.gantt-timeline-header` → `.gantry-canvas-header-sticky`.
  - Header cells, header bands, non-working bands and vertical grid lines are mounted only around the viewport (the
    viewport ± half its width, in 256 px chunks), as in `<gantry-element>` in the same release: DOM queries see only
    the mounted ones; `.g-header` keeps the full width.
  - The task list uses the Web Component's markup (classed spans, 24 × 24 px collapse targets, the column resize handle
    inside its header cell; see the task-list changeset). The drag ghost label has the class
    `gantt-drag-preview-label`, the load-more sentinel `gantt-load-sentinel`.
  
  ### Rendering and performance
  
  - State lives in signals with the Web Component's equality cut-offs: scrolling inside the overscan or a 256 px chunk,
    re-assigning equal `options` or changing only theme colours re-renders nothing else. No `NgZone`.
  - The timeline model is rebuilt once per local day (per hour on hour scales) instead of every minute; the today line
    still follows the one-minute clock.
  - Only the arrows crossing the rendered rows are drawn (`buildArrowIndex`), each one memoised and keyed by its link,
    so repeated dependencies no longer trigger `NG0955`.
  
  ### Server-side rendering
  
  - SSR-safe: no `window`, `document` or `localStorage` globals (`DOCUMENT`, `Renderer2`). The `ResizeObserver`, the
    load-more `IntersectionObserver`, the clock, the scroll listeners, `matchMedia` and animation frames start after the
    first browser render (`afterNextRender`) and stop through `DestroyRef`. The server HTML has the rows, bars, header
    and theme tokens; with `darkMode: 'auto'` it is light until the browser reports the colour scheme.
  
  ### Migration
  
  ```ts
  // 1.x                                              // 2.0
  bootstrapApplication(App, { providers: [            bootstrapApplication(App, { providers: [
    provideGanttLib({ locale: 'it' }),                  provideGantt({ locale: 'it' }),        // optional
    provideGanttBar({ type: 'epic', component: E }),    provideGanttBar({ type: 'epic', component: E }),
  ]});                                                ]});
  @NgModule({ imports: [GanttModule.forRoot()] })     // standalone: imports: [GantryComponent]
  
  // per page / per chart (new)
  @Component({ providers: [provideGanttBar({ type: 'epic', component: EpicBar })], … })
  // route
  { path: 'plan', providers: [provideGantt({ labels: { legend: 'Legend' } }, withBars(epic))], … }
  
  // custom bar
  export class EpicBar extends GanttBarBase {         export class EpicBar extends GanttBarBase {
    ctx = input.required<GanttBarContext>();            geo = computed(() => this.ctx().geometry);
    override get context() { return this.ctx(); }     }
  }
  inject(GanttBarRegistry).register({…})              // → provideGanttBar({…}) in providers
  
  <gantry [options]="{ viewMode }"                    <gantry [(viewMode)]="zoom" …>
          (viewModeChange)="zoom = $event">
  <gantry [style.--gantt-bar]="c">                    <gantry [style]="{ '--gantt-bar': c }">
  gantry[data-theme='dark'] …                         gantry.gantry-dark …
  body > .gantt-tooltip, .gantt-tooltip-popover       gantry .gantt-tooltip (+ dl / dt / dd classes)
  .gantt-panel-scroll / .gantt-timeline-scroll        .gantry-panel / .gantry-canvas-viewport
  .gantt-timeline-header                              .gantry-canvas-header-sticky
  import { GanttLayoutService, … } from 'ngx-gantry'  not public: use GantryComponent's inputs / methods
  ```
- 6522a87: Breaking changes in low-level APIs: `selectionChange` on `<gantry>` reports every selection (canvas, task list, keyboard, API; the inner components that emitted their own are internal in `ngx-gantry` 2.0); `GanttDataSource.page()` is the last page loaded (0 before the first load) and `createGanttDataSource()` no longer needs an injection context; `GanttLabels` has new required members — `loadError`, `retry`, `colId`, `colType`, `colStatus`, `colPredecessors`, `colBaselineStart`, `colBaselineEnd`, `colVariance`, `statusNotStarted`, `statusInProgress`, `statusOverdue`, `statusDone`, `typeTask`, `typeMilestone`, `typeGroup`, `ariaTaskList`, `dependency(name, type, lagDays)` and `varianceDays(days)` (only objects typed as a full `GanttLabels` need them — `options.labels` stays partial). The default split panel shows `name`, `start`, `end`, `progress` (Duration moved to the table view; restore it with `panel: { columns: ['name', 'start', 'end', 'duration', 'progress'] }`). `ColumnCellContext` gains `ctx`; `<ng-template ganttColumn>` inputs left unset inherit from a built-in column with the same key.
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
- 6522a87: Data sources share one framework-free loader (`PagedLoader` in `@gantry/core`): failed pages retry automatically with exponential backoff (`retries`, `retryDelay`), then show an error with a Retry button, emit `loadError` / `load-error` and pause until `retry()` — no more request storm. `reset()` replaces instead of duplicating, `initialPage` works (`0` = wait for `loadNext()`), and Angular's `GanttDataSource` gains `error()`, `nextPage()` and `retry()` and no longer needs an injection context. New labels `loadError` and `retry`.
- 6522a87: New `initialScroll` option: `'auto'` (default) centres today only when it is near the tasks and otherwise shows the earliest task, so the first view always has bars; also `'today'`, `'start'`, a `Date` or `false`. Zooming now keeps the date at the viewport centre in place, an earlier range start no longer shifts the view, and smooth scrolling honours `prefers-reduced-motion`.
- 19d6e62: Gantry is now available for every framework: `@gantry/core` (framework-free logic), `@gantry/vanilla` (`<gantry-element>` Web Component), and `@gantry/react`, `@gantry/vue`, `@gantry/svelte` bindings. `ngx-gantry` now builds on `@gantry/core` and gains column sorting (built-in columns are sortable).
- c8b61c7: The drag ghost is painted from the model `@gantry/core` computes (`computeDragGhostVisual`), the same one the Web Component
  uses: a dashed outline at 35 % opacity with a translatable movement label (`labels.dragDelta`). The drag preview component is
  internal in `ngx-gantry` 2.0.
- cdcf5cd: `ViewMode`, `DependencyType` and `TaskType` are now `as const` objects with a same-name union type (no TS enums): `ViewMode.Week` reads the same and plain strings are accepted. Optional properties of the public types accept `undefined` explicitly, so projects using `exactOptionalPropertyTypes` can pass `undefined`. `ngx-gantry` declares `"type": "module"`. Task-list rows can be focused and selected with Enter / Space.
  
  New `GanttThemeOverrides` type for `options.theme`, `setTheme()` and `patchTheme()`: any token may be `undefined` (keeps the built-in value).
- 6522a87: Split panel vs table view: the panel next to the timeline shows `name`, `start`, `end`, `progress`; the table view shows type, a derived status (with a coloured dot), duration, predecessors and, with `baseline`, baseline start/end and variance. `panel.columns` / `table.columns` pick columns by key (built-ins need no definition), `setColumnVisible(key, visible, view)` works per view, sorting reorders siblings at every tree level, and custom columns get `sortValue` and a `ColumnContext`. Visible changes: the default panel drops Duration; Duration reads the drawn span (one day longer with `endDateMode: 'inclusive'`); code building a full `GanttLabels` object needs the new keys.

### Patch Changes

- 334435c: Header cells and non-working bands are tracked by position, like the Web Component's keyed layers, and grid lines by index: a rebuilt timeline (new tasks, a zoom, a loaded page) patches them in place instead of re-creating every node, and development builds no longer warn with NG0956.
- 6522a87: Angular: bars follow their rows after collapse / sort, `selectionChange` is emitted for every selection (task list, keyboard, API), Ctrl/Cmd+A selects all visible rows and honours `enableSelection` / `multiSelect`. Shortcuts are ignored while typing in inputs or `contenteditable` cells. The resolved config gains `selection: { enabled, multi }`.
- d2e5d4c: Bug fixes: tasks with a `null`, empty, unknown or cyclic `parentId` no longer disappear and a task without `end` no longer breaks the date columns (new `normalizeTasks()`); the critical path reads dates like the bars (`endDateMode: 'inclusive'` counts the last day, hour-level tasks use fractional days); column sorting is linear-time.
- 7bf8c1b: **One task-list markup for both renderers.** `<gantry-element>` and `<gantry>` render the task list
  with the same elements, classes and colours:
  
  - Every inner span has a class: the header label `gantt-tl-ellipsis`, the sort icon
    `gantt-tl-sort-icon` (`gantt-tl-sort-icon--active` on the sorted column), and the name cell
    `span.gantt-tl-name` > `button.gantt-collapse-btn` | `span.gantt-tl-indent` · `span.gantt-tl-icon` ·
    `span.gantt-tl-ellipsis`. Text colours, sizes and weights are stylesheet rules on the theme tokens:
    header labels and group names `--gantt-text` (groups 600), leaf names and cell texts
    `--gantt-text-sec`, sort and task icons `--gantt-text-muted` (an active sort icon `--gantt-text`).
    Only a task's `barColor` stays an inline colour, on its icon.
  - The column resize handle lies inside its header cell (`right: 0`): all 6 px can be grabbed (the
    header cell clipped half of it before).
  - `@gantry/vanilla`: cell texts use the secondary text colour (`--gantt-text-sec`, was `--gantt-text`),
    the collapse toggle is a 24 × 24 px target (WCAG 2.5.8; it was 20 px wide) and leaf names are
    indented to match (4 px more), and rows with children get the class `gantt-tl-row--group`, as in Angular.
  - `ngx-gantry`: the toolbar group that holds `[ganttToolbar]` content has the classes
    `gantt-tb-slot gantt-tb-group` (in that order; was `gantt-tb-group gantt-tb-slot`), and the task-list
    `grid-template-columns` comes from `columnsGridTemplate()` in `@gantry/core`.
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
- 6522a87: Web Component: focus stays on toolbar buttons, sort headers and collapse toggles across re-renders; bands and vertical grid lines are built once per timeline instead of on every scroll frame (Angular draws them full height too, so scrolling no longer rewrites them); a pending state save is written on disconnect and the data source re-syncs on reconnect.
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
- Updated dependencies [c8b61c7]
- Updated dependencies [c8b61c7]
  - @gantry/core@2.0.0
