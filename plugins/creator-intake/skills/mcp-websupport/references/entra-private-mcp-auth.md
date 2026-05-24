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

## Entra App Registration

V Microsoft Entra admin center vytvor single-tenant app registration pre konkretne MCP API:

```text
Name: AgeVolt MCP - <plugin/server name>
Supported account types: Accounts in this organizational directory only
Tenant: AgeVolt
```

Zapis:

- Directory tenant ID,
- Application client ID,
- Application ID URI alebo audience,
- exposed scope, napriklad `MCP.Access`,
- volitelne app roles alebo group assignment pre obmedzenie pristupu.

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
        'audiences' => ['<Application client ID alebo Application ID URI>'],
        'allowed_domains' => ['agevolt.com'],
        'allowed_users' => [],
        'allowed_groups' => [],
        'required_scopes' => ['MCP.Access'],
        'required_roles' => [],
    ],
],
```

`audiences` je povinne pre private produkciu. Bez audience validacie je token validation prilis siroka.

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

## Test Plan

Pred nasadenim:

- `GET /health` vracia `auth=entra`, ak je `public_health=true`,
- `POST /mcp` bez tokenu vracia `401`,
- `WWW-Authenticate` obsahuje `resource_metadata`,
- `GET /.well-known/oauth-protected-resource` vracia resource metadata,
- request s validnym tokenom pre spravne audience prejde,
- request s tokenom pre Microsoft Graph alebo inu app odmietne `audience is not allowed`,
- request z ineho tenant/domain odmietne.
