# Creator Intake Pravidla

## Rozhodovacia Matica

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

## Stop Pravidla

- Nevytvaraj firemny skill len preto, ze pouzivatel povedal "sprav skill".
- Nevytvaraj novy skill, ak existujuci skill v rovnakom plugine ma podobny trigger, scope a pravidla; najprv navrhni update alebo zlucenie.
- Neupravuj firemne marketplace/plugin/skill/KB/MCP/rule bez explicitnej poziadavky.
- Nekopiruj stare `AI/` priecinky cele.
- Nedavaj internu KB, customer data, secrets ani systemove exporty do public Git.
- Pri MCP write akcii vyzaduj read/preview a explicitne potvrdenie.
- Pri MCP nikdy nenavrhuj bezny user workflow cez priamy HTTP fallback namiesto MCP tool callu.
- Nikdy nerob `git push` do `main`, `master`, release branchu ani inej zdielanej vetvy bez explicitneho potvrdenia pouzivatela v aktualnom chate.

## Jazyk A Source Of Truth

- Vsetky nove alebo upravovane `.md` subory v AgeVolt AI Agent, marketplaces, pluginoch, skilloch, KB, MCP dokumentacii a public Git mirroroch pis po slovensky.
- Vynimky su iba technicke identifikatory, nazvy suborov, prikazy, JSON/YAML kluce, frontmatter kluce, API/tool nazvy, presne citacie alebo explicitna poziadavka pouzivatela na iny jazyk.
- UI nazvy a kratke UI texty pre marketplaces, pluginy a skilly pis vzdy po
  anglicky: `.agents/plugins/marketplace.json` `interface.displayName`,
  `.codex-plugin/plugin.json` `interface.displayName`, `shortDescription` a
  `defaultPrompt`, `plugin.yaml` `display_name` a `skills/*/agents/openai.yaml`
  `display_name`, `short_description` a `default_prompt`.
- Pri firemnej implementacnej zmene najprv uprav SharePoint source v `AI Agent/marketplaces/<marketplace-id>/...`.
- Public Git repo je public-safe distribucny/update kanal pre Codex, nie interny zdroj pravdy.
- Pri kazdej implementacnej zmene naraz kontroluj SharePoint source aj Git checkout. Pred editom si pozri cielovy SharePoint priecinok, zodpovedajuci Git repozitar a rozdiel medzi nimi; po edite over, ze vsetky public-safe subory zmenene v SharePointe su premietnute do Gitu.
- Ak Git obsahuje iba public-safe projekciu SharePoint zdroja, rozdiel musi byt zamerne pomenovany: ktore subory su Git public-safe, ktore su SharePoint-only private a preco.
- Pri kazdom update skontroluj aj jazyk: nove alebo upravene `.md` subory v SharePointe aj Gite musia byt po slovensky, okrem technickych identifikatorov a povolenych vynimiek.
- Pri kazdom update skontroluj referencie zo skillov na KB/subory. Ak public Git skill odkazuje na KB, dana KB musi byt v Gite, alebo skill musi jasne povedat, ze ide o SharePoint-only sukromny zdroj a pri absencii nahlasit chybajuci pristup/zdroj.
- Internu KB, raw exporty, zakaznicke data, produkcne dumpy a citlive podklady nechaj v SharePointe alebo inom schvalenom private ulozisku.
- Ak public Git skill potrebuje internu KB, musi vediet najst lokalny `AI Agent` root a pri jeho absencii jasne povedat, ze interna KB nie je dostupna. Nesmie si ju vymysliet ani ju duplikovat do Gitu.
- Bez potvrdenia pushu moze agent pripravit SharePoint source, Git working tree, validacie, diff alebo lokalny commit na review, ale musi zastavit pred `git push`.
- Vseobecne zadanie typu "oprav to", "implementuj plan" alebo "sprav update" nie je suhlas s pushom do `main`.

## Marketplace Instalacia Pluginov

- Kazdy realne instalovatelny AgeVolt plugin v `.agents/plugins/marketplace.json` ma mat defaultne `policy.installation: "INSTALLED_BY_DEFAULT"`.
- Default `policy.authentication` je `ON_INSTALL`. Vynimka je povolena pri user-data alebo MCP OAuth pluginoch, kde je zamerne lepsie `ON_USE`.
- Ak plugin nema byt instalovany automaticky po pridani marketplace, musi mat zdokumentovany dovod v Creator navrhu alebo pri marketplace update.
- Samotne `codex plugin marketplace add` alebo `codex plugin marketplace upgrade` pluginy cez CLI realne nenainstaluje. Po pridani alebo upgrade marketplace treba spustit `install-marketplace-plugins` alebo rovnocenny krok `codex plugin add <plugin>@<marketplace>` pre kazdy default plugin.
- Skilly sa neinstaluju samostatne. Dostupne su az po instalacii pluginu, ktory ich obsahuje.
- Pri novom plugine alebo zmene marketplace manifestu vzdy over `codex plugin list`, aby kazdy default plugin skoncil ako `installed, enabled`.

## Pravidla Jedinecnosti Skillov

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

## MCP Tool Pravidla

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

## Povinne Vystupy Pri Nejasnej Poziadavke

Vrat artifact proposal alebo sa opytaj tri intake otazky.

Nepokracuj implementaciou, ak nie je jasne:

- kto to pouziva,
- ci ide o osobnu alebo firemnu vec,
- ci je potrebny zapis do systemu,
- ako to otestujeme.
