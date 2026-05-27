# AgeVolt Creator

`Public-safe` Git marketplace pre riadenie tvorby AgeVolt AI artefaktov.

Tento marketplace je zamerne maly a public-safe. Obsahuje plugin `creator-intake`, jeho public-safe skilly, KB a MCP build pravidla potrebne pre Codex marketplace update. Secrets, private server configy, priklady so zakaznickymi datami, fixtures a migracna praca ostavaju v SharePoint zdroji:

```text
AI Agent/marketplaces/agevolt-creator-marketplace/
```

## Na Co Sluzi

Pouzivaj ho pre AI adminov, module ownerov a power userov, ktori vytvaraju alebo udrziavaju AgeVolt AI marketplaces, pluginy, skilly, KB, MCP servery, pravidla a scripts.

Bezni zamestnanci tento marketplace nepotrebuju iba na pouzivanie hotovych biznis pluginov.

## Plugin V Tomto MVP

```text
creator-intake
```

`creator-intake` rozhoduje, akym typom artefaktu ma poziadavka byt, este pred implementaciou. Pri nejasnych poziadavkach ma vratit artifact proposal, napriklad:

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

Kedze ide o Git marketplace, Codex ho vie aktualizovat standardnym marketplace upgrade flowom.

```powershell
codex plugin marketplace upgrade creator
```

## Jazyk A Pravidla Public-Safe Obsahu

Vsetky `.md` subory v tomto marketplace a jeho public Git kopii pis po slovensky. Vynimky su technicke identifikatory, prikazy, JSON/YAML kluce, frontmatter kluce, nazvy API/toolov, presne citacie alebo explicitna poziadavka na iny jazyk.

Tento SharePoint priecinok je interny zdroj pravdy. Public Git repo je iba distribucny/update kanal pre Codex a nesmie obsahovat internu KB.

Repo moze obsahovat:

- plugin manifesty,
- public-safe Creator skilly,
- public-safe Creator KB a MCP build pravidla,
- verejne instalacne a testovacie pokyny.

Repo nesmie obsahovat:

- internu KB,
- zakaznicke data,
- secrets alebo credentials,
- interne exporty,
- skopirovany legacy `AI/` obsah,
- nepublikovane produktove, financne, support alebo production detaily.
