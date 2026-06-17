---
name: cloudflare-effect-alchemy
description: Use when building Cloudflare Workers/Durable Objects apps with Effect, Alchemy, Bun, D1, R2, or agent-native Cloudflare infrastructure. Captures package choices, stack layout, local dev verification, and common Alchemy Effect pitfalls.
---

# Cloudflare + Effect + Alchemy

Use this skill when starting or fixing a Cloudflare-native app that combines:

- Cloudflare Workers, Durable Objects, D1, R2, or Workers AI/Agents
- Effect for application code and typed handlers
- Alchemy for infrastructure-as-code
- Bun as package manager/runtime for TypeScript stack files

This skill is intentionally practical. It records conventions and failure modes that are easy to rediscover the hard way.

## Documentation Source

- Use `https://v2.alchemy.run` as the primary documentation source for Alchemy + Effect work.
- Prefer `v2.alchemy.run` pages for the Effect-native Stack API, `alchemy/Cloudflare` imports, `Alchemy.Stack`, platform resources, bindings, layers, local development, testing, and typed Cloudflare errors.
- Do not rely on plain `https://alchemy.run` for Effect-native guidance; it may describe the older async/top-level-await Alchemy API or omit the v2 Effect APIs.
- When researching a Cloudflare resource, start with the matching `v2.alchemy.run` docs page, then verify exact installed APIs against `alchemy@next` package types if the docs are incomplete or ambiguous.

## Package Choices

- Use `alchemy@next` for the Alchemy Effect API.
- Do not use the separate `alchemy-effect` npm package unless the project explicitly proves it needs that package.
- Prefer Bun for this stack: set `"packageManager": "bun@<version>"` and commit `bun.lock`.
- Keep `effect` and every installed `@effect/*` package version-aligned.
- Install only the Effect platform package the app actually uses.
- For Bun-driven Cloudflare/Alchemy projects, start with `effect` plus `@effect/platform-bun`.
- Do not add `@effect/platform-node` unless there is Node-specific application code or tooling importing it.
- If Alchemy CLI crashes inside Effect internals after installing floating beta versions, pin Effect packages to a known working version that satisfies Alchemy's peer range.

Known working combination from this methodology:

```json
{
  "dependencies": {
    "alchemy": "next",
    "effect": "4.0.0-beta.74",
    "@effect/platform-bun": "4.0.0-beta.74"
  },
  "packageManager": "bun@1.3.7"
}
```

Treat exact versions as a compatibility baseline, not a permanent rule. Move forward only after `bun run alchemy --version`, typecheck, and local `alchemy dev` all work.

## Scripts

Prefer direct Alchemy scripts under Bun:

```json
{
  "scripts": {
    "dev": "alchemy dev --stage poc",
    "plan": "alchemy plan --stage poc",
    "deploy": "alchemy deploy --stage poc --yes",
    "destroy": "alchemy destroy --stage poc --yes",
    "typecheck": "tsc --noEmit",
    "build": "tsc --noEmit"
  }
}
```

Avoid `pnpm alchemy` or Node-launched Alchemy for TypeScript stack files if it fails with `Unknown file extension ".ts" for alchemy.run.ts`. Running through Bun avoids that class of failure.

## Effect Language Service Setup

Install the Effect language service as part of initial setup so editors and patched typechecks catch Effect-specific mistakes such as floating Effects, layer leaks, unnecessary `Effect.gen`, and misused error handlers.

For Bun projects:

```sh
bun add -d @effect/language-service
```

Add the TypeScript plugin to `tsconfig.json`:

```json
{
  "compilerOptions": {
    "plugins": [
      {
        "name": "@effect/language-service"
      }
    ]
  }
}
```

For Code, Cursor, Zed, NVim, or other TypeScript-LSP based editors, make sure the editor uses the workspace TypeScript version. The plugin must run from the project's installed TypeScript, not the editor-bundled TypeScript.

For build-time diagnostics, patch the local TypeScript installation after dependencies install:

```sh
bunx effect-language-service patch
```

If the project wants this enforced for every developer and CI install, add a prepare script:

```json
{
  "scripts": {
    "prepare": "effect-language-service patch"
  }
}
```

After patching, `tsc --noEmit` should emit Effect language-service diagnostics. A useful smoke test is a file containing an unassigned `Effect.log("Hello world!")`; patched `tsc` should report that the Effect value is neither yielded nor assigned.

## Infrastructure Source Of Truth

- Use Alchemy as the single infrastructure source of truth.
- Do not add `wrangler.jsonc` just because Workers examples use it.
- Add Wrangler only as fallback/transitive tooling when Alchemy does not support a concrete required feature.
- Let Alchemy define Worker config, Durable Object namespaces, D1, R2, secrets, bindings, observability, and routes/domains when supported.

Typical stack file:

```ts
import * as Alchemy from "alchemy"
import * as Cloudflare from "alchemy/Cloudflare"
import * as Effect from "effect/Effect"
import AppWorker from "./src/worker"
import { AppDb, Workspaces } from "./src/resources"

export default Alchemy.Stack(
  "App",
  {
    providers: Cloudflare.providers(),
    state: Cloudflare.state()
  },
  Effect.gen(function* () {
    const worker = yield* AppWorker

    return {
      url: worker.url,
      database: (yield* AppDb).databaseName,
      workspaces: (yield* Workspaces).bucketName
    }
  })
)
```

## Resource Layout

Do not import the stack file from Worker runtime code.

Use this split:

- `alchemy.run.ts`: stack composition only.
- `src/resources.ts`: reusable Alchemy resource declarations and config values.
- `src/worker.ts`: Worker and Durable Object runtime code importing resource handles from `src/resources.ts`.

Why: importing `alchemy.run.ts` from a Worker can pull Alchemy provider/stack code into the Worker bundle. That can introduce Node-only dependencies and local `workerd` failures such as `createRequire(undefined)`.

Example `src/resources.ts`:

```ts
import * as Cloudflare from "alchemy/Cloudflare"

export const AppDb = Cloudflare.D1Database("APP_DB", {
  name: "my-app",
  migrationsDir: "./migrations"
})

export const Workspaces = Cloudflare.R2Bucket("WORKSPACES", {
  name: "my-app-workspaces"
})
```

## Worker Configuration

For Alchemy Effect Worker classes, an explicit main path is often more robust than `import.meta.filename`:

```ts
export default class AppWorker extends Cloudflare.Worker<AppWorker>()(
  "AppWorker",
  {
    name: "my-app",
    main: "./src/worker.ts",
    compatibility: {
      date: "2026-06-02",
      flags: ["nodejs_compat"]
    },
    observability: {
      enabled: true
    }
  },
  // Effect worker shape
) {}
```

If local `alchemy dev` fails with `This Worker requires compatibility date ... but the newest date supported by this server binary is ...`, lower the compatibility date to the newest date supported by the local `workerd` binary or update the dependency set that provides `workerd`.

If local `alchemy dev` fails because `import.meta.filename` becomes `undefined`, use an explicit `main: "./src/worker.ts"` path.

## Effect + Alchemy Binding APIs

Alchemy's Cloudflare APIs are Effect-native. Check return types before yielding.

D1 prepared statements are created synchronously; statement execution returns Effects:

```ts
const db = yield* Cloudflare.D1Connection.bind(AppDb)
const statement = db.prepare("select 1 as ok")
const row = yield* statement.first<{ ok: number }>()
```

Do not write:

```ts
const statement = yield* db.prepare("select 1 as ok")
```

Durable Object SQLite `exec` returns an Effect of a cursor; cursor methods also return Effects:

```ts
const state = yield* Cloudflare.DurableObjectState
const cursor = yield* state.storage.sql.exec<{ status: string }>("select 'ok' as status")
const row = yield* cursor.one()
```

## Durable Object Routing

When a Worker forwards an Effect `HttpServerRequest` to an Alchemy Durable Object binding, keep the request type intact:

```ts
const response = yield* namespace.getByName(name).fetch(request)
```

Do not replace it with a platform `new Request(...)` unless the binding method type accepts a platform request. If the Durable Object needs a different path, either make the DO handler accept the externally routed path or introduce a properly typed adapter after checking Alchemy's current API.

## Local Dev Flow

Expected first-run flow:

1. Run `bun install`.
2. Run `bun run alchemy login --configure` or otherwise configure an Alchemy Cloudflare profile.
3. Run `bun run dev`.
4. Accept the Cloudflare State Store deployment prompt if using `Cloudflare.state()` and the state store does not exist yet.

Local `alchemy dev` may still create or update real Cloudflare resources such as Alchemy state store, D1, R2, and Worker metadata before serving locally. Treat it as a Cloudflare-authenticated dev flow, not a purely offline emulator.

Verify with real HTTP calls once the local URL is printed:

```sh
curl -i http://localhost:1337/health
curl -i http://localhost:1337/api/status
curl -i http://localhost:1337/api/agents/test-thread/health
```

## TypeScript Config

Use bundler-oriented TypeScript settings for Bun and Alchemy TypeScript imports:

```json
{
  "compilerOptions": {
    "target": "ESNext",
    "module": "Preserve",
    "moduleResolution": "Bundler",
    "customConditions": ["bun"],
    "allowImportingTsExtensions": true,
    "rewriteRelativeImportExtensions": true,
    "verbatimModuleSyntax": true,
    "strict": true,
    "types": ["@cloudflare/workers-types", "node"],
    "noEmit": true
  },
  "include": ["alchemy.run.ts", "src/**/*.ts", "types/**/*.d.ts"]
}
```

Avoid ambient declaration shims for Alchemy packages. If types are missing or wrong, first verify that the project is using `alchemy@next`, Bun conditions, and the correct TypeScript module resolution.

## Documentation Conventions

When establishing this stack, document decisions that affect future upgrades:

- Why exact Effect versions are pinned.
- Which package manager is authoritative.
- Whether Alchemy or Wrangler owns infrastructure.
- The local `workerd` compatibility date constraint.
- Why resources are split out of `alchemy.run.ts`.
- What local route smoke tests prove D1/R2/DO bindings work.

These details prevent future agents from “cleaning up” the constraints that keep local dev working.
