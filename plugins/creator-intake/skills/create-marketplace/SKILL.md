---
name: create-marketplace
description: Pouzi ked pouzivatel chce vytvorit, navrhnut alebo upravit AgeVolt AI marketplace, rozdelit pluginy medzi marketplaces, pripravit Git marketplace repo, alebo rozhodnut ci poziadavka patri do noveho marketplace alebo existujuceho pluginu. Najprv vytvor artifact proposal a az potom minimalny overitelny krok.
---

# Create Marketplace

Tento skill vedie tvorbu AgeVolt marketplace. Ciel je minimalny, overitelny a udrzatelny marketplace, nie velky build.

## Najprv Nacitaj Kontext

Najdi `AI Agent` root v tomto poradi:

1. `AGEVOLT_AI_AGENT_ROOT`, ak existuje.
2. `%USERPROFILE%\OneDrive - AgeVolt Slovakia, s.r.o\Dokumenty - Produkt\AI Agent`.
3. Aktualny workspace alebo jeho rodic, ak sa vola `AI Agent`.

Ak root existuje, precitaj:

- `README.md`
- `marketplaces/agevolt-marketplace-creator/plugins/creator-intake/kb/rules.md`
- `marketplaces/agevolt-marketplace-creator/plugins/creator-intake/kb/marketplace-structure.md`
- `marketplaces/agevolt-marketplace-creator/plugins/creator-intake/kb/marketplace-catalog.md`
- `marketplaces/agevolt-marketplace-creator/plugins/creator-intake/kb/git-update-flow.md`
- `marketplaces/agevolt-marketplace-creator/plugins/creator-intake/kb/artifact-proposal.md`

Ak root neexistuje, skus precitat bundlovane KB v tomto plugine:

- `../../kb/marketplace-structure.md`
- `../../kb/marketplace-catalog.md`
- `../../kb/git-update-flow.md`
- `../../kb/artifact-proposal.md`

Ak ani tie nie su dostupne, pouzi pravidla v tomto SKILL.md a povedz, ze interny SharePoint root sa nenasiel.

## Rozhodnutie

Novy marketplace vytvor iba ked sa meni cielova skupina, citlivost, public/private hranica, instalacny kanal alebo update kanal.

Ak ide len o dalsiu schopnost pre rovnaku skupinu, navrhni plugin v existujucom marketplace.

Ak ide len o postup, navrhni skill.

Ak ide len o znalosti, navrhni KB.

Ak ide o zive data, API, vypocet alebo zapis, navrhni MCP a skill, ktory vysvetli jeho bezpecne pouzitie.

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
8. Po Git pushi over `codex plugin marketplace upgrade <marketplace-id>`.

## Test

Minimalny test marketplace je:

- SharePoint folder existuje a ma standardne priecinky.
- `marketplace.yaml` ma `id`, `display_name`, `status`, `git_repo`, `audience`, `purpose`, `plugins`.
- Git repo obsahuje `.agents/plugins/marketplace.json`.
- Codex vie Git marketplace pridat alebo upgradovat.
- Pluginy sa instaluju az vtedy, ked marketplace obsahuje aspon jeden schvaleny plugin.
