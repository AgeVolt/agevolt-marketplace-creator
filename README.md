# AgeVolt Creator

Verejne bezpecny Git marketplace pre riadenie tvorby AgeVolt AI artefaktov.

Tento marketplace je zamerne maly a verejne bezpecny. Obsahuje plugin `creator-intake`, jeho verejne bezpecne skilly, KB a MCP pravidla pre tvorbu potrebne pre Codex marketplace aktualizaciu. Tajne hodnoty, sukromne serverove konfiguracie, priklady so zakaznickymi datami, testovacie fixture data a migracna praca ostavaju v SharePoint zdroji:

```text
AI Agent/marketplaces/agevolt-creator-marketplace/
```

## Na Co Sluzi

Pouzivaj ho pre AI adminov, vlastnikov modulov a power userov, ktori vytvaraju alebo udrziavaju AgeVolt AI marketplaces, pluginy, skilly, KB, MCP servery, pravidla a skripty.

Bezni zamestnanci tento marketplace nepotrebuju iba na pouzivanie hotovych biznis pluginov.

## Plugin V Tomto MVP

```text
creator-intake
```

`creator-intake` rozhoduje, akym typom artefaktu ma poziadavka byt, este pred implementaciou. Pri nejasnych poziadavkach ma vratit navrh artefaktu, napriklad:

- "sprav skill"
- "vytvor marketplace"
- "urob plugin"
- "potrebujem MCP"
- "premigruj AI"

## Instalacia

V Codexe pridaj tento Git marketplace:

```text
AgeVolt/agevolt-creator-marketplace
```

Potom nainstaluj plugin `Creator Intake`.

CLI ekvivalent:

```powershell
codex plugin marketplace add AgeVolt/agevolt-creator-marketplace
codex plugin add creator-intake@creator
```

## Aktualizacia

Kedze ide o Git marketplace, Codex ho vie aktualizovat standardnym marketplace upgrade postupom.

```powershell
codex plugin marketplace upgrade creator
```

## Jazyk A Pravidla Verejne Bezpecneho Obsahu

Vsetky `.md` subory v tomto marketplace a jeho verejnej Git kopii pis po slovensky. Vynimky su technicke identifikatory, prikazy, JSON/YAML kluce, frontmatter kluce, nazvy API/toolov, presne citacie alebo explicitna poziadavka na iny jazyk.

SharePoint priecinok je interny zdroj pravdy. Verejny Git repo je iba distribucny/aktualizacny kanal pre Codex a nesmie obsahovat internu KB.

`git push` do `main`, `master`, release branchu alebo inej zdielanej vetvy je povoleny iba po explicitnom potvrdeni pouzivatela v aktualnom chate. Bez potvrdenia priprav iba SharePoint zdroj, verejne bezpecne Git zmeny, validacie, diff alebo lokalny commit na kontrolu.

Repo moze obsahovat:

- plugin manifesty,
- verejne bezpecne Creator skilly,
- verejne bezpecne Creator KB a MCP build pravidla,
- verejne instalacne a testovacie pokyny.

Repo nesmie obsahovat:

- internu KB,
- zakaznicke data,
- secrets alebo credentials,
- interne exporty,
- skopirovany legacy `AI/` obsah,
- nepublikovane produktove, financne, support alebo production detaily.
