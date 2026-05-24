# Creator Intake Rules

## Decision Matrix

Vyber najmensi artefakt, ktory realne riesi poziadavku.

| Artefakt | Kedy ho vytvorit |
| --- | --- |
| Marketplace | Ina cielova skupina, viditelnost, citlivost alebo update kanal. |
| Plugin | Instalovatelny pracovny blok, ktory dava zmysel zapnut samostatne. |
| Skill | Opakovatelny agenticky postup s jasnym triggerom. |
| KB/reference | Hlavna hodnota su znalosti, schema, procesy, priklady alebo dokumenty. |
| MCP tool | Treba zive data, API, vypocet alebo akciu v inom systeme. |
| MCP resource | Server poskytuje citatelny dynamicky kontext. |
| MCP prompt | Server poskytuje pouzivatelom vyberatelny prompt template. |
| Firemne rule | Pravidlo plati pre viac ludi a ma ho zdedit novy clovek v rovnakej roli. |
| Personal rule | Preferencia jedneho cloveka. |
| Script | Deterministicka opakovatelna operacia. |

## Stop Rules

- Nevytvaraj firemny skill len preto, ze pouzivatel povedal "sprav skill".
- Nevytvaraj novy skill, ak existujuci skill v rovnakom plugine ma podobny trigger, scope a pravidla; najprv navrhni update alebo zlucenie.
- Neupravuj firemne marketplace/plugin/skill/KB/MCP/rule bez explicitnej poziadavky.
- Nekopiruj stare `AI/` priecinky cele.
- Nedavaj internu KB, customer data, secrets ani systemove exporty do public Git.
- Pri MCP write akcii vyzaduj read/preview a explicitne potvrdenie.
- Pri MCP nikdy nenavrhuj bezny user workflow cez priamy HTTP fallback namiesto MCP tool callu.

## Skill Uniqueness Rules

Kazdy skill v plugine musi mat jasny vlastny dovod existencie.

Pred vytvorenim alebo upravou `SKILL.md`:

1. Precitaj vsetky ostatne `skills/*/SKILL.md` v tom istom plugine.
2. Porovnaj `name`, frontmatter `description`, trigger slova, scope, non-goals, povinne KB, pouzivane nastroje a bezpecnostne pravidla.
3. Ak je novy skill podobny existujucemu skillu, nevytvaraj ho; navrhni update existujuceho skillu alebo presun spolocnych pravidiel do `kb/`.
4. Ak dva existujuce skilly robia to iste, upozorni na duplicitu a navrhni zlucenie.
5. Novy skill je povoleny iba ked ma iny opakovatelny workflow, ine spustacie situacie, iny scope a ine pravidla.
6. `description` musi jasne povedat, kedy skill pouzit a idealne aj co patri do ineho skillu.
7. Pri kazdom update `SKILL.md` znova over, ci sa hranica s ostatnymi skillmi nerozmazala.

Kontrolna veta pred vytvorenim skillu:

```text
Precital som ostatne skilly v plugine a tento skill ma iny trigger/scope/rules ako: ...
```

## MCP Tool Rules

MCP pre Codex musi byt navrhnuty ako priamo volatelny tool surface.

- Tool name musi obsahovat iba pismena, cisla, `_` alebo `-` a mat najviac 64 znakov.
- Nepouzivaj bodky, medzery ani lomitka v MCP tool names.
- Pre namespacing pouzivaj underscore, napriklad `sf_documents_list`, `sf_expenses_create_preview`.
- Ak stary API wrapper pouziva `sf.documents.list`, server musi pridat alias `sf_documents_list` a skill ma pouzivat alias.
- Skill nesmie odporucat `Invoke-RestMethod`, `curl`, browser URL, priamy `/index.php/sf.*` fallback, citanie `.codex/.credentials.json` ani rucny bearer token pre bezne pouzitie.
- Pri neviditelnom MCP v chate zastav a nepouzivaj HTTP/token fallback. Najprv over `codex mcp list`; pri private AgeVolt MCP spusti alebo odporuc `codex mcp login <mcp-server-id> --scopes MCP.Access`, potom novy chat/refresh/restart. Marketplace upgrade alebo reinstall ries az po overeni loginu.
- Skill s MCP ma mat v `agents/openai.yaml` `dependencies.tools` s `type: "mcp"`, server `value`, `transport` a `url`.
- Write tools musia mat preview/execute model a execute musi vyzadovat `confirmation_id`.
- HTTP/streamable HTTP MCP server musi spravne obsluzit JSON-RPC notifications: request bez `id`, napriklad `notifications/initialized`, nesmie vratit JSON-RPC response s `id: null`.
- Pred odovzdanim MCP over minimalne `initialize`, `notifications/initialized`, `tools/list`, jeden realny read-only `tools/call`, `codex mcp login`, `Auth OAuth` a novy chat alebo `codex exec` bez shell fallbacku.

## Required Output Pri Nejasnej Poziadavke

Vrat artifact proposal alebo sa opytaj tri intake otazky.

Nepokracuj implementaciou, ak nie je jasne:

- kto to pouziva,
- ci ide o osobnu alebo firemnu vec,
- ci je potrebny zapis do systemu,
- ako to otestujeme.
