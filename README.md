# AgeVolt Creator

Public-safe Git marketplace for AgeVolt AI artifact creation governance.

This repository is intentionally small and public-safe. It contains the Creator Intake plugin, its public-safe skills, KB, and MCP build rules needed for Codex marketplace updates. Secrets, private server configs, examples with customer data, fixtures, and migration work remain in the SharePoint folder:

```text
AI Agent/marketplaces/agevolt-creator-marketplace/
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
AgeVolt/agevolt-creator-marketplace
```

Then install the `Creator Intake` plugin.

CLI equivalent:

```powershell
codex plugin marketplace add AgeVolt/agevolt-creator-marketplace
codex plugin add creator-intake@creator
```

## Upgrade

Because this is a Git marketplace, Codex can upgrade it through the normal marketplace upgrade flow.

```powershell
codex plugin marketplace upgrade creator
```

## Public-Safe Rules

This repo may contain:

- plugin manifests,
- public-safe Creator skills,
- public-safe Creator KB and MCP build rules,
- public install and test instructions.

This repo must not contain:

- internal KB,
- customer data,
- secrets or credentials,
- internal exports,
- copied legacy `AI/` content,
- unpublished product, finance, support, or production details.
