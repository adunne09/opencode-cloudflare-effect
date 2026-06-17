# opencode Cloudflare Effect

An opencode configuration repository for building small, personal software on Cloudflare with Effect.

Use this repository by cloning it as a base project or by copying its `.opencode` directory into an existing project. The goal is to give opencode enough local guidance to help you build durable Cloudflare applications with Effect as the TypeScript framework.

## What Is Included

- Cloudflare-focused opencode skills for Workers, Durable Objects, Wrangler, email, Agents SDK, and related platform services.
- Effect-focused guidance for typed errors, services, layers, schemas, and repository conventions.
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

This setup is optimized for software built by and for one person: small tools, durable apps, private workflows, prototypes, and personal systems. Cloudflare provides low-ops deployment primitives, while Effect provides a disciplined TypeScript foundation for reliability as the project grows.
