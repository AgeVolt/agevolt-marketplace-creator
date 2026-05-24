---
name: creator-intake
description: Pouzi ked pouzivatel chce vytvorit, zmenit alebo navrhnut AgeVolt marketplace, plugin, skill, knowledge base, MCP, rule, personal rule, script, automatizaciu alebo migraciu zo stareho AI; najma pri poziadavkach ako "sprav skill", "vytvor marketplace", "urob plugin", "potrebujem MCP", "premigruj AI" alebo "daj to pre vsetkych". Najprv rozhodni typ artefaktu a vytvor artifact proposal, nie priamu implementaciu.
---

# Creator Intake

Creator Intake je public-safe bootstrap skill. Jeho uloha je brzdit vznik chaosu: pri poziadavkach na nove AI artefakty najprv rozhodni, co sa ma vobec vytvarat, a az potom navrhni najmensi overitelny krok.

Minimal structure marker: `creator-intake-minimal-structure-001`.

## Najprv Najdi Interny Root

Pred rozhodovanim skus najst lokalny SharePoint root v tomto poradi:

1. Hodnota `AGEVOLT_AI_AGENT_ROOT`, ak existuje.
2. `%USERPROFILE%\OneDrive - AgeVolt Slovakia, s.r.o\Dokumenty - Produkt\AI Agent`.
3. Aktualny workspace alebo jeho rodic, ak sa vola `AI Agent`.

Ak root existuje, citaj tieto interne pravidla:

1. `marketplaces/agevolt-creator-marketplace/README.md`
2. `marketplaces/agevolt-creator-marketplace/marketplace.yaml`
3. `marketplaces/agevolt-creator-marketplace/plugins/creator-intake/README.md`
4. `marketplaces/agevolt-creator-marketplace/plugins/creator-intake/kb/rules.md`
5. `marketplaces/agevolt-creator-marketplace/plugins/creator-intake/kb/marketplace-structure.md`
6. `marketplaces/agevolt-creator-marketplace/plugins/creator-intake/kb/marketplace-catalog.md`
7. `marketplaces/agevolt-creator-marketplace/plugins/creator-intake/kb/git-update-flow.md`
8. `marketplaces/agevolt-creator-marketplace/plugins/creator-intake/kb/artifact-proposal.md`

Ak root neexistuje, skus precitat bundlovane KB v tomto plugine:

- `../../kb/rules.md`
- `../../kb/marketplace-structure.md`
- `../../kb/marketplace-catalog.md`
- `../../kb/git-update-flow.md`
- `../../kb/artifact-proposal.md`

Ak ani tie nie su dostupne, pouzi fallback pravidla nizsie a povedz, ze interny SharePoint root sa nenasiel.

## Fallback Decision Rules

- Marketplace: vytvor iba ked ide o inu cielovu skupinu, viditelnost, citlivost alebo update kanal.
- Plugin: vytvor ked ide o instalovatelny pracovny blok, ktory dava zmysel zapnut samostatne.
- Skill: vytvor ked ide o opakovatelny agenticky postup s jasnym triggerom.
- KB/reference: vytvor ked hlavna hodnota su znalosti, schema, procesy, priklady alebo dokumenty.
- MCP tool: vytvor ked treba zive data, API, vypocet alebo akciu v inom systeme.
- MCP resource: vytvor ked server poskytuje citatelny dynamicky kontext.
- MCP prompt: vytvor ked server poskytuje pouzivatelom vyberatelny prompt template.
- Firemne rule: vytvor ked pravidlo plati pre viac ludi a ma ho zdedit novy clovek v rovnakej roli.
- Personal rule: vytvor ked ide o preferenciu jedneho cloveka.
- Script: vytvor ked treba deterministicku opakovatelnu operaciu.

## Stop Pravidlo

Nevytvaraj ani neupravuj firemny marketplace, plugin, skill, KB, rule alebo MCP bez explicitnej poziadavky. Pri nejasnej poziadavke vrat `artifact proposal`.

## Prve Otazky

Ak chyba kontext, pytaj sa najviac tri otazky naraz:

1. Kto to bude pouzivat a kto to nema vidiet?
2. Ake su 2-3 realne poziadavky, ktore ma agent zvladnut?
3. Ma agent iba navrhovat, alebo aj citat/zapisovat do systemov?

## Artifact Proposal Format

Vystup pri nejasnej alebo novej poziadavke:

```text
Recommended artifact:
Why this:
Why not alternatives:
Audience and access:
Minimal MVP:
Test:
Required approval before writing:
```

## Public Repo Pravidlo

Do public Git marketplace neukladaj internu KB, customer data, credentials, exporty zo SharePointu/ClickUpu/mailov/Teams/SuperFaktury ani nepublikovane firemne rozhodnutia. Public repo je iba bootstrap a update kanal.
