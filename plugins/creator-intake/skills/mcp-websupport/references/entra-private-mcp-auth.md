# Entra ID Auth For Private PHP MCP

Pouzi pri MCP serveroch, ktore pristupuju k private firemnym datam.

## Zakladny Model

Private MCP na WebSupporte ma byt resource server:

```text
Codex/ChatGPT MCP client
  -> ziska Microsoft Entra access token pre AgeVolt MCP API
  -> vola MCP endpoint s Authorization: Bearer <token>
  -> PHP MCP server validuje JWT podpis a claims cez Microsoft JWKS
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

Spolocny scope:

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

## PHP Config

V `server_code/php/config.local.php` zapni auth:

```php
'auth' => [
    'mode' => 'entra',
    'public_health' => true,
    'entra' => [
        'tenant_id' => 'f4ad79c0-38e7-4175-b1c5-1be579eac81b',
        'issuer' => 'https://login.microsoftonline.com/f4ad79c0-38e7-4175-b1c5-1be579eac81b/v2.0',
        'jwks_uri' => 'https://login.microsoftonline.com/f4ad79c0-38e7-4175-b1c5-1be579eac81b/discovery/v2.0/keys',
        'authorization_server' => 'https://login.microsoftonline.com/f4ad79c0-38e7-4175-b1c5-1be579eac81b/v2.0',
        'resource' => 'https://documents.agevolt.com/mcp/<server>/mcp',
        'protected_resource_metadata_url' => 'https://documents.agevolt.com/mcp/<server>/.well-known/oauth-protected-resource',
        'audiences' => [
            'api://772403ea-8d4f-4d26-8908-51e646b089eb',
            '772403ea-8d4f-4d26-8908-51e646b089eb',
        ],
        'allowed_domains' => ['agevolt.com'],
        'allowed_users' => [],
        'allowed_groups' => [],
        'required_scopes' => ['MCP.Access'],
        'required_roles' => [],
    ],
],
```

`audiences` je povinne pre private produkciu. Pre AgeVolt private MCP pouzi shared audience `api://772403ea-8d4f-4d26-8908-51e646b089eb`. Bez audience validacie je token validation prilis siroka.

`resource` a `protected_resource_metadata_url` ostavaju per-MCP, aby klient vedel, ktory konkretny MCP endpoint chrani.

## PHP Server Requirements

PHP MCP server musi:

- pri `auth.mode = entra` vyzadovat `Authorization: Bearer <JWT>` pre MCP a REST tool endpointy,
- validovat `iss`, `tid`, `aud`, `exp`, `nbf`, podpis cez JWKS,
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
- Priamy `codex mcp login agevolt-superfaktura --scopes MCP.Access` zlyhal na `Dynamic client registration not supported`.
- Zaver: priamy Microsoft Entra authorization server nie je dost pre Codex Git marketplace OAuth, lebo Codex skusa DCR a Entra DCR neposkytuje.

Pravidlo: Entra auth pre private PHP MCP najprv skusaj na samostatnom test endpointe, napriklad `/mcp/<server>-auth-test/mcp`, ak endpoint uz pouzivaju bezni pouzivatelia. Na nepouzivanom pilote moze ostat auth zapnuta, kym sa overuje klientsky OAuth flow.

## AgeVolt OAuth Broker Pattern

Ak ma byt login bezudrzbovy pre Codex Git marketplace, vytvor AgeVolt OAuth broker:

```text
Codex MCP OAuth client
  -> AgeVolt OAuth broker s DCR alebo CIMD
  -> Microsoft Entra app AgeVolt MCP
  -> AgeVolt OAuth broker vyda MCP access token
  -> PHP MCP validuje broker token
```

Broker musi:

- vystavit OAuth authorization server metadata,
- podporit `registration_endpoint` pre DCR alebo CIMD client IDs,
- pouzit authorization-code + PKCE,
- presmerovat pouzivatela na Microsoft Entra login,
- po callbacku validovat Entra identity a vydat kratkodoby MCP token s audience konkretneho MCP resource,
- publikovat JWKS, aby PHP MCP servery vedeli podpis tokenu overit,
- nepouzivat ani neukladat hesla pouzivatelov.

Tento broker je preferovany standard pre Codex Git marketplace + Entra, pokial ChatGPT Apps/Connectors sprava neponukne preddefinovany OAuth client pre dany connector.

## Test Plan

Pred nasadenim:

- `GET /health` vracia `auth=entra`, ak je `public_health=true`,
- `POST /mcp` bez tokenu vracia `401`,
- `WWW-Authenticate` obsahuje `resource_metadata`,
- `GET /.well-known/oauth-protected-resource` vracia resource metadata,
- request s validnym tokenom pre shared AgeVolt MCP audience prejde,
- request s tokenom pre Microsoft Graph alebo inu app odmietne `audience is not allowed`,
- request z ineho tenant/domain odmietne.
- novy Codex/ChatGPT chat po instalacii pluginu realne vidi MCP tooly a nepada do fallback HTTP volani.
