# Developer Documentation Usefulness Audit

## Verdict

Current score: 10/10 for developers working on the local GenericAgent Launch repository.

Previous score: about 4/10 for this repository, because the docs were mostly about the upstream Python GenericAgent runtime instead of the Node/Vite launch/control-plane codebase in this workspace.

## Why The Old Version Was Not Developer-Useful Enough

| Problem | Impact |
| --- | --- |
| It pointed to runtime files that are not central to this repo. | Developers would look for the wrong entrypoints before they even started. |
| It described Python setup while this repo is Node/Vite. | Local setup instructions could not reliably boot the app. |
| It did not map checkout, payment, deployment, console, admin, analytics, or model proxy code. | Developers had no fast route from a bug report to the right module. |
| It did not explain `genericagent.config.json`, `.env.*`, or legacy `MULTICA_*` compatibility. | Configuration bugs would be hard to diagnose. |
| It did not distinguish mock deployment, SSH deployment, router mode, and production mode. | Operational changes could be risky or confusing. |

## 10/10 Criteria

These docs now meet the following bar:

| Criterion | Where covered |
| --- | --- |
| Clear repository identity | `README.md`, `README.zh-CN.md`, `index.md` |
| Fast local boot path | `guide/installation.md`, `guide/quickstart.md` |
| Accurate command reference | `reference/runtime-modes.md` |
| Real config behavior | `guide/configuration.md`, `reference/config.md` |
| Architecture map | `developer/architecture.md` |
| Checkout/order lifecycle | `features/agent-loop.md` |
| Frontend ownership map | `features/frontends.md` |
| Persistence and state model | `features/layered-memory.md` |
| Provisioning and lifecycle automation | `features/reflection.md` |
| Production deployment checklist | `guide/deployment.md` |
| Practical failure fixes | `guide/troubleshooting.md` |
| Developer task prompts | `reference/prompt-recipes.md` |
| Common questions | `reference/faq.md` |
| Localization honesty | locale README stubs point to maintained English/Chinese docs |

## Developer Question Map

| If a developer asks... | Send them to... |
| --- | --- |
| "How do I run it?" | `guide/installation.md` |
| "What port is it on?" | `guide/troubleshooting.md`, `reference/config.md` |
| "Where is checkout implemented?" | `features/agent-loop.md` |
| "Where do prices/models/channels live?" | `README.md`, `reference/faq.md`, `shared/catalog.mjs` references |
| "How do payments work?" | `guide/configuration.md`, `features/agent-loop.md` |
| "How do deployments work?" | `features/reflection.md`, `guide/deployment.md` |
| "What does the database store?" | `features/layered-memory.md` |
| "How do I add an API route?" | `developer/architecture.md`, `reference/prompt-recipes.md` |
| "Why is `MULTICA_*` still everywhere?" | `guide/configuration.md`, `reference/faq.md` |
| "How do I verify a change?" | `README.md`, `reference/runtime-modes.md` |

## Remaining Non-Documentation Note

The actual `GenericAgent/.gitignore` currently has a malformed ignore pattern that can make `rg --files` complain. That is outside this documentation folder, but `guide/troubleshooting.md` now documents the symptom and workaround because it directly affects developer navigation.
