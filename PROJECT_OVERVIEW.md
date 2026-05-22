# Kanri — Project Overview

## High-Level Architecture

- **Desktop Kanban app** — offline-first, built with Tauri v2 (Rust backend) + Nuxt 4 (Vue 3 frontend)
- **Frontend**: Nuxt 4 SPA (SSR disabled via `ssr: false`), Vue 3.5, TypeScript, Pinia state, Tailwind CSS
- **Backend**: Tauri v2 Rust shell, no custom Rust commands — all Tauri plugin APIs used from frontend
- **Persistence**: Tauri Store plugin (`@tauri-apps/plugin-store`, `LazyStore(".kanri.dat")`) — JSON file on disk
- **Desktop shell**: Tauri v2 window, single-instance lock, autostart plugin, window state save/restore
- **No HTTP server, no database** — fully local, no network calls

## Important Directories

| Path | Purpose |
|------|---------|
| `pages/` | Nuxt page routes (index, kanban/[id], settings, import, licenses) |
| `components/` | Vue components: `Sidebar`, `PinnedBar`, `PinnedItem`, `Dropdown`, `Tooltip`, `Modal`, `HexColorInput`, `LanguageSelector`, `CustomThemeEditor` |
| `components/kanban/` | Kanban-specific: `Column`, `Card`, `SearchBar`, `ZoomAdjustment`, `TagEdit`, `TagDisplay`, `KanbanDescriptionEditor`, `BoardPreview`, `MoveTo` |
| `components/modal/` | Modal components: `EditCard`, `NewBoard`, `RenameBoard`, `Confirmation`, `Help`, `CustomBackground`, `CardTags`, `Changelog` |
| `components/icon/` | SVG icon components |
| `stores/` | Pinia stores: `boards`, `theme`, `settings`, `layout`, `tauriStore` |
| `composables/` | Vue composables: `useBoard`, `useBackgroundImage`, `useContextMenuClasses` |
| `utils/` | Utility modules: `colorUtils`, `themes`, `sorting`, `drag-n-drop`, `emitter`, `idGenerator`, `exampleData`, `iconManager`, `objects` |
| `types/` | TypeScript type declarations: `kanban-types.d.ts`, `json-schemas.ts` (Zod validation) |
| `plugins/` | Nuxt plugins: `directives`, `errorHandler`, `importedDirectives` (v-calendar, vue-dragscroll) |
| `i18n/` | Internationalization: 11 locale JSON files, config |
| `assets/css/` | Global CSS: `global.css`, `scrollbars.css` |
| `layouts/` | Nuxt layout: `default.vue` |
| `test/` | Vitest setup and test files |
| `src-tauri/` | Rust Tauri backend: `src/lib.rs` (main Tauri builder), `src/main.rs` (entrypoint), `Cargo.toml`, `tauri.conf.json` |
| `src-tauri/capabilities/` | Tauri permission capabilities |
| `scripts/` | Build scripts: `embed-license-reports.js` |
| `eslint-rules/` | Custom ESLint rule (`no-store-outside-stores`) |

## Frontend / Backend Boundaries

- **Frontend (Vue/Nuxt SPA)** handles all UI, state, drag-and-drop, keyboard shortcuts
- **Backend (Tauri/Rust)** provides:
  - File system access (`@tauri-apps/plugin-fs`)
  - File dialogs (`@tauri-apps/plugin-dialog`)
  - Persistent key-value store (`@tauri-apps/plugin-store`)
  - Window state save/restore (`tauri-plugin-window-state`)
  - Single-instance lock (`tauri-plugin-single-instance`)
  - Autostart (`@tauri-apps/plugin-autostart`)
  - OS info (`@tauri-apps/plugin-os`)
  - Logging (`@tauri-apps/plugin-log`)
  - External URL opener (`@tauri-apps/plugin-opener`)
- **No custom Tauri IPC commands** — all interaction via plugin APIs
- Test setup mocks `window.__TAURI_IPC__` for headless testing

## State Management (Pinia)

| Store | File | Key State |
|-------|------|-----------|
| `useBoardsStore` | `stores/boards.ts` | `boards[]`, `pins[]`, `initialized`. CRUD for boards, columns, cards, tags. Auto-save via `$subscribe` (100ms debounced). |
| `useThemeStore` | `stores/theme.ts` | `activeTheme`, `colors` (Theme object), `savedCustomTheme`, `autoThemeEnabled`. |
| `useSettingsStore` | `stores/settings.ts` | Locale, animations, autostart, spellcheck, column zoom, card count, due dates, board sorting. |
| `useLayoutStore` | `stores/layout.ts` | Sidebar state (help modal, back arrow, add button), changelog version tracking. |
| `useTauriStore` | `stores/tauriStore.js` | Wraps `LazyStore(".kanri.dat")` — single Tauri store instance used by all other stores. |

**Important**: The `tauriStore` is the **only** persistence layer. All stores read/write through it. There is no separate DB.

Data flow: Pinia store → `tauriStore.set()` → `.kanri.dat` JSON file. Loaded on app init.

## Routing (Nuxt pages)

| Route | File | Purpose |
|-------|------|---------|
| `/` | `pages/index.vue` | Board list with search, sort, preview cards; CRUD board actions |
| `/kanban/:id` | `pages/kanban/[id].vue` | Kanban board view: columns, cards, drag/drop, edit modals |
| `/settings` | `pages/settings.vue` | Theme selection, locale, autostart, zoom, card display settings |
| `/import` | `pages/import.vue` | Import/export: Kanri JSON, Trello JSON, Kanban Electron JSON |
| `/licenses` | `pages/licenses/` | Open source license information |

## Important Abstractions

### Type Hierarchy (`types/kanban-types.d.ts`)
```
Board → columns: Column[], globalTags?: Tag[], background?: BackgroundSettings
Column → cards: Card[], title, id
Card → name, description?, tasks?: Task[], tags?: Tag[], dueDate?, color?
Tag → id?, text, style?, color?
Task → name, finished, id?
Theme → accent, bgPrimary, elevation1-3, text, textD1-4, etc.
```

### Composables
- **`useBoard(id)`** — main interface for board ops. Wraps `useBoardsStore`. Returns reactive `board`, `isPinned`, and CRUD methods for columns/cards/tags. Used directly in `pages/kanban/[id].vue`.
- **`useBackgroundImage(boardContent)`** — manages custom board background images. Handles file existence checks, `convertFileSrc`, blur/brightness CSS vars, title text color contrast.
- **`useContextMenuClasses()`** — returns computed CSS classes for Radix context menus.

### Event Bus
- `utils/emitter.ts` — typed `mitt` event bus. Events: `createBoard`, `columnActionDone`, `enableColumnCardAddMode`, `enableColumnTitleEditing`, `openBoardDeleteModal`, `openBoardRenameModal`, `openModalWithCustomDescription`, etc.

### Drag and Drop
- `vue3-smooth-dnd` library for column and card reordering
- `utils/drag-n-drop.js` — `applyDrag()` function processes `smooth-dnd` drop results
- `vue-dragscroll` for horizontal scrolling of kanban columns

### Theme System
- 4 built-in themes defined in `utils/themes.ts` (dark, light, catppuccin, custom)
- CSS custom properties via computed style on `.default-layout` — all colors are CSS vars
- Theme persistence through Tauri store
- Auto-theme follows system preference via `@vueuse/core` `useDark`

### Zod Schemas
- `types/json-schemas.ts` — Zod schemas for: `kanriBoardSchema`, `kanriThemeSchema`, `kanriJsonSchema` (full export), `kanbanElectronJsonSchema`, `trelloJsonSchema`
- Used for import validation

### Custom ESLint Rule
- `eslint-rules/no-store-outside-stores.js` — warns if `store.get()`/`store.set()` is called outside `stores/` directory

## Dev / Build / Test Commands

```bash
yarn dev              # Nuxt dev server (http://localhost:3000)
yarn generate         # Nuxt static generation -> .output/public/
yarn preview          # Nuxt preview of generated output
yarn lint             # ESLint with --fix
yarn test             # Vitest (via nuxt/test-utils)

npm run tauri dev     # Full Tauri dev (frontend + Rust window)
npm run tauri build   # Production Tauri build
```

## Important Dependencies

| Package | Purpose |
|---------|---------|
| `nuxt` 4 | Meta-framework for Vue |
| `@nuxtjs/tailwindcss` 6 | Tailwind CSS integration |
| `@pinia/nuxt` | Pinia state management |
| `@nuxtjs/i18n` | Internationalization |
| `@vueuse/core` + `@vueuse/nuxt` | Vue utilities (useDark, useConfirmDialog, etc.) |
| `radix-vue` | Headless UI primitives (dropdowns, context menus, tabs) |
| `@tauri-apps/api` + plugins | Tauri desktop APIs |
| `@tauri-apps/plugin-store` | Persistent key-value store |
| `@tauri-apps/plugin-fs` | File system access |
| `@tauri-apps/plugin-dialog` | File open/save dialogs |
| `vue3-smooth-dnd` | Drag & drop for columns and cards |
| `vue-dragscroll` | Horizontal scroll drag |
| `@tiptap/*` | Rich text description editor for cards |
| `v-calendar` | Date picker for card due dates |
| `@phosphor-icons/vue` | Icon set |
| `@heroicons/vue` | Icon set |
| `@paralleldrive/cuid2` | Unique ID generation (used in all entities) |
| `mitt` | Typed event emitter |
| `zod` | Schema validation for imports |
| `radix-vue` | Context menu, dropdown menu, tabs primitives |

## Common Edit Locations

- **Adding a new store property**: `stores/settings.ts` (add to state + loadSettings + setter). Add to `kanriJsonSchema` in `types/json-schemas.ts` for export compatibility.
- **Adding a new page**: create file in `pages/`, add link in `Sidebar.vue`
- **Modifying kanban interactions**: `components/kanban/Column.vue` (column layout), `components/kanban/Card.vue` (card rendering), `pages/kanban/[id].vue` (board orchestration)
- **Adding a new locale**: add JSON file in `i18n/locales/`, add entry in `nuxt.config.ts` i18n config
- **Board CRUD logic**: `stores/boards.ts` (data ops), `composables/useBoard.ts` (convenience wrapper)
- **Modifying modals**: `components/modal/` — each modal is a self-contained component
- **Theme/tailwind changes**: `utils/themes.ts` (theme definitions), `tailwind.config.js` (animations), `layouts/default.vue` (CSS variable mapping)
- **Import/export**: `pages/import.vue` (UI + logic), `types/json-schemas.ts` (validation schemas)
- **Tauri backend changes**: `src-tauri/src/lib.rs` (plugins), `src-tauri/tauri.conf.json` (window config, CSP, bundle)
- **Testing**: add `.spec.ts` next to page files or in `test/`, mock Tauri IPC via `window.__TAURI_IPC__ = vi.fn()`
