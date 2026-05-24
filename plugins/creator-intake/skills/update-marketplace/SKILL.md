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
- `marketplaces/agevolt-creator-marketplace/plugins/creator-intake/kb/mcp-build-runbook.md`

Ak root neexistuje, skus precitat bundlovane KB v tomto plugine:

- `../../kb/marketplace-structure.md`
- `../../kb/marketplace-catalog.md`
- `../../kb/git-update-flow.md`
- `../../kb/mcp-build-runbook.md`

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

## MCP Update Standard

Pri kazdom `new-mcp`, `update-mcp`, `new-skill` alebo `update-skill`, ktory pouziva MCP:

- MCP tool names musia byt Codex/OpenAI kompatibilne: iba pismena, cisla, `_` alebo `-`, maximalne 64 znakov.
- Nepouzivaj bodky v MCP tool names. Z `sf.documents.list` urob `sf_documents_list`.
- Ak backend potrebuje stare bodkovane cesty, nech ostanu ako interne HTTP endpointy; MCP `tools/list` ma vracat Codex kompatibilne aliasy.
- Skill ma instruovat agenta, aby volal priamo MCP tooly a neobchadzal ich cez `curl`, `Invoke-RestMethod`, priame HTTP endpointy, `.codex/.credentials.json` alebo rucne bearer tokeny.
- Ak MCP tooly nie su v chate viditelne, skill ma zakazat HTTP/token fallback a najprv overit registraciu + OAuth login MCP servera. Pri private AgeVolt MCP pouzi `codex mcp login <mcp-server-id> --scopes MCP.Access`; po uspesnom login ma `codex mcp list` ukazat `Auth OAuth`. Potom otvor novy chat alebo sprav refresh/restart Codexu, aby sa tool surface nacital. Nepokracuj rucnym volanim MCP cez shell.
- Ak browser ukaze `Authentication complete` bez zadania hesla, ber to ako uspesny SSO login cez uz prihlaseny MS365 browser session.
- HTTP/streamable HTTP MCP server musi spravne ignorovat JSON-RPC notifications: `notifications/initialized` bez `id` nesmie vratit JSON-RPC response s `id: null`.
- `skills/<skill-id>/agents/openai.yaml` ma deklarovat MCP dependency:

```yaml
dependencies:
  tools:
    - type: "mcp"
      value: "<mcp-server-id>"
      description: "<human-readable server>"
      transport: "streamable_http"
      url: "https://..."
```

## Validacia

Pred pushom spusti:

```powershell
python "C:\Users\Ján Zuštiak\.codex\skills\.system\skill-creator\scripts\quick_validate.py" "<skill-dir>"
python "C:\Users\Ján Zuštiak\.codex\skills\.system\plugin-creator\scripts\validate_plugin.py" "<plugin-dir>"
```

Pri MCP navyse over:

- `initialize` vrati validnu JSON-RPC response,
- `notifications/initialized` bez `id` vrati prazdne telo s HTTP `202` alebo `204`,
- `tools/list` vrati Codex kompatibilne nazvy bez bodiek,
- jeden read-only `tools/call` funguje priamo cez MCP,
- private MCP ma `codex mcp login <mcp-server-id> --scopes MCP.Access`, `codex mcp list` = `Auth OAuth` a novy chat alebo `codex exec` vidi tool bez shell fallbacku.

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
