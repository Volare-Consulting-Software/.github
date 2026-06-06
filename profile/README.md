<p align="center">
  <img src="./volare-logo.png" alt="Volare Consulting" width="320" />
</p>

# Volare Consulting

> Your ideas taking flight.

Volare is the engineering team you bring on when the team you have can't get to it. Engagements are scoped to ship, not to bill — we commit to outcomes, not hours.

**What you're looking at** is the public slice of how Volare actually works: the brand system every artifact draws from, the AI-driven SDLC engine that ships our engagements, the agent that talks to customers, and the reusable CI/CD that builds and deploys all of it. Each repo below is a real, in-use piece of the platform — not a demo.

## How we build software

A handful of repos here do most of the work. Three of them define how Volare ships, one defines how Volare sounds, and the reusable workflows in *this* repo define how it all gets built and deployed.

### `volare-brand` — how Volare presents itself

The single source of truth for colors, typography, components, voice, naming, and logo. Every Volare-built artifact — proposals, marketing, product UI, customer email — draws from it. One brand, two registers (external and internal), same personality.

### `pilota` — the SDLC engine

Pilota drives a software project end-to-end with a coordinated team of specialist AI agents and a human in the loop at the gates that matter: requirements → PRD → design → walking skeleton → vertical slices → tests → review → merge. It's **language-agnostic** and **integration-agnostic** — works whether you're shipping TypeScript on Vercel through GitHub Actions with Linear tickets, or C# on Azure through Azure Pipelines with Jira. Every spoke (language, CI/CD, deployment, project management, branding, LLM provider) is interchangeable per engagement.

Two design choices set it apart from "AI generated my repo":

- **Three named runtimes** (`cli`, `app-cli`, `sdk`) share one state machine, agent roster, and set of human gates — only *where state lives* and *who drives turn cadence* differ. The same agent prompts run in a local Claude Code session, against a hosted pilota-app over MCP, or inside a long-lived Node service using the Claude Agent SDK.
- **Per-step model selection.** Each agent step binds to whichever model best fits it — heavier reasoning models for architecture, cheap models for classification — declared in `plugin.json`, not a single project-wide LLM.

### `volare-pilota-pack` — Volare's house style for Pilota

Pilota stays generic. `volare-pilota-pack` is the opinionated overlay for Volare-internal engagements — coding conventions, delivery patterns for the GitHub and Azure DevOps stacks, and per-product context. Drops into `~/.pilota/` so any Pilota engagement on Volare work picks up Volare standards automatically. Other Pilota consumers ship their own packs.

### `vito.ai` — Volare's AI agent

Vito is the voice of Volare for chatbots, agentic workflows, and customer-facing surfaces — he inherits brand voice from `volare-brand` and adds character on top. At runtime he's also the **human-in-the-loop layer between Pilota and the people who own the gates**: he receives gate hand-offs from Pilota, routes them to the right operators over a real channel (Telegram first, others behind the same interface), captures *which* operator made each call, and hands the resolution back to Pilota's resume loop. Next.js dashboard + API, channel-agnostic by design.

### Shared CI/CD — the reusable workflows in *this* repo

This `.github` repo isn't just an org profile — it's Volare's central library of **reusable GitHub Actions workflows** (`.github/workflows/*.yml`), called by every service and library repo in the org so build, publish, and deploy logic lives in one place instead of being copy-pasted per project. The lineup:

| Area | Workflows | What they do |
|---|---|---|
| **.NET build & publish** | `dotnet-build-and-test` (action), `build-dotnet-package`, `push-container-image` | Restore/build/test via the composite action, then pack & publish NuGet to GitHub Packages, or containerize a microservice and push to GHCR / GitLab / ACR / ECR. |
| **Node build & publish** | `node-build-and-test` (action), `build-node-package`, `publish-npm-public` | Install/build/test Next.js apps via the composite action, containerize via `push-container-image`; publish internal npm packages to GitHub Packages, or open-source ones to the public npm registry. |
| **Container plumbing** | `push-container-image` | One implementation for both building+pushing an image and promoting (registry-side re-tagging) an existing one across registries. |
| **Azure deploy** | `deploy-azure-app-service`, `deploy-azure-function-app` | Pull a pre-built image from ACR and deploy it to the env-specific App Service / Function App via OIDC, gated through GitHub Environments (`dev` / `staging` / `production`). |
| **Database (EF Core)** | `ef-capture-migration`, `ef-deploy` | Auto-capture migration drift on a push to main; generate and apply idempotent migration scripts on release. |
| **Housekeeping** | `cancel-on-close` | Kill an in-flight build when its PR is admin-merged or closed, so no runner minutes burn on an outcome no one's waiting on. |

These cover the full path from PR to production — build, test, version, containerize, publish, deploy, and migrate — with consistent tagging, environment gating, and loop-safety baked in.

## How they fit together

```
                    volare-brand
                  (voice + visual)
                  ─────────┬──────────
                           │
                           ├──→  vito.ai
                           │     (talks to customers; relays Pilota gates to operators)
                           │
                           └──→  pilota engagements
                                 ⬑ volare-pilota-pack   (Volare standards)
                                 ⬑ shared CI/CD          (this repo's reusable workflows)
                                 (builds + ships software)
```

`volare-brand` is the throughline — it's what makes a Volare proposal, a Pilota-built feature, and a Vito reply all sound recognizably like Volare. Pilota is the engine, `volare-pilota-pack` is Volare's configuration of it, the reusable workflows here are how everything gets built and deployed, and Vito is the voice tying customer conversations back to the work.
