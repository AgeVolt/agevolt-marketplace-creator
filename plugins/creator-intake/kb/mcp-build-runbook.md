# MCP Build Postup

Tento runbook pouzi pri kazdom novom alebo upravovanom AgeVolt MCP. Ciel je, aby dalsi MCP fungoval v Codexe priamo ako tool surface, bez shell fallbackov a bez rucneho citania tokenov.

Vsetky nove alebo upravovane MCP `.md` subory pis po slovensky. Vynimky su technicke identifikatory, nazvy suborov, prikazy, JSON/YAML kluce, API/tool nazvy, presne citacie alebo explicitna poziadavka na iny jazyk. Najprv uprav SharePoint source pri konkretnom marketplace/pluginu a do public Gitu posielaj iba public-safe cast.

`git push` do `main`, `master`, release branchu alebo inej zdielanej vetvy nikdy nerob bez explicitneho potvrdenia pouzivatela v aktualnom chate. Bez potvrdenia priprav iba diff alebo lokalny commit na review.

## Referencny Vzor

Najprv si pozri existujuci SuperFaktura MCP, lebo je aktualny overeny vzor:

```text
SharePoint:
AI Agent/marketplaces/agevolt-finance-admin-marketplace/plugins/superfaktura/
AI Agent/marketplaces/agevolt-finance-admin-marketplace/plugins/superfaktura/mcp/server_code/

Git:
C:\AiAgent\repos\agevolt-finance-admin-marketplace\plugins\superfaktura/

Public endpoint:
https://documents.agevolt.com/mcp/superfaktura/mcp

Shared OAuth broker:
https://documents.agevolt.com/mcp/auth
```

Nevytvaraj novy MCP naslepo. Pri novom MCP si porovnaj `.mcp.json`, `.codex-plugin/plugin.json`, skill, KB, `mcp/README.md`, `server_code/README.md`, PHP entrypoint a deploy script so SuperFaktura vzorom.

## Kedy Vytvorit MCP

MCP vytvor iba ked agent potrebuje zive data, aktualny stav, API, vypocet alebo zapis do externeho systemu. Ak ide iba o postup, vytvor skill. Ak ide iba o staticke znalosti, vytvor KB.

Typicke MCP priklady:

- citanie faktur, objednavok, klientov, taskov, skladovych stavov alebo vyroby,
- vytvorenie alebo uprava zaznamu v inom systeme,
- bezpecne spustenie firemneho workflowu,
- dynamicky kontext, ktory nemoze byt v statickej KB.

## Povinna Hierarchia

MCP patri pod konkretny plugin, nie na root marketplace a nie pod skill.

```text
marketplaces/<marketplace-id>/plugins/<plugin-id>/
  README.md
  plugin.yaml
  .mcp.json
  .codex-plugin/plugin.json
  skills/<skill-id>/SKILL.md
  skills/<skill-id>/agents/openai.yaml
  kb/<domain-reference>.md
  mcp/README.md
  mcp/server_code/
```

V public Git repozitari mozu byt iba public-safe subory. Secret konfiguracia, realne tokeny, FTP pristupy, API kluce a `config.local.php` ostavaju iba v SharePointe alebo na serveri.

## Minimalny Public Git Obsah Pre MCP Plugin

Plugin v Git marketplace musi obsahovat:

```text
plugins/<plugin-id>/
  .codex-plugin/plugin.json
  .mcp.json
  assets/icon.png
  assets/logo.png
  skills/<skill-id>/SKILL.md
  skills/<skill-id>/agents/openai.yaml
  skills/<skill-id>/assets/icon.png
  skills/<skill-id>/assets/logo.png
  kb/<public-safe-reference>.md
```

`.agents/plugins/marketplace.json` musi obsahovat plugin entry:

```json
{
  "name": "<plugin-id>",
  "source": {
    "source": "local",
    "path": "./plugins/<plugin-id>"
  },
  "policy": {
    "installation": "INSTALLED_BY_DEFAULT",
    "authentication": "ON_INSTALL"
  },
  "category": "Productivity"
}
```

`.codex-plugin/plugin.json` musi mat:

```json
{
  "name": "<plugin-id>",
  "version": "0.1.0",
  "skills": "./skills/",
  "mcpServers": "./.mcp.json",
  "interface": {
    "displayName": "<Short Plugin Name>",
    "shortDescription": "<co robi>",
    "developerName": "AgeVolt",
    "brandColor": "#280046",
    "composerIcon": "./assets/icon.png",
    "logo": "./assets/logo.png"
  }
}
```

`.mcp.json` musi pouzivat Codex kompatibilny HTTP MCP server id:

```json
{
  "mcpServers": {
    "<mcp-server-id>": {
      "type": "http",
      "url": "https://documents.agevolt.com/mcp/<server>/mcp"
    }
  }
}
```

`<mcp-server-id>` pouzivaj v tvare `agevolt-<domain>`, napriklad `agevolt-superfaktura`.

## Skill Pokyny Pre MCP

Kazdy skill, ktory pouziva MCP, musi jasne povedat:

```text
Pouzivaj priamo MCP tooly zo servera `<mcp-server-id>`.
Nevolaj HTTP endpointy cez shell, curl, Invoke-RestMethod, browser URL ani iny fallback.
Necitaj `.codex/.credentials.json` a nikdy si neskladaj Authorization Bearer token rucne.
```

Ak MCP tooly nie su v aktualnom chate viditelne, skill ma zastavit normalny user workflow a pouzit tento postup:

1. Over `codex mcp list`.
2. Ak server nie je `Auth OAuth`, spusti alebo odporuc:

```text
codex mcp login <mcp-server-id> --scopes MCP.Access
```

3. Ak browser ukaze `Authentication complete`, povazuj to za uspesny MS365 SSO callback.
4. Po `Auth OAuth` otvor novy chat alebo restartuj/refreshni Codex.
5. Nepokracuj rucnym volanim MCP cez `Invoke-WebRequest`, `curl`, `.credentials.json` alebo priamy bearer token.

Toto pravidlo je dolezite: ked MCP nie je vystaveny do tool surface, agent nema "zachranovat" user request cez shell. Ma opravit onboarding stav a nechat novy chat nacitat tooly cisto.

## agents/openai.yaml Dependency

Kazdy skill, ktory potrebuje MCP, ma v `agents/openai.yaml` deklarovat zavislost:

```yaml
dependencies:
  tools:
    - type: "mcp"
      value: "<mcp-server-id>"
      description: "<Human MCP name>"
      transport: "streamable_http"
      url: "https://documents.agevolt.com/mcp/<server>/mcp"
```

Ikony pouzivaj lokalne pri skille:

```yaml
interface:
  icon_small: "./assets/icon.png"
  icon_large: "./assets/logo.png"
  brand_color: "#280046"
```

Nepouzivaj `../` v ikonach; Codex cache tieto cesty ignoruje.

## Tool Naming

Tool names vystavene v `tools/list` musia byt priamo volatelne modelom:

- iba `A-Z`, `a-z`, `0-9`, `_`, `-`,
- maximalne 64 znakov,
- bez bodiek, medzier a lomitok,
- namespace cez underscore.

Priklady:

```text
sf_documents_list
sf_documents_get
sf_documents_create_preview
sf_documents_create_execute
sf_expenses_list
```

Ak historicky backend alebo REST endpoint pouziva bodkovane nazvy ako `sf.documents.list`, nech ostanu iba ako kompatibilne HTTP aliasy. MCP `tools/list` musi vracat underscore nazvy.

## Read/Write Bezpecnost

Read-only tool moze bezat priamo.

Write, delete, send, payment, import alebo akakolvek nezvratna akcia musi mat preview/execute model:

1. `*_preview` vrati sumar zmeny, rizika a `confirmation_id`.
2. Agent ukaze preview pouzivatelovi.
3. Execute tool sa vola az po explicitnom potvrdeni v aktualnom chate.
4. Execute tool prijme iba `confirmation_id`, nie cely payload znova.

Skill musi obsahovat zakaz priameho execute bez preview.

## WebSupport Server Code

SharePoint `mcp/server_code/` ma obsahovat konkretne veci pre dany MCP:

```text
mcp/server_code/
  README.md
  .gitignore
  deploy/deploy-websupport.ps1
  deploy/test-results-YYYY-MM-DD.md
  php/.htaccess
  php/index.php
  php/config.php
  php/config.local.example.php
  php/lib/
  php/storage/.htaccess
```

`config.local.php` s realnymi secretmi nesmie ist do public Gitu. Ak existuje v SharePointe, musi byt v `.gitignore` a nikdy sa nesmie vypisat do chatu ani commitu.

WebSupport deploy root:

```text
FTP root: /agevolt.com/sub/documents/mcp
Public root: https://documents.agevolt.com/mcp/
MCP server path: /<server>
MCP endpoint: https://documents.agevolt.com/mcp/<server>/mcp
Health: https://documents.agevolt.com/mcp/<server>/health
Protected resource metadata: https://documents.agevolt.com/mcp/<server>/.well-known/oauth-protected-resource
```

Spolocne FTP pristupy citaj iba zo SharePoint-only Creator private reference. Nikdy ich nedavaj do public Git.

## Auth Standard Pre Private MCP

Ak MCP cita alebo zapisuje private firemne data, musi pouzit shared AgeVolt OAuth Broker:

```text
Authorization server: https://documents.agevolt.com/mcp/auth
Scope: MCP.Access
Issuer/JWKS: broker metadata
Audience: https://documents.agevolt.com/mcp/<server>/mcp
```

Nepouzivaj priamy Microsoft Entra authorization server v protected resource metadata. Entra nepodporuje DCR sposobom, ktory Codex MCP login potrebuje. Protected resource metadata MCP servera ma ukazovat na broker:

```json
{
  "authorization_servers": [
    "https://documents.agevolt.com/mcp/auth"
  ]
}
```

Detailny auth postup je v:

```text
skills/mcp-websupport/references/entra-private-mcp-auth.md
```

Pri novom private MCP si najprv precitaj tento auth reference a porovnaj ho so SuperFaktura `server_code`.

## Server Testy

Pred instalacnym testom over server:

1. `GET https://documents.agevolt.com/mcp/<server>/health`
   - ma vratit health,
   - pri private MCP ma ukazat auth rezim, ak je `public_health=true`.
2. `GET https://documents.agevolt.com/mcp/<server>/.well-known/oauth-protected-resource`
   - ma vratit resource metadata,
   - `authorization_servers` ma ukazovat na `https://documents.agevolt.com/mcp/auth`.
3. `POST https://documents.agevolt.com/mcp/<server>/mcp` bez tokenu
   - pri private MCP ma vratit `401`,
   - `WWW-Authenticate` ma obsahovat `resource_metadata`.
4. MCP handshake na test/dev endpointe alebo v autorizovanom prostredi:
   - `initialize` s `id` vrati JSON-RPC response,
   - `notifications/initialized` bez `id` vrati prazdne telo s HTTP `202` alebo `204`,
   - `tools/list` vrati tool names bez bodiek,
   - jeden read-only `tools/call` vrati realne data alebo korektnu domenu chybu.

Pri private produkcnom MCP nepouzivaj manualne citanie Codex credential tokenov na test. E2E test rob cez `codex mcp login` a novy Codex proces/chat.

## Codex E2E Testy

Po explicitne schvalenom Git pushi a instalacii pluginu otestuj:

1. Marketplace sa da pridat z Git repo.
2. Plugin sa da nainstalovat.
3. Cache obsahuje novu verziu pluginu.
4. `codex mcp list` obsahuje `<mcp-server-id>`.
5. Pri private MCP prebehne:

```text
codex mcp login <mcp-server-id> --scopes MCP.Access
codex mcp list
```

6. `codex mcp list` ukazuje `Auth OAuth`.
7. Otvor novy chat alebo spusti izolovany test:

```text
codex exec --skip-git-repo-check --dangerously-bypass-approvals-and-sandbox -C C:\AiAgent "Pouzi iba MCP tool <tool-name> ..."
```

8. Vystup musi ukazat, ze sa spustil MCP tool, nie shell fallback.

Zakazane E2E skratky:

- necitaj `%USERPROFILE%\.codex\.credentials.json`,
- nepouzivaj access token z credentials v `Invoke-WebRequest`,
- nevolaj MCP cez raw HTTP v beznom user workflowe,
- nevracaj userovi "nasiel som data", ak si ich ziskal mimo MCP tool surface.

## User Onboarding Test

Simulacia noveho zamestnanca:

1. Odstran AgeVolt marketplace, plugin, MCP login a cache.
2. V Codexe pridaj Git marketplace `AgeVolt/<marketplace-id>`.
3. `Referencia Git`: `main`.
4. `Selektivne cesty`: prazdne.
5. Nainstaluj plugin.
6. Ak sa login nespusti automaticky, spusti alebo odporuc `codex mcp login <mcp-server-id> --scopes MCP.Access`.
7. Ak browser iba ukaze `Authentication complete`, je to uspesne SSO.
8. Otvor novy chat.
9. Daj jednoduchy read-only prompt, ktory musi pouzit MCP tool.
10. Agent nesmie citat `.credentials.json` ani pouzit raw HTTP fallback.

## Done Definition

MCP je hotovy az ked plati:

- struktura je pri konkretnom plugine,
- public Git obsah je bez secrets,
- `.mcp.json` je v plugin root a `plugin.json` ma `mcpServers`,
- skill ma MCP dependency a zakazuje HTTP/token fallbacky,
- KB obsahuje tool mapping a domenove pravidla,
- server ma health, protected resource metadata a MCP handshake,
- private MCP pouziva shared OAuth Broker,
- `codex mcp login` funguje,
- novy chat alebo `codex exec` vidi MCP tooly,
- read-only smoke test prejde priamo cez MCP tool,
- write tooly maju preview/execute model,
- SharePoint `revision-history.md` ma zapis zmien,
- Git repo je bumpnute a commitnute; push prebehol iba po explicitnom potvrdeni pouzivatela.
