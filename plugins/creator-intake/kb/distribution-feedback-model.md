# Distribucia, Update A Feedback Model

Tento subor definuje zakladny model, ako AgeVolt AI marketplaces dostanu bezni
pouzivatelia, co patri do Gitu, co patri iba na SharePoint a ako sa zbiera
spatna vazba bez toho, aby bezny pouzivatel menil ostre zdroje skillov alebo KB.

## Zakladne Pravidlo

```text
SharePoint = interny source of truth a admin pracovny priestor.
Git = public-safe distribucny a update kanal pre Codex.
Codex cache = lokalne nainstalovana kopia pre konkretneho pouzivatela.
Feedback inbox = bezpecne miesto, kam moze pouzivatel alebo skill zapisat problem.
```

## Co Patri Do SharePointu

SharePoint `AI Agent/marketplaces/**` je miesto, kde admini udrziavaju ostre
zdroje:

- marketplace README a `marketplace.yaml`,
- plugin README a `plugin.yaml`,
- public-safe aj private KB,
- skilly a ich `agents/openai.yaml`,
- MCP dokumentacia a private `server_code`,
- private reference, pristupy, konfiguracie a interny kontext,
- `revision-history.md`.

Bezny pouzivatel nema byt vedeny k tomu, aby upravoval `marketplaces/**`.
Ak chce zmenu, zapisuje feedback alebo navrh do `AI Agent/feedback/inbox/`.

## Co Patri Do Gitu

Public Git marketplace obsahuje iba to, co musi Codex vediet stiahnut,
nainstalovat a upgradnut:

- `.agents/plugins/marketplace.json`,
- public-safe `README.md`,
- `.codex-plugin/plugin.json`,
- public-safe `SKILL.md`,
- public-safe `agents/openai.yaml`,
- public-safe KB a reference potrebne na beh skillu,
- `.mcp.json` bez secretov,
- assets a instalacne/testovacie pokyny.

Git nesmie obsahovat:

- hesla, tokeny, API kluce, FTP pristupy,
- `config.local.php`,
- zakaznicke data,
- produkcne exporty, dumpy alebo cele maily,
- internu KB, ktora nie je schvalena ako public-safe,
- feedback zaznamy od realnych pouzivatelov.

## Zamestnanecky Install Flow

Novy pouzivatel ma dostat co najmenej technicky postup:

1. Otvori Codex.
2. Da Codexu Git link na AgeVolt marketplace.
3. Codex prida marketplace.
4. Codex spusti Creator alebo onboarding skill na instalaciu default pluginov.
5. Pouzivatel otvori novy chat alebo refreshne Codex, aby sa nacitali skilly.

Kazdy realne pouzivany AgeVolt plugin ma byt v Git `.agents/plugins/marketplace.json`
nastaveny ako:

```json
"policy": {
  "installation": "INSTALLED_BY_DEFAULT",
  "authentication": "ON_INSTALL"
}
```

`authentication: "ON_USE"` pouzi iba pri user-data alebo MCP OAuth pluginoch, kde
je lepsie login vyziadat az pri prvom realnom pouziti.

Pozor: `INSTALLED_BY_DEFAULT` je AgeVolt zamer, nie zaruka, ze Codex CLI plugin
automaticky nainstaluje. Po `marketplace add` alebo `marketplace upgrade` treba
spustit `install-marketplace-plugins` alebo ekvivalentny `codex plugin add`
pre default pluginy.

## Update Flow Pre Pouzivatela

Minimalny update flow:

1. `codex plugin marketplace upgrade <codex-marketplace-id>`
2. `install-marketplace-plugins`
3. novy chat alebo refresh Codexu

Personal productivity marketplace ma pouzivatelovi ponuknut automatizaciu typu
`AgeVolt Marketplace Update Check`, ktora v bezpecnom intervale skontroluje a
spusti update dostupnych AgeVolt marketplaces a default pluginov.

Takato automatizacia nesmie sama menit SharePoint source ani Git. Iba aktualizuje
lokalnu instalaciu pouzivatela.

## Feedback Flow

Feedback sa zapisuje do:

```text
AI Agent/feedback/inbox/
```

Feedback moze vzniknut dvoma sposobmi:

1. Pouzivatel explicitne povie, ze chce zapisat problem, navrh alebo poziadavku.
2. Skill sa spusti implicitne, lebo pouzivatel pise frustrovane alebo hovori, ze
   AgeVolt AI, Codex, plugin, skill, KB, MCP alebo automatizacia zlyhala.

Skill ma zapisat kratky zaznam, nie cele konverzacie.

## Feedback Zaznam

Minimalny tvar:

```text
Date/time:
User:
Marketplace/plugin/skill:
Original request:
What went wrong:
Expected behavior:
Likely missing KB/rule/tool:
Suggested owner:
Source links:
Severity:
Privacy notes:
```

Ak chybaju informacie, skill doplni neistoty a nepytaj sa pouzivatela na dlhy
formular, pokial to nie je nutne.

## Hranica Bezpecnosti

Bezny feedback skill:

- smie zapisovat do `feedback/inbox/`,
- nesmie upravovat `marketplaces/**`,
- nesmie robit Git commit alebo push,
- nesmie menit ostre skilly, KB alebo MCP server code,
- nesmie kopirovat citlive data do feedbacku.

Creator/admin workflow:

- cita `feedback/inbox/`,
- rozhodne, ci ide o KB update, skill update, MCP update, novu automatizaciu
  alebo ziadnu zmenu,
- presunie zaznam do `triage/` alebo `processed/`,
- zapracuje zmenu cez SharePoint source-first postup a public-safe Git mirror.

## Pasivne Zachytenie Limit

Codex skill sa vie spustit na zaklade svojej `description`, ale neslubuj globalne
pasivne sledovanie vsetkych chatov, ak aktualny Codex/ChatGPT runtime neposkytuje
globalny hook. Preto musi mat feedback skill velmi silnu `description` a musi byt
instalovany defaultne cez personal productivity marketplace.
