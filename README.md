# opencode Cloudflare Effect

An opencode configuration repository for building small, personal software on Cloudflare with Effect.

Use this repository by cloning it as a base project or by copying its `.opencode` directory into an existing project. The goal is to give opencode enough local guidance to help you build durable Cloudflare applications with Effect as the TypeScript framework.

## What Is Included

- Cloudflare-focused opencode skills for Workers, Durable Objects, Wrangler, email, Agents SDK, and related platform services.
- Effect-focused guidance for typed errors, services, layers, schemas, and repository conventions.
- Slash commands for focused project workflows.

## Slash Commands

- `/migrate-framework` starts an interactive flow for migrating an existing Cloudflare project toward Effect.

The migration command intentionally focuses on framework migration. It assumes the project already runs on Cloudflare and does not try to migrate infrastructure providers.

## Usage

Clone this repository into a new project:

```sh
git clone https://github.com/adunne09/opencode-cloudflare-effect my-project
cd my-project
opencode
```

Install only the opencode configuration into an existing project:

```sh
tmp=$(mktemp -d) && git clone --depth 1 --filter=blob:none --sparse https://github.com/adunne09/opencode-cloudflare-effect "$tmp" && git -C "$tmp" sparse-checkout set .opencode && cp -R "$tmp/.opencode" . && rm -rf "$tmp"
```

Restart opencode after copying or editing configuration files so the new commands and skills are loaded.

## Philosophy

This setup is optimized for software built by and for one person: small tools, durable apps, private workflows, prototypes, and personal systems. Cloudflare provides low-ops deployment primitives, while Effect provides a disciplined TypeScript foundation for reliability as the project grows.
