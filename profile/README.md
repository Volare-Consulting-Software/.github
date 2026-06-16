<p align="center">
  <img src="./volare-logo.png" alt="Volare Consulting" width="320" />
</p>

# Volare Consulting

> Your ideas taking flight.

Volare is the engineering team you bring on when the team you have can't get to it. Engagements are scoped to ship, not to bill — we commit to outcomes, not hours.

🌐 **[go-volare.com](https://go-volare.com)** — what we build, how we work, and how to start a project.

**What you're looking at** is the public slice of how Volare actually works: the brand system every artifact draws from, the AI-driven SDLC engine that ships our engagements, the agent that talks to customers, and the reusable CI/CD that builds and deploys all of it. Each repo below is a real, in-use piece of the platform — not a demo.

## How we build software

A handful of repos here do most of the work. Several define how Volare ships, one defines how Volare sounds, and the reusable workflows in *this* repo define how it all gets built and deployed.

### `volare-brand` — how Volare presents itself

The single source of truth for colors, typography, components, voice, naming, and logo. Every Volare-built artifact — proposals, marketing, product UI, customer email — draws from it. One brand, two registers (external and internal), same personality.

### `pilota` — the SDLC engine

Pilota is the AI engine Volare uses to deliver engagements: a coordinated team of specialist agents does the work between checkpoints, and **a person signs off at every gate**. AI is fast but it gets things wrong — so the calls that matter (what to build, what's good enough, what ships) always belong to a human. It's **language-** and **integration-agnostic** too: the same engine ships TypeScript on Vercel or C# on Azure, with your tickets, your cloud, your stack.

Every project moves through the same stages, and **the middle of it is a loop** — Pilota builds a slice, a human reviews it, and the cycle repeats until the product is done:

```
                          ┌──── repeat for each slice ────┐
                          │                               │
   Plan ──▶ Skeleton ──▶ Slice ──▶ Review ────────────────┘──▶ Ship
    (👤)      (👤)        (👤)       (👤)                       (👤)

   (👤) = human gate — AI does the work between gates; people approve what advances
```

- **Plan it** — what to build, who it's for, and what *done* looks like — agreed, and signed off by a human, before any code is written.
- **Build it small first** — a working version of the whole thing on day one: real routes, real data, real deploys. Not a mockup, not a slide.
- **Grow it in usable pieces, iteratively** — Pilota builds one slice, a human reviews it, then it builds the next. That build-review loop repeats slice by slice, each pass adding a feature you can demo or ship — the product grows one approved step at a time, on a schedule that holds.
- **Review every step** — humans stay in the loop at every gate, deciding what's good enough and what ships next. Quality is never an afterthought, and nothing advances past a checkpoint no one approved.

→ See how we work, in plain terms: **[go-volare.com/pilota](https://go-volare.com/pilota)**

### `pilota-app` — the hosted runtime for Pilota

Pilota is the engine; `pilota-app` is where you watch it run and steer it. A Next.js web app that turns a live engagement into a Linear-style project tracker — every stage and every gate visible as it happens — and routes approvals to the right people (through Vito) so a project never stalls waiting on a decision. The engine decides *what* happens next; `pilota-app` is *where the work shows up and the humans steer*.

### `volare-ai` — Volare's house style for Pilota

Pilota stays generic. `volare-ai` is the opinionated overlay for Volare-internal engagements — coding conventions, delivery patterns for the GitHub and Azure DevOps stacks, and per-product context. Drops into `~/.pilota/` so any Pilota engagement on Volare work picks up Volare standards automatically. Other Pilota consumers ship their own packs.

### `vito.ai` — Volare's AI agent

Vito is the voice of Volare for chatbots, agentic workflows, and customer-facing surfaces — he inherits brand voice from `volare-brand` and adds character on top. He's also the **human-in-the-loop layer between Pilota and the people who own the gates**: when a project needs a decision, Vito brings it to the right person on the channel they already use and carries their answer back. Channel-agnostic by design.

→ Meet Vito: **[go-volare.com/vito](https://go-volare.com/vito)**

### Shared CI/CD — the reusable workflows in *this* repo

This `.github` repo isn't just an org profile — it's Volare's central library of **reusable GitHub Actions workflows** (`.github/workflows/*.yml`), called by every service and library repo in the org so build, publish, and deploy logic lives in one place instead of being copy-pasted per project. The lineup:

| Area | Workflows | What they do |
|---|---|---|
| **.NET build & publish** | `dotnet-build-and-test` (action), `build-dotnet-package`, `push-container-image` | Restore/build/test via the composite action, then pack & publish NuGet to GitHub Packages, or containerize a microservice and push to GHCR / GitLab / ACR / ECR. |
| **Node build & publish** | `node-build-and-test` (action), `build-node-package`, `publish-npm-public` | Install/build/test Next.js apps via the composite action, containerize via `push-container-image`; publish internal npm packages to GitHub Packages, or open-source ones to the public npm registry. |
| **Container plumbing** | `push-container-image` | One implementation for both building+pushing an image and promoting (registry-side re-tagging) an existing one across registries. |
| **Release staging** | `draft-release` | Drafts a v-tagged release per main-push build with auto-generated notes; publishing the release is what ships. |
| **Azure deploy** | `deploy-azure-app-service`, `deploy-azure-function-app` | Pull a pre-built image from ACR and deploy it to the env-specific App Service / Function App via OIDC, gated through GitHub Environments (`dev` / `staging` / `production`). |
| **AWS deploy** | `deploy-aws-lambda` | Point an existing container Lambda at an image already in ECR (the released version tag) via OIDC and wait for the update, gated through GitHub Environments. |
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
                                 ⬑ pilota-app           (hosted runtime + dashboard)
                                 ⬑ volare-ai            (Volare standards)
                                 ⬑ shared CI/CD         (this repo's reusable workflows)
                                 (builds + ships software)
```

`volare-brand` is the throughline — it's what makes a Volare proposal, a Pilota-built feature, and a Vito reply all sound recognizably like Volare. Pilota is the engine, `pilota-app` is the hosted runtime that drives it, `volare-ai` is Volare's configuration of it, the reusable workflows here are how everything gets built and deployed, and Vito is the voice tying customer conversations back to the work.
