# Little Shelf · Vue Reading List

A Vue 3 + Vite starter for a personal reading list. Includes adding books, author labels, search, status filters, completion tracking, removal, and browser-local saving. Built with the Composition API and a single-file component.

Vue 阅读清单模板：添加书籍、搜索、筛选、标记已读、删除及本地保存。

## Run later

Requires Node.js `^20.19.0 || >=22.12.0` and npm. The following commands are instructions for future use; they were not executed when preparing this template.

```sh
npm install
npm run dev
```

To produce and preview a static build later:

```sh
npm run build
npm run preview
```

## Files

- `src/App.vue`: UI, reading-list state, and scoped application logic.
- `src/main.js`: Vue application entry.
- `vite.config.js`: Vue plugin and relative asset base.
- `package.json`: pinned framework and build dependencies, with npm scripts.

## Customize

Edit the CSS variables and sample books in `src/App.vue`. Replace the sample copy with your own content. This small starter does not need a router, state-management library, or backend.

## Storage

Up to 500 books are saved to this browser's `localStorage`. There is no synchronization or backup. Clearing browser data can remove the list. Storage failures are reported on the page. Sample titles and authors are demonstration content. Use localhost or HTTPS so `crypto.randomUUID()` is available.

Dependencies were not installed, no lockfile was generated, and tests, builds, and runtime checks were not run.
