---
name: create-marketplace
description: Pouzi iba ked sa vytvara novy AgeVolt marketplace, meni sa rozdelenie medzi marketplace-mi, pripravuje sa nove Git marketplace repo, alebo sa riesi marketplace-level katalog/naming. Nepouzivaj na pridanie alebo upravu pluginu, skillu, KB alebo MCP v existujucom marketplace; to patri do update-marketplace.
---

# Create Marketplace

Tento skill vedie tvorbu AgeVolt marketplace. Ciel je minimalny, overitelny a udrzatelny marketplace, nie velky build.

## Hranica Skillu

Tento skill pracuje na urovni marketplace:

- novy marketplace,
- premenovanie alebo rozdelenie marketplace,
- Git marketplace repo a `.agents/plugins/marketplace.json`,
- SharePoint `marketplace.yaml`, README a katalog.

Nepouzivaj ho na tvorbu skillu, KB, MCP alebo pluginu v existujucom marketplace. Ak uz marketplace existuje a menime jeho obsah, pouzi `update-marketplace`.

## Najprv Nacitaj Kontext

Najdi `AI Agent` root v tomto poradi:

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
- `marketplaces/agevolt-creator-marketplace/plugins/creator-intake/kb/artifact-proposal.md`

Ak root neexistuje, skus precitat bundlovane KB v tomto plugine:

- `../../kb/marketplace-structure.md`
- `../../kb/marketplace-catalog.md`
- `../../kb/git-update-flow.md`
- `../../kb/mcp-build-runbook.md`
- `../../kb/artifact-proposal.md`

Ak ani tie nie su dostupne, pouzi pravidla v tomto SKILL.md a povedz, ze interny SharePoint root sa nenasiel.

## Povinne Poradie A Jazyk

Kazdu firemnu marketplace zmenu rob najprv v SharePoint source:

```text
AI Agent/marketplaces/<marketplace-id>/
```

Az potom priprav public-safe Git repo alebo Git mirror. Git nesmie byt jedine miesto, kde vznikne marketplace README, KB, skill alebo MCP dokumentacia.

Vsetky nove alebo upravovane `.md` subory pis po slovensky. Vynimky su technicke identifikatory, nazvy suborov, prikazy, JSON/YAML kluce, frontmatter kluce, API/tool nazvy, presne citacie alebo explicitna poziadavka pouzivatela na iny jazyk.

Do public Gitu nedavaj internu KB, raw exporty, zakaznicke data, produkcne dumpy ani citlive podklady. Public Git obsahuje iba manifesty, public-safe README, public-safe skill workflowy, public-safe KB a instalacne/testovacie pokyny.

`git push` do `main`, `master`, release branchu alebo inej zdielanej vetvy nerob bez explicitneho potvrdenia pouzivatela v aktualnom chate. Ak potvrdenie chyba, priprav iba SharePoint source, Git zmeny, validacie, diff alebo lokalny commit na review a zastav sa pred pushom.

## Rozhodnutie

Novy marketplace vytvor iba ked sa meni cielova skupina, citlivost, public/private hranica, instalacny kanal alebo update kanal.

Ak ide len o dalsiu schopnost pre rovnaku skupinu, navrhni plugin v existujucom marketplace.

Ak ide len o postup, navrhni skill.

Ak ide len o znalosti, navrhni KB.

Ak ide o zive data, API, vypocet alebo zapis, navrhni MCP a skill, ktory vysvetli jeho bezpecne pouzitie.

Pred navrhnutim noveho marketplace over `marketplace-catalog.md`. Pred navrhnutim noveho skillu v buducej implementacii upozorni, ze cielovy plugin musi mat kontrolu existujucich `skills/*/SKILL.md`, aby nevznikol duplicitny skill s rovnakym triggerom.

## MCP Standard Pri Navrhu

Ak marketplace alebo plugin potrebuje MCP, navrhni ho tak, aby Codex vedel volat tooly priamo.

- Tool names v MCP musia pouzivat iba pismena, cisla, `_` alebo `-`, maximalne 64 znakov.
- Nepouzivaj bodkovane nazvy toolov. Napriklad pouzi `sf_documents_list`, nie `sf.documents.list`.
- Ak historicky endpoint pouziva bodky, nech server vystavi Codex aliasy bez bodiek.
- Skill k MCP musi zakazat priame HTTP fallbacky cez shell, citanie `.codex/.credentials.json` a rucne skladanie bearer tokenu.
- Ak MCP tool nie je viditelny, skill ma zastavit user workflow, spravit/odporucit `codex mcp login <mcp-server-id> --scopes MCP.Access`, potom vyziadat novy chat alebo refresh/restart Codexu.
- `agents/openai.yaml` pri kazdom MCP skille ma mat `dependencies.tools` s MCP serverom.
- HTTP/streamable HTTP MCP server musi mat overeny handshake: `initialize`, `notifications/initialized` bez JSON-RPC response, `tools/list` a realny read-only `tools/call`.
- Pri private MCP pouzi `mcp-build-runbook.md` a `mcp-websupport/references/entra-private-mcp-auth.md`.

## Minimalny Postup

1. Vrat artifact proposal, ak nie je explicitne schvalena implementacia.
2. Pouzi ASCII slug: `agevolt-<domain>-marketplace`.
3. V SharePointe vytvor iba:

```text
marketplaces/<marketplace-id>/
  README.md
  marketplace.yaml
  plugins/
```

4. V Git repozitari vytvor iba public-safe Codex marketplace katalog:

```text
README.md
.agents/plugins/marketplace.json
```

`plugins/` v Gite pridaj az s prvym realnym pluginom.

5. Aktualizuj `AI Agent/README.md` a `creator-intake/kb/marketplace-catalog.md`.
6. Nepridavaj realny plugin, skill, KB ani MCP do prazdneho marketplace bez samostatneho schvalenia.
7. Pri firemnej zmene zapis `revision-history.md`, ak existuje.
8. Po explicitne schvalenom Git pushi over `codex plugin marketplace upgrade <codex-marketplace-id>`.

## Test

Minimalny test marketplace je:

- SharePoint folder existuje a ma standardne priecinky.
- `marketplace.yaml` ma `id`, `display_name`, `status`, `git_repo`, `audience`, `purpose`, `plugins`.
- Git repo obsahuje `.agents/plugins/marketplace.json`.
- Codex vie Git marketplace pridat alebo upgradovat.
- Pluginy sa instaluju az vtedy, ked marketplace obsahuje aspon jeden schvaleny plugin.
