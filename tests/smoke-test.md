# Rychly Test

## Instalacia

```powershell
codex plugin marketplace add AgeVolt/agevolt-creator-marketplace
codex plugin add creator-intake@creator
```

Ocakavanie:

- marketplace sa zobrazi ako `creator`
- plugin sa zobrazi ako `creator-intake`
- skill sa zobrazi ako `creator-intake`

## Test Spravania

```text
Pouzi Creator Intake a sprav mi skill na triedenie mailov.
```

Ocakavanie:

- nevytvori sa priamo firemny skill pre vsetkych
- odpoved si najprv vypyta publikum/priklady/riziko zapisu alebo vrati navrh artefaktu
- ak lokalny SharePoint `AI Agent` existuje, skill cita interne Creator pravidla

## Aktualizacia

1. bumpni verziu v `plugins/creator-intake/.codex-plugin/plugin.json`
2. push do `main`
3. spusti:

```powershell
codex plugin marketplace upgrade creator
```

Ocakavanie:

- marketplace snapshot sa aktualizuje
- plugin cache sa aktualizuje na novu verziu
