# AgeVolt Marketplace Structure

Toto je aktualny standard pre AgeVolt AI Agent marketplaces.

## Co Je Co

| Vrstva | Uloha | Kedy ju vytvorit |
| --- | --- | --- |
| Marketplace | Katalog pluginov pre cielovu skupinu alebo domenu. | Ked sa lisi publikum, citlivost, instalacia alebo update kanal. |
| Plugin | Instalovatelny pracovny blok v marketplace. | Ked dava zmysel zapnut/vypnut celu oblast prace samostatne. |
| Skill | Agenticky postup alebo reference workflow s jasnym triggerom. | Ked sa opakuje postup, checklist, sposob rozhodovania alebo pouzitia nastrojov. |
| KB | Znalosti, pravidla, priklady, schema, procesy. | Ked hlavna hodnota nie je postup, ale obsah, ktory skill cita podla potreby. |
| MCP | Zive nastroje, dynamicke zdroje alebo prompt templates cez server. | Ked agent potrebuje API, databazu, vypocet, zapis alebo aktualne data. |
| Template | Vzor vystupu alebo suboru. | Ked chceme konzistentny format navrhu, dokumentu alebo konfiguracie. |
| Test | Minimalny scenar overenia. | Pri kazdom artefakte, ktory ma byt opakovane pouzivany alebo distribuovany. |

## Standardny Marketplace

```text
<marketplace-id>/
  README.md
  marketplace.yaml
  plugins/
```

Na urovni marketplace nedrz prazdne `kb/`, `mcp/`, `templates/` ani `tests/`. Marketplace iba popisuje cielovu skupinu a zoskupuje instalovatelne pluginy.

## Naming Standard

Marketplace musi mat tri zosuladene nazvy:

| Miesto | Standard | Priklad |
| --- | --- | --- |
| SharePoint priecinok | `agevolt-<oblast>-marketplace` | `agevolt-finance-admin-marketplace` |
| Git repo | `AgeVolt/agevolt-<oblast>-marketplace` | `AgeVolt/agevolt-creator-marketplace` |
| Codex display name | slug bez finalneho `-marketplace`, citatelny Title Case | `AgeVolt Creator` |

Pravidla:

- Slug vzdy konci na `-marketplace`.
- Slug pouziva iba lowercase ASCII, cisla a pomlcky.
- SharePoint priecinok a Git repo musia mat rovnaky slug.
- `marketplace.json` top-level `name` musi byt rovnaky slug.
- `marketplace.yaml` `id` musi byt rovnaky slug.
- Codex display name vznikne zo slugu bez posledneho `-marketplace`: `agevolt-creator-marketplace` -> `AgeVolt Creator`.
- Display name nikdy nekonci slovom `Marketplace`, ak je to iba technicky suffix slugu.
- `interface.displayName` nesmie byt rovnaky pre viac AgeVolt marketplaces, inak je Codex UI neprehladne.
- Nepouzivaj genericke display names ako `Built by AgeVolt`.
- Priklady: `agevolt-finance-admin-marketplace` -> `AgeVolt Finance Admin`; `agevolt-product-myagevolt-marketplace` -> `AgeVolt Product myAgeVolt`.

## Standardny Plugin

```text
plugins/<plugin-id>/
  README.md
  plugin.yaml
  skills/
  kb/
  mcp/
```

Aj v plugine vytvaraj iba priecinky s realnym obsahom. `skills/` vytvor, ked plugin obsahuje skill. `kb/` vytvor, ked skill potrebuje reference alebo pravidla. `mcp/` vytvor az ked existuje realny server alebo konfiguracia. V public Git Codex baliku musi mat plugin navyse `.codex-plugin/plugin.json`. Ak ma plugin MCP konfiguraciu pre Codex, pouzi Codex kompatibilny subor `.mcp.json` v plugin package.

## Pravidlo Urovni

- Nedavaj skill pod MCP server.
- Nedavaj MCP server pod skill.
- V plugine su skill a MCP surodenci.
- Skill vysvetluje workflow a bezpecne pouzitie.
- MCP poskytuje nastroje, resources alebo prompts.
- KB drzi znalosti, ktore by zbytocne zatazovali `SKILL.md`.

## Kedy Vytvorit Novy Marketplace

Novy marketplace vytvor iba ked aspon jedna vec plati:

- ina cielova skupina ho ma instalovat samostatne,
- obsah ma inu citlivost alebo pristup,
- update kanal ma byt oddeleny,
- pluginy by bez rozdelenia zbytocne zahltili nepovolane role,
- public-safe a interne veci by sa miesali.

Ak ide iba o dalsi pracovny blok pre rovnaku skupinu, vytvor plugin v existujucom marketplace.

## Minimalny Postup Pri Novom Marketplace

1. Najprv priprav artifact proposal.
2. Over publikum, citlivost a public/private hranicu.
3. V SharePointe vytvor minimalnu strukturu marketplace.
4. V public Git repozitari vytvor iba Codex marketplace katalog: README a `.agents/plugins/marketplace.json`.
5. Pluginy pridavaj az po schvaleni prveho minimalneho scenara.
6. Pri kazdej firemnej zmene zapis `revision-history.md`, ak marketplace uz ma revision history.
