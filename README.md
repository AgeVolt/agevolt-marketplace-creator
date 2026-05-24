# AgeVolt Marketplace Creator

Public-safe Git marketplace for AgeVolt AI artifact creation governance.

This repository is intentionally small. It contains only bootstrap plugin metadata and a minimal Creator Intake skill. Internal rules, examples, knowledge base, fixtures, and migration work remain in the SharePoint folder:

```text
AI Agent/
```

## What This Marketplace Is For

Use this marketplace for AI admins, module owners, and power users who create or maintain AgeVolt AI marketplaces, plugins, skills, KB, MCP servers, rules, and scripts.

Regular employees do not need this marketplace just to use finished business plugins.

## Plugin In This MVP

```text
creator-intake
```

`creator-intake` decides what kind of artifact a request should become before anything is implemented. It should produce an artifact proposal for unclear requests such as:

- "sprav skill"
- "vytvor marketplace"
- "urob plugin"
- "potrebujem MCP"
- "premigruj AI"

## Install

In Codex, add this Git marketplace:

```text
AgeVolt/agevolt-marketplace-creator
```

Then install the `Creator Intake` plugin.

CLI equivalent:

```powershell
codex plugin marketplace add AgeVolt/agevolt-marketplace-creator
codex plugin add creator-intake@agevolt-marketplace-creator
```

## Upgrade

Because this is a Git marketplace, Codex can upgrade it through the normal marketplace upgrade flow.

```powershell
codex plugin marketplace upgrade agevolt-marketplace-creator
```

## Public-Safe Rules

This repo may contain:

- plugin manifests,
- minimal bootstrap skills,
- public install and test instructions.

This repo must not contain:

- internal KB,
- customer data,
- secrets or credentials,
- internal exports,
- copied legacy `AI/` content,
- unpublished product, finance, support, or production details.
