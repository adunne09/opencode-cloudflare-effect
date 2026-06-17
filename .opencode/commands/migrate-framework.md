---
description: Migrate an existing Cloudflare project framework toward Effect.
---

Help migrate this existing project to Effect while keeping the current Cloudflare infrastructure provider and deployment target.

Assume Cloudflare is already the platform. Do not plan or perform an infrastructure-provider migration as part of this command.

Start interactively by asking the user these questions before making changes:

- What framework or application structure is used today?
- What Cloudflare target is already deployed?
- Which routes, modules, or workflows are most painful today?
- What behavior must not change during the migration?
- What tests or manual checks prove the migration worked?
- Do you want an incremental migration or a focused first slice?

After the user answers, inspect the repository. Prefer an incremental migration that introduces Effect at the application boundaries first, preserves deployed behavior, and keeps Cloudflare configuration stable. Avoid broad rewrites. Implement the smallest safe slice.
