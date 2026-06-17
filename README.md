# opencode Cloudflare Effect

An opencode configuration repository for building small, personal software on Cloudflare with Effect and Alchemy.

Use this repository by cloning it as a base project or by copying its `.opencode` directory into an existing project. The goal is to give opencode enough local guidance to help you build durable Cloudflare applications where Cloudflare is the platform, Effect is the TypeScript framework, and Alchemy is the infrastructure bridge that keeps deployment resources and application code in one explicit system.

## What Is Included

- Cloudflare-focused opencode skills for Workers, Durable Objects, Wrangler, email, Agents SDK, and related platform services.
- Effect-focused guidance for typed errors, services, layers, schemas, and repository conventions.
- Alchemy-focused guidance for defining Cloudflare infrastructure alongside application code instead of treating infrastructure as a separate concern.
- An `effect` reference to the Effect source repository for current APIs and migration guidance.
- Slash commands for focused project workflows.

## Slash Commands

- `/migrate-framework` starts an interactive flow for migrating an existing Cloudflare project toward Effect.

The migration command intentionally focuses on framework migration. It assumes the project already runs on Cloudflare and does not try to migrate infrastructure providers.

## Usage

Clone this repository into a new project:

```sh
mkdir my-project && cd my-project && git init && git submodule add https://github.com/adunne09/opencode-cloudflare-effect .opencode-config && git submodule update --init --recursive .opencode-config && ln -s .opencode-config/.opencode .opencode && opencode
```

Install the opencode configuration into an existing project:

```sh
(git rev-parse --is-inside-work-tree >/dev/null 2>&1 || git init) && { test -e .opencode-config/.git || git submodule add --force https://github.com/adunne09/opencode-cloudflare-effect .opencode-config; } && git submodule update --init --recursive --remote .opencode-config && { test -e .opencode || ln -s .opencode-config/.opencode .opencode; }
```

This installs the configuration repository as a Git submodule at `.opencode-config` and exposes its `.opencode` directory through a local `.opencode` symlink. If you improve the shared config from inside another project, commit and push from `.opencode-config` to update the upstream configuration repository.

Restart opencode after copying or editing configuration files so the new commands and skills are loaded.

## Philosophy

This setup is optimized for software built by and for one person: small tools, durable apps, private workflows, prototypes, and personal systems.

The approach depends on three pieces working together:

- Cloudflare provides low-ops deployment primitives: Workers, Durable Objects, D1, R2, KV, Queues, email, AI, and the rest of the platform surface.
- Effect provides a disciplined TypeScript foundation for typed errors, dependency injection, services, layers, schemas, retries, concurrency, and explicit failure handling as the project grows.
- Alchemy ties the infrastructure provider to the application code. Infrastructure is not a distant Terraform folder or a dashboard checklist; it is part of the same TypeScript system that defines the app. That tighter coupling makes bindings, resources, permissions, deployment assumptions, and provider errors more explicit, easier for opencode to reason about, and harder to accidentally ignore.

The result is a stack where application logic, runtime dependencies, and cloud resources can be handled as one coherent program. Cloudflare gives the primitives, Effect makes failure and dependency boundaries explicit, and Alchemy keeps the infrastructure shape close enough to the code that mismatches surface early instead of becoming production surprises.
