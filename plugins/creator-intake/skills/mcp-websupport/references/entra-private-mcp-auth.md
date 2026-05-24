# Entra ID Auth For Private PHP MCP

Pouzi pri MCP serveroch, ktore pristupuju k private firemnym datam.

## Zakladny Model

Private MCP na WebSupporte ma byt resource server za AgeVolt OAuth Brokerom:

```text
Codex/ChatGPT MCP client
  -> AgeVolt OAuth Broker https://documents.agevolt.com/mcp/auth
  -> Microsoft Entra app AgeVolt MCP
  -> broker vyda MCP access token + refresh token
  -> MCP endpoint dostane Authorization: Bearer <access_token>
  -> PHP MCP server validuje broker JWT podpis cez broker JWKS
```

PHP server nepozna heslo pouzivatela a nikdy ho nema poznat.

## Shared AgeVolt MCP App Registration

AgeVolt pouziva jednu spolocnu single-tenant app registration pre firemne private MCP servery. Nevytvaraj novu Entra app registration pre kazdy MCP, pokial pouzivatel explicitne neschvali iny security model.

```text
Display name: AgeVolt MCP
Application (client) ID: 772403ea-8d4f-4d26-8908-51e646b089eb
Object ID: 30151999-1e72-4023-b24f-c185f5febbd6
Directory (tenant) ID: f4ad79c0-38e7-4175-b1c5-1be579eac81b
Supported account types: Accounts in this organizational directory only
Tenant: AgeVolt
```

Spolocna Application ID URI:

```text
api://772403ea-8d4f-4d26-8908-51e646b089eb
```

Spolocny scope pouzivany brokerom a MCP:

```text
MCP.Access
```

Ak `Expose an API` este nie je hotove, v Entra admin center nastav:

1. `Expose an API` -> `Set` Application ID URI na `api://772403ea-8d4f-4d26-8908-51e646b089eb`.
2. `Add a scope`:
   - Scope name: `MCP.Access`
   - Who can consent: `Admins only`
   - Admin consent display name: `Access AgeVolt MCP`
   - Admin consent description: `Allows approved clients to access AgeVolt private MCP servers.`
   - State: `Enabled`

Volitelne pre role/oddelenia pridaj app roles alebo pouzi group assignment v Enterprise application `AgeVolt MCP`.

Pre AgeVolt tenant bol discovery overeny cez:

```text
https://login.microsoftonline.com/agevolt.com/v2.0/.well-known/openid-configuration
```

Tenant ID:

```text
f4ad79c0-38e7-4175-b1c5-1be579eac81b
```

## AgeVolt OAuth Broker

Finálny shared broker:

```text
Issuer: https://documents.agevolt.com/mcp/auth
Authorization endpoint: https://documents.agevolt.com/mcp/auth/authorize
Token endpoint: https://documents.agevolt.com/mcp/auth/token
Dynamic registration endpoint: https://documents.agevolt.com/mcp/auth/register
JWKS: https://documents.agevolt.com/mcp/auth/.well-known/jwks.json
Entra callback: https://documents.agevolt.com/mcp/auth/callback/entra
```

Pouzivatel sa prihlasi raz cez Microsoft. Broker vydava kratkodoby access token a dlhodoby refresh token, aby Codex vedel obnovovat pristup bez casteho prihlasovania. Standardny refresh token TTL je 365 dni, pokial firemna security politika neurci kratsi interval.

## PHP MCP Config

V `server_code/php/config.local.php` zapni auth:

```php
'auth' => [
    'mode' => 'entra',
    'public_health' => true,
    'entra' => [
        'tenant_id' => 'f4ad79c0-38e7-4175-b1c5-1be579eac81b',
        'issuer' => 'https://documents.agevolt.com/mcp/auth',
        'jwks_uri' => 'https://documents.agevolt.com/mcp/auth/.well-known/jwks.json',
        'authorization_server' => 'https://documents.agevolt.com/mcp/auth',
        'resource' => 'https://documents.agevolt.com/mcp/<server>/mcp',
        'protected_resource_metadata_url' => 'https://documents.agevolt.com/mcp/<server>/.well-known/oauth-protected-resource',
        'audiences' => [
            'https://documents.agevolt.com/mcp/<server>/mcp',
        ],
        'allowed_domains' => ['agevolt.com'],
        'allowed_users' => [],
        'allowed_groups' => [],
        'required_scopes' => ['MCP.Access'],
        'required_roles' => [],
    ],
],
```

`audiences` je povinne pre private produkciu. Pre AgeVolt private MCP pouzi per-MCP resource URL ako audience, nie Microsoft Graph ani Entra client ID. Bez audience validacie je token validation prilis siroka.

`resource` a `protected_resource_metadata_url` ostavaju per-MCP, aby klient vedel, ktory konkretny MCP endpoint chrani. `authorization_server` je shared broker pre vsetky MCP.

## PHP Server Requirements

PHP MCP server musi:

- pri `auth.mode = entra` vyzadovat `Authorization: Bearer <JWT>` pre MCP a REST tool endpointy,
- validovat `iss`, `aud`, `exp`, `nbf`, podpis cez broker JWKS,
- obmedzit pouzivatelov cez `allowed_domains`, `allowed_users`, `allowed_groups`, `required_scopes` alebo `required_roles`,
- vratit `401` s `WWW-Authenticate`,
- verejne vystavit protected resource metadata endpoint:

```text
/.well-known/oauth-protected-resource
```

## Client Reality Check

Nezamienaj tieto dve veci:

1. PHP resource server validation: server vie odmietnut request bez validneho Entra tokenu.
2. MCP client OAuth flow: Codex/ChatGPT klient musi vediet token ziskat a poslat.

Ak aktualny Codex Git plugin nevie pre dany custom marketplace spustit OAuth flow, Entra auth bude serverovo spravne, ale plugin v chate neuvidi tools. Vtedy treba:

- pouzit ChatGPT Apps/Connectors OAuth konfiguraciu, ak cielovy surface podporuje custom remote MCP OAuth,
- alebo postavit OAuth gateway/proxy,
- alebo ponechat MCP interne iba v sieti/VPN, kym Codex custom plugin OAuth flow nebude overeny.

Neobchadzaj to ulozenym admin heslom, refresh tokenom alebo dlhodobym user tokenom v PHP.

AgeVolt overenie 2026-05-24:

- SuperFaktura MCP na WebSupporte bol zapnuty do `auth.mode = entra`.
- `GET /health` fungoval ako public health a ukazal `auth=entra`.
- `POST /mcp` bez tokenu korektne vratil `401` a `WWW-Authenticate` s `resource_metadata`.
- Codex Git marketplace po reinstall/restart nevypytal Microsoft login.
- V novom chate sa MCP tooly `sf_documents_*` nevystavili.
- Kedze pilot MCP v case testu nepouzivali ostatni pouzivatelia, endpoint ostal v `auth.mode = entra` a dalsi krok je rozbehat klientsky OAuth login cez Codex/ChatGPT podporovany flow.
- Priamy `codex mcp login agevolt-superfaktura --scopes MCP.Access` proti Entra zlyhal na `Dynamic client registration not supported`.
- Po nasadeni AgeVolt OAuth Broker na `https://documents.agevolt.com/mcp/auth`, prepojeni SuperFaktura protected resource metadata na broker a nastaveni Entra callbacku `https://documents.agevolt.com/mcp/auth/callback/entra` login presiel.
- Codex ulozil OAuth credentials a novy `codex exec` proces uspesne zavolal MCP tool `sf_documents_list`.

Pravidlo: Entra auth pre private PHP MCP najprv skusaj na samostatnom test endpointe, napriklad `/mcp/<server>-auth-test/mcp`, ak endpoint uz pouzivaju bezni pouzivatelia. Na nepouzivanom pilote moze ostat auth zapnuta, kym sa overuje klientsky OAuth flow.

## AgeVolt OAuth Broker Pattern

Ak ma byt login bezudrzbovy pre Codex Git marketplace, pouzi AgeVolt OAuth Broker:

```text
Codex MCP OAuth client
  -> AgeVolt OAuth broker s DCR
  -> Microsoft Entra app AgeVolt MCP
  -> AgeVolt OAuth broker vyda MCP access token + refresh token
  -> PHP MCP validuje broker token
```

Broker musi:

- vystavit OAuth authorization server metadata,
- podporit `registration_endpoint` pre DCR client IDs,
- pouzit authorization-code + PKCE,
- presmerovat pouzivatela na Microsoft Entra login,
- po callbacku validovat Entra identity a vydat kratkodoby MCP token s audience konkretneho MCP resource,
- vydat refresh token s firemnym TTL, aktualne 365 dni, aby sa pouzivatel neprihlasoval stale znova,
- publikovat JWKS, aby PHP MCP servery vedeli podpis tokenu overit,
- nepouzivat ani neukladat hesla pouzivatelov.

Tento broker je preferovany standard pre Codex Git marketplace + Entra, pokial ChatGPT Apps/Connectors sprava neponukne preddefinovany OAuth client pre dany connector.

## Test Plan

Pred nasadenim:

- `GET https://documents.agevolt.com/mcp/auth/health` vracia broker health,
- `GET /health` na MCP vracia `auth=entra`, ak je `public_health=true`,
- `POST /mcp` bez tokenu vracia `401`,
- `WWW-Authenticate` obsahuje `resource_metadata`,
- `GET /.well-known/oauth-protected-resource` vracia resource metadata,
- resource metadata ukazuje `authorization_servers` na `https://documents.agevolt.com/mcp/auth`,
- `codex mcp login <server>` prejde a `codex mcp list` ukazuje `Auth OAuth`,
- request s validnym broker tokenom pre per-MCP audience prejde,
- request s tokenom pre Microsoft Graph alebo inu app odmietne `audience is not allowed`,
- request z ineho tenant/domain odmietne.
- novy Codex/ChatGPT chat po instalacii pluginu realne vidi MCP tooly a nepada do fallback HTTP volani.
