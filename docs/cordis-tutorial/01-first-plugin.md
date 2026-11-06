# 1. Your first plugin

English | [中文](01-first-plugin.zh.md)

In the loader configuration used here, a Cordis plugin module named-exports an `apply` function. When Cordis loads it, it calls `apply` with a **context** — the `ctx` object through which the plugin registers everything it contributes.

## Write the plugin

In your `tmp/cordis-tutorial` directory (see [setup](index.md#setup)), create `hello.ts`:

```ts
import type { Context } from '@deepseek-ai/cordis'

export const name = 'hello'

export function apply(ctx: Context) {
  console.log('hello from my first plugin')
}
```

The `name` export is optional display metadata; it labels the plugin in diagnostics.

## Compose the app

This tutorial's launcher assembles the application from configuration. Create `cordis.yml`:

```yaml
- name: './hello.ts'
```

The file is a list of plugin entries. `name` is a module specifier — a relative path or an npm package name — and the loader mounts every entry. Entries start concurrently, so list position guarantees nothing about which plugin loads first; ordering comes from service dependencies (`inject`, [chapter 3](03-services.md)), not from position in the file.

## Run it

```sh
node --import tsx ../../vendor/cordis/bin.js
```

Expected output:

```
hello from my first plugin
```

The process exits on its own once nothing is left running. What happened:

1. The launcher created a root `Context` and mounted the **Loader** plugin.
2. The Loader read `cordis.yml`, resolved `./hello.ts`, and mounted it as a child plugin.
3. Cordis called your `apply(ctx)`.

There is no framework bootstrap code in your file: a plugin describes what it contributes, and `cordis.yml` composes the application. The [`dsh` base](../../packages/bundle/base/cordis.patch.yml), for example, is a longer plugin composition that deployment overlays patch.

## The two other plugin shapes

A function is the most common form, but Cordis accepts three:

```ts
import { Service, type Context } from '@deepseek-ai/cordis'

// 1. Function plugin (what you just wrote).
export function apply(ctx: Context) {}

// 2. Object plugin: an object with an `apply` method.
export const objectPlugin = {
  name: 'object-plugin',
  apply(ctx: Context) {},
}

// 3. Class plugin: a Service subclass (covered in chapter 3).
export class MyService extends Service {
  constructor(ctx: Context) {
    super(ctx, 'myTutorialService')
  }
}
```

Use the function form until you need to expose a service; [chapter 3](03-services.md) covers when the class form earns its place.

## Try breaking it

Make `apply` throw:

```ts ignore-check
export function apply(ctx: Context) {
  throw new Error('apply exploded')
}
```

Run again: the terminal prints nothing and the process exits 0. The failure is still recorded — the fiber enters `FAILED` and reports through the Cordis logger service — but this tutorial's launcher mounts no console exporter, so the report stays in the logger's in-memory buffer instead of reaching your terminal.

The same reporting path covers a config entry whose module cannot be **resolved** — a typo'd path or package name — so both failures look like nothing happened. If a freshly added entry seems to do nothing, check the spelling first. To see these reports, mount `@deepseek-ai/cordis-plugin-logger-console` or register your own exporter with `ctx.logger.exporter()`.

Next: [Lifecycle and effects](02-lifecycle-and-effects.md) — what happens when a plugin unloads.

[![](https://img.shields.io/badge/powered_by-dsh-4D6BFE?style=flat-square&logo=deepseek&logoColor=white)](https://github.com/deepseek-ai/deepseek-harness)
