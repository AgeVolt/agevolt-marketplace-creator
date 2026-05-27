---
name: mcp-websupport
description: Pouzi ked treba vytvorit, upravit, opravit, presunut alebo deploynut AgeVolt MCP server na WebSupporte cez PHP/FTP, najma ked poziadavka spomina MCP server, WebSupport, FTP, PHP endpoint, server_code, tools/list, tools/call alebo priame MCP tool volania v Codexe.
---

# MCP WebSupport

Tento skill je pre AgeVolt MCP servery hostovane ako PHP endpointy na WebSupporte. Ciel je, aby MCP fungoval priamo v Codexe ako tool surface, nie cez shell, `curl`, `Invoke-RestMethod` alebo fallback HTTP endpointy.

Pred vytvorenim alebo vacsou upravou MCP precitaj aj `../../kb/mcp-build-runbook.md`. Pri private data MCP precitaj aj `references/entra-private-mcp-auth.md`.

## Jazyk A Source Of Truth

Vsetky MCP `.md` subory, ktore vytvoris alebo upravis, pis po slovensky. Vynimky su technicke identifikatory, nazvy suborov, prikazy, JSON/YAML kluce, API/tool nazvy, presne citacie alebo explicitna poziadavka na iny jazyk.

MCP dokumentaciu, server_code README a private reference najprv uprav v SharePoint source pod konkretnym marketplace/pluginom. Do public Gitu synchronizuj iba public-safe cast bez secrets, accessov, raw exportov a internych konfiguracii.

`git push` do `main`, `master`, release branchu alebo inej zdielanej vetvy rob iba po explicitnom potvrdeni pouzivatela v aktualnom chate. Bez potvrdenia priprav len SharePoint source, Git zmeny, validacie, diff alebo lokalny commit na review.

## Bezpecnost

- Nikdy nevkladaj FTP hesla, API tokeny ani ine secrets do public Git repozitara.
- Nikdy nepouzivaj heslo poslane v chate. Ak ho pouzivatel posle, upozorni ho, ze ho ma zmenit, a pokracuj iba bezpecnym admin flowom.
- Nikdy nevypisuj heslo do terminalu, chatu, commit message ani logu.
- Nikdy necitaj `%USERPROFILE%\.codex\.credentials.json` a nikdy nepouzivaj rucne vybraty access token na bezny user workflow.
- Public Git moze obsahovat iba bez-secrets skill, KB, `.mcp.json`, README, deployment postup a kod, ktory neobsahuje tajomstva.
- Spolocne WebSupport pristupy citaj iba zo SharePoint-only private reference suboru.

Private reference pre spolocne WebSupport pristupy:

```text
<AI Agent root>/marketplaces/agevolt-creator-marketplace/plugins/creator-intake/skills/mcp-websupport/references/websupport-private-access.md
```

AI Agent root typicky najdes tu:

```text
%UserProfile%/OneDrive - AgeVolt Slovakia, s.r.o/Dokumenty - Produkt/AI Agent
```

Ak private reference chyba, zastav a vypytaj si doplnenie pristupov alebo potvrdenie bezpecneho zdroja. Nevytvaraj nahradny public subor so secrets.

Pri MS365/Entra administracii nikdy neziadaj ani nepouzivaj admin heslo v chate. Admin sa ma prihlasit interaktivne v Microsoft Entra admin center, `az login` alebo inom schvalenom bezpečnom flowe s MFA.

WebSupport FTP root pre AgeVolt MCP je verejny root `https://documents.agevolt.com/mcp/`. Konkretny MCP server deployuj do vlastneho podadresara, napriklad `/superfaktura`. Shared OAuth broker deployuj do `/auth`.

## Kam Patri MCP

Kazdy MCP patri pod konkretny plugin, nie na root marketplace:

```text
marketplaces/<marketplace-id>/plugins/<plugin-id>/mcp/
marketplaces/<marketplace-id>/plugins/<plugin-id>/mcp/server_code/
```

Plugin root pre Codex obsahuje MCP konfiguraciu:

```text
plugins/<plugin-id>/.mcp.json
plugins/<plugin-id>/.codex-plugin/plugin.json
```

`plugin.json` ma obsahovat `mcpServers: "./.mcp.json"` iba ked `.mcp.json` realne existuje.

## Server Code Standard

`mcp/server_code/` obsahuje iba konkretne veci pre dany MCP:

- deployovatelny PHP kod,
- `.htaccess`, ak WebSupport routing potrebuje `/mcp`, `/health` alebo `/openapi`,
- deploy script bez hardcodovaneho hesla,
- MCP-specific README,
- MCP-specific config alebo tokeny, iba ak maju ostat interne na SharePointe a nejdu do public Gitu.

Spolocne FTP/WebSupport pristupy ostavaju v Creator private reference. V `server_code` nechaj iba pointer na Creator skill/reference.

## PHP MCP Kontrakt

HTTP/streamable HTTP MCP endpoint musi podporovat:

- `initialize` s `id` -> validna JSON-RPC response,
- `notifications/initialized` bez `id` -> ziadna JSON-RPC response, prazdne telo s HTTP `202` alebo `204`,
- `tools/list` -> zoznam toolov,
- `tools/call` -> priame volanie toolu.

Tool names pre Codex:

- iba `A-Z`, `a-z`, `0-9`, `_`, `-`,
- maximalne 64 znakov,
- bez bodiek, medzier a lomitok,
- pouzivaj namespace cez underscore, napriklad `sf_documents_list`, nie `sf.documents.list`.

Ak historicky REST endpoint pouziva bodkovane nazvy, server moze stare endpointy dalej podporovat interne, ale MCP `tools/list` musi vracat Codex-safe aliasy.

## Private Data Auth

Ak MCP pristupuje k private firemnym datam, musi mat auth pred tools/list/tools/call. Preferovany AgeVolt model je spolocny AgeVolt OAuth Broker napojeny na jednu Microsoft Entra ID app registration `AgeVolt MCP`:

- jedna shared single-tenant Entra app `AgeVolt MCP` pre prihlasenie pouzivatela,
- jeden shared broker `https://documents.agevolt.com/mcp/auth`,
- kazdy MCP ma vlastny resource/audience, napriklad `https://documents.agevolt.com/mcp/superfaktura/mcp`,
- PHP MCP server validuje brokerom vydany JWT cez broker JWKS,
- access token je kratkodoby, refresh token je dlhodoby pre bezudrzbove pouzivanie v Codexe,
- `Authorization: Bearer <access_token>`,
- povinna kontrola `issuer`, `audience`, expiracie, podpisu a scope,
- volitelne `required_scopes`, `required_roles`, `allowed_groups` alebo `allowed_users`.

Precitaj detailny postup:

```text
references/entra-private-mcp-auth.md
```

Nepouzivaj priamy Microsoft Entra authorization server v protected resource metadata. Codex CLI OAuth login skusa Dynamic Client Registration a Entra ID DCR nepodporuje. Protected resource metadata MCP servera ma ukazovat na AgeVolt OAuth Broker:

```text
authorization_servers: ["https://documents.agevolt.com/mcp/auth"]
```

## Write Flow

Kazda write/delete/send/payment akcia musi mat preview/execute model:

1. `*_preview` vrati prehlad zmeny a `confirmation_id`.
2. Execute tool prijme iba `confirmation_id`.
3. Execute volaj az po explicitnom potvrdeni pouzivatela.

Read-only tool moze bezat priamo.

## Create/Edit Workflow

1. Identifikuj marketplace, plugin a cielovy MCP server.
2. Najdi alebo vytvor `mcp/server_code/` v SharePointe pri konkretnom plugine.
3. Pri existujucom MCP najprv precitaj `server_code/README.md`, `.mcp.json`, skill, KB a PHP entrypoint.
4. Pri novom MCP navrhni minimalny tool surface a prvy read-only smoke test.
5. V PHP implementuj najprv `initialize`, `notifications/initialized`, `tools/list`, `tools/call`, `/health`.
6. Pridaj `.mcp.json` do plugin rootu a `mcpServers: "./.mcp.json"` do `.codex-plugin/plugin.json`.
7. Skill, ktory MCP pouziva, musi hovorit o priamych MCP tooloch a nesmie odporucat HTTP fallbacky, citanie `.codex/.credentials.json` ani rucne bearer tokeny.
8. Ak MCP pristupuje k private datam, pouzi shared AgeVolt OAuth Broker `https://documents.agevolt.com/mcp/auth` a nastav MCP audience na jeho public MCP URL.
9. Pri zmene public pluginu bumpni verziu a priprav Git marketplace; pushni az po explicitnom potvrdeni pouzivatela.
10. Pri zmene private server_code zapis zmenu do SharePoint revision history.

## Deploy Workflow

1. Precitaj WebSupport private reference zo SharePoint-only Creator skillu.
2. Heslo drz iba v premennej v pamati, nikdy ho neechoj.
3. Deployuj obsah `server_code/php/` na WebSupport target.
4. Po deployi otestuj `/health`.
5. Otestuj MCP handshake a minimalny read-only `tools/call`.
6. Pri private MCP otestuj Codex E2E cez `codex mcp login`, `codex mcp list` a novy chat alebo `codex exec`; nepouzivaj rucne citanie credentials.

Minimalna validacia:

```text
initialize -> 200 JSON-RPC response
notifications/initialized -> 202/204 empty body
tools/list -> tool names bez bodiek
tools/call -> jeden read-only tool vrati realne data alebo korektnu domenu chybu
```

## Codex OAuth Onboarding

Pri private MCP po deployi otestuj aj Codex OAuth login:

```text
codex mcp login <mcp-server-id> --scopes MCP.Access
codex mcp list
```

`codex mcp list` ma ukazat MCP server ako `Auth OAuth`. Browser nemusi pytat heslo, ak je pouzivatel uz prihlaseny do MS365; stranka `Authentication complete` znamena uspesny callback a ulozenie prihlasenia v Codexe.

Nikdy po login teste necitaj `.codex/.credentials.json` a nepokracuj rucnym `Invoke-WebRequest` s bearer tokenom. Ak tool surface v aktualnom chate stale nevidi MCP tooly, otvor novy chat alebo spusti izolovany `codex exec` test. User workflow musi bezat cez MCP tool surface.

Ak MCP nie je v novom chate vystaveny, postupuj v tomto poradi:

1. Over server handshake a `.mcp.json`.
2. Over, ze MCP server je v `codex mcp list`.
3. Pri private MCP spusti `codex mcp login <mcp-server-id> --scopes MCP.Access`.
4. Po `Auth OAuth` otvor novy chat alebo restartuj/refreshni Codex.
5. Az potom ries reinstall alebo marketplace upgrade.
