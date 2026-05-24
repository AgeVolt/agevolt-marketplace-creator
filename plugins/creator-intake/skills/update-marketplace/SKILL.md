---
name: update-marketplace
description: Pouzi ked pouzivatel chce pridat alebo upravit plugin, skill, knowledge base, MCP, script alebo iny artefakt v existujucom AgeVolt marketplace a treba spravne upravit SharePoint aj Git marketplace tak, aby si ostatni pouzivatelia vedeli zmenu aktualizovat cez Codex upgrade.
---

# Update Marketplace

Tento skill je pre update existujuceho AgeVolt marketplace. Pouzi ho pri poziadavkach typu "pridaj plugin", "pridaj skill", "dopln KB", "zapoj MCP", "uprav existujuci plugin", "publikuj update" alebo "nech si to vsetci vedia upgradnut".

## Najprv Nacitaj Kontext

Najdi `AI Agent` root:

1. `AGEVOLT_AI_AGENT_ROOT`, ak existuje.
2. `%USERPROFILE%\OneDrive - AgeVolt Slovakia, s.r.o\Dokumenty - Produkt\AI Agent`.
3. Aktualny workspace alebo jeho rodic, ak sa vola `AI Agent`.

Ak root existuje, precitaj:

- `README.md`
- `marketplaces/agevolt-creator-marketplace/plugins/creator-intake/kb/rules.md`
- `marketplaces/agevolt-creator-marketplace/plugins/creator-intake/kb/marketplace-structure.md`
- `marketplaces/agevolt-creator-marketplace/plugins/creator-intake/kb/marketplace-catalog.md`
- `marketplaces/agevolt-creator-marketplace/plugins/creator-intake/kb/git-update-flow.md`

Ak root neexistuje, skus precitat bundlovane KB v tomto plugine:

- `../../kb/marketplace-structure.md`
- `../../kb/marketplace-catalog.md`
- `../../kb/git-update-flow.md`

Ak ani tie nie su dostupne, pouzi pravidla v tomto SKILL.md a povedz, ze interny SharePoint root sa nenasiel.

## Povinne Rozhodnutie

Najprv urci:

1. existujuci marketplace,
2. plugin,
3. typ zmeny: `new-plugin`, `new-skill`, `update-skill`, `new-kb`, `update-kb`, `new-mcp`, `update-mcp`, `manifest-update`,
4. ci je obsah public-safe pre Git.

Ak marketplace nesedi, nepresuvaj obsah nasilu. Pouzi `marketplace-catalog.md` a navrhni spravny marketplace alebo novy marketplace proposal.

## SharePoint Struktura

Marketplace root musi ostat minimalny:

```text
marketplaces/<marketplace-id>/
  README.md
  marketplace.yaml
  plugins/
```

Plugin root:

```text
plugins/<plugin-id>/
  README.md
  plugin.yaml
```

Pridaj iba realne potrebne priecinky:

- skill -> `skills/<skill-id>/SKILL.md`
- skill UI -> `skills/<skill-id>/agents/openai.yaml`
- KB -> `kb/*.md`
- MCP -> `.mcp.json` plus `mcp/README.md`, ak treba vysvetlit server
- Codex plugin -> `.codex-plugin/plugin.json`

Nevytvaraj prazdne `templates/`, `tests/`, `mcp/`, `kb/`, `assets/` ani `scripts/`.

## Git Update

Kazdy public-safe update musi ist aj do Git repozitara marketplace:

```text
C:\AiAgent\repos\<marketplace-id>
```

Ak lokalny checkout neexistuje, naklonuj:

```powershell
gh repo clone AgeVolt/<marketplace-id> C:\AiAgent\repos\<marketplace-id>
```

Pri novom plugine uprav `.agents/plugins/marketplace.json` a pridaj entry s `source.path = "./plugins/<plugin-id>"`.

Pri update existujuceho pluginu bumpni `.codex-plugin/plugin.json` `version`.

Pri MCP pridaj alebo uprav:

```text
plugins/<plugin-id>/.mcp.json
plugins/<plugin-id>/.codex-plugin/plugin.json
```

`plugin.json` musi mat `mcpServers: "./.mcp.json"` iba ked `.mcp.json` realne existuje.

## Validacia

Pred pushom spusti:

```powershell
python "C:\Users\Ján Zuštiak\.codex\skills\.system\skill-creator\scripts\quick_validate.py" "<skill-dir>"
python "C:\Users\Ján Zuštiak\.codex\skills\.system\plugin-creator\scripts\validate_plugin.py" "<plugin-dir>"
```

Skontroluj JSON:

```powershell
Get-Content "<json-path>" -Raw | ConvertFrom-Json
```

Potom:

```powershell
git -C "C:\AiAgent\repos\<marketplace-id>" add .
git -C "C:\AiAgent\repos\<marketplace-id>" commit -m "<kratky popis>"
git -C "C:\AiAgent\repos\<marketplace-id>" push
codex plugin marketplace upgrade <codex-marketplace-id>
```

## Stop Pravidla

- Nemen firemny marketplace bez explicitnej poziadavky.
- Nepridavaj internu KB ani customer data do public Git.
- Nepridavaj MCP bez `.mcp.json` a bez validacie plugin manifestu.
- Nepridavaj plugin do marketplace JSON bez realneho `plugins/<plugin-id>`.
- Nevynechaj Git push, ak ma byt update dostupny ostatnym cez Codex.
