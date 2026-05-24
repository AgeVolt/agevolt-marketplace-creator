# Smoke Test

## Install

```powershell
codex plugin marketplace add AgeVolt/agevolt-marketplace-creator
codex plugin add creator-intake@agevolt-marketplace-creator
```

Expected:

- marketplace appears as `agevolt-marketplace-creator`
- plugin appears as `creator-intake`
- skill appears as `creator-intake`

## Behavior Prompt

```text
Pouzi Creator Intake a sprav mi skill na triedenie mailov.
```

Expected:

- no direct company-wide skill is created
- response first asks for audience/examples/write risk or returns an artifact proposal
- if local SharePoint `AI Agent` exists, the skill reads internal creator rules

## Upgrade

1. bump `plugins/creator-intake/.codex-plugin/plugin.json` version
2. push to `main`
3. run:

```powershell
codex plugin marketplace upgrade agevolt-marketplace-creator
```

Expected:

- marketplace snapshot updates
- plugin cache updates to the new version
