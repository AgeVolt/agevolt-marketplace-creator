---
name: mcp-websupport
description: Pouzi ked treba vytvorit, upravit, opravit, presunut alebo deploynut AgeVolt MCP server na WebSupporte cez PHP/FTP, najma ked poziadavka spomina MCP server, WebSupport, FTP, PHP endpoint, server_code, tools/list, tools/call alebo priame MCP tool volania v Codexe.
---

# MCP WebSupport

Tento skill je pre AgeVolt MCP servery hostovane ako PHP endpointy na WebSupporte. Ciel je, aby MCP fungoval priamo v Codexe ako tool surface, nie cez shell, `curl`, `Invoke-RestMethod` alebo fallback HTTP endpointy.

## Bezpecnost

- Nikdy nevkladaj FTP hesla, API tokeny ani ine secrets do public Git repozitara.
- Nikdy nevypisuj heslo do terminalu, chatu, commit message ani logu.
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
7. Skill, ktory MCP pouziva, musi hovorit o priamych MCP tooloch a nesmie odporucat HTTP fallbacky.
8. Pri zmene public pluginu bumpni verziu a pushni Git marketplace.
9. Pri zmene private server_code zapis zmenu do SharePoint revision history.

## Deploy Workflow

1. Precitaj WebSupport private reference zo SharePoint-only Creator skillu.
2. Heslo drz iba v premennej v pamati, nikdy ho neechoj.
3. Deployuj obsah `server_code/php/` na WebSupport target.
4. Po deployi otestuj `/health`.
5. Otestuj MCP handshake a minimalny read-only `tools/call`.

Minimalna validacia:

```text
initialize -> 200 JSON-RPC response
notifications/initialized -> 202/204 empty body
tools/list -> tool names bez bodiek
tools/call -> jeden read-only tool vrati realne data alebo korektnu domenu chybu
```

Ak MCP nie je v novom chate vystaveny, najprv over server handshake. Az potom ries reinstall, upgrade alebo restart Codexu.
