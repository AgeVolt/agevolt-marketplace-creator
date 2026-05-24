# Git Update Flow

Tento subor definuje, co sa musi zmenit v SharePointe a v Git marketplace, aby si zmenu vedeli ostatni pouzivatelia aktualizovat v Codexe.

## Zaklad

Codex vie automaticky upgradovat iba Git marketplace. SharePoint lokalny priecinok je interny source of truth, ale samotny Codex upgrade pre beznych pouzivatelov ide cez public Git repo marketplace.

Preto kazdy public-safe artefakt musi mat:

1. SharePoint source v `AI Agent/marketplaces/<marketplace-id>/`.
2. Git publikovatelny obsah v `https://github.com/AgeVolt/<marketplace-id>`.
3. Commit a push do Git repozitara.
4. Overenie cez `codex plugin marketplace upgrade <codex-marketplace-id>`.

## Pri Novom Marketplace

V SharePointe vytvor iba:

```text
marketplaces/<marketplace-id>/
  README.md
  marketplace.yaml
  plugins/
```

V Git repozitari vytvor iba:

```text
README.md
.agents/plugins/marketplace.json
```

`marketplace.json` musi mat:

```json
{
  "name": "<codex-marketplace-id>",
  "interface": {
    "displayName": "<Human Name>"
  },
  "plugins": []
}
```

`<marketplace-id>` je dlhy SharePoint/Git slug, napriklad `agevolt-finance-admin-marketplace`.
`<codex-marketplace-id>` je kratky UI slug, napriklad `finance-admin`. Tento kratky nazov Codex zobrazuje vedla nazvu pluginu v zozname doplnkov, preto nepouzivaj `agevolt` ani `marketplace`, ak to nie je nutne.

Potom aktualizuj:

- root `AI Agent/README.md`,
- `creator-intake/kb/marketplace-catalog.md`,
- GitHub public repo URL v `marketplace.yaml`.

## Pri Novom Plugine V Existujucom Marketplace

V SharePointe vytvor:

```text
marketplaces/<marketplace-id>/plugins/<plugin-id>/
  README.md
  plugin.yaml
```

Pridaj iba realne potrebne casti:

```text
skills/<skill-id>/SKILL.md
skills/<skill-id>/agents/openai.yaml
  kb/*.md
  .mcp.json
  mcp/README.md
  .codex-plugin/plugin.json
```

Pravidla:

- `skills/` vytvor iba ked plugin obsahuje skill.
- `kb/` vytvor iba ked plugin ma realnu knowledge base alebo reference.
- `.mcp.json` vytvor iba ked plugin ma MCP server.
- `mcp/` vytvor iba ked treba dokumentovat alebo drzat zdroj MCP casti.
- Pri MCP si precitaj `creator-intake/kb/mcp-build-runbook.md` a pouzi ho ako checklist.
- Nevytvaraj prazdne `templates/`, `tests/`, `assets/`, `scripts/` ani `mcp/`.

V Git repozitari `<marketplace-id>` pridaj public-safe plugin:

```text
plugins/<plugin-id>/
  .codex-plugin/plugin.json
  skills/
  kb/
  .mcp.json
```

Do `.agents/plugins/marketplace.json` pridaj entry:

```json
{
  "name": "<plugin-id>",
  "source": {
    "source": "local",
    "path": "./plugins/<plugin-id>"
  },
  "policy": {
    "installation": "AVAILABLE",
    "authentication": "ON_INSTALL"
  },
  "category": "Productivity"
}
```

Plugin manifest musi mat standardne AgeVolt UI assety:

```json
{
  "interface": {
    "brandColor": "#280046",
    "composerIcon": "./assets/icon.png",
    "logo": "./assets/logo.png"
  }
}
```

Do pluginu pridaj realne subory `assets/icon.png` a `assets/logo.png`. Kazdy skill s `agents/openai.yaml` ma mat vlastne `assets/icon.png`, `assets/logo.png`, `icon_small`, `icon_large` a `brand_color`.

V SharePoint `marketplace.yaml` pridaj plugin do `plugins`.

## Pri Novom Alebo Upravovanom Skille

Pred vytvorenim alebo upravou `SKILL.md` v existujucom plugine:

1. Precitaj vsetky `skills/*/SKILL.md` v cielovom plugine.
2. Porovnaj `name`, `description`, trigger slova, scope, non-goals, KB, nastroje a pravidla.
3. Novy skill vytvor iba ked ma iny trigger, scope, description a pravidla.
4. Ak sa obsah prekryva, uprav existujuci skill alebo vytiahni spolocne pravidla do `kb/`.
5. Ak su dva skilly podobne, navrhni merge namiesto dalsieho delenia.
6. Po update skontroluj, ze `agents/openai.yaml` kratky popis stale zodpoveda novemu `SKILL.md`.

## Pri Update Existujuceho Pluginu

Pri zmene skillu, KB, MCP alebo plugin manifestu:

1. Uprav SharePoint source.
2. Uprav Git repo public-safe obsah.
3. Bumpni `plugins/<plugin-id>/.codex-plugin/plugin.json` `version` semverom.
4. Ak pribudol MCP server, pridaj `mcpServers` do `plugin.json` a `.mcp.json` do plugin rootu.
5. Ak MCP server zmizol, odstran `mcpServers` aj `.mcp.json`.
6. Commitni a pushni Git repo.
7. Spusti `codex plugin marketplace upgrade <codex-marketplace-id>`.
8. Over, ze cache obsahuje novu verziu pluginu a novy skill/KB/MCP.
9. Pri private MCP over `codex mcp login <mcp-server-id> --scopes MCP.Access`, potom `codex mcp list` = `Auth OAuth` a novy chat/refresh Codexu.

Bez version bumpu moze byt tazsie overit, ci sa pouzivatelovi naozaj refreshol plugin cache.

## Public Safe Hranica

Do public Git repozitara patri:

- plugin manifest,
- public-safe skill instrukcie,
- public-safe KB a tool mapping,
- `.mcp.json` s URL bez secretov,
- README bez internych dat.

Do public Git repozitara nepatri:

- customer data,
- realne doklady,
- tokeny, API kluce, FTP pristupy,
- `config.local.php`,
- exporty zo SharePointu, ClickUpu, Teams, mailov alebo SuperFaktury,
- interne KB, ktore nie su schvalene ako public-safe.

Ak je KB interna iba pre SharePoint, skill v Gite musi vediet najst lokalny `AI Agent` root a precitat ju odtial. Ak root nenajde, musi povedat, ze interna KB nie je dostupna.

## Minimalne Overenie

Pred pushom:

- `python <skill-creator>/scripts/quick_validate.py <skill-dir>`
- `python <plugin-creator>/scripts/validate_plugin.py <plugin-dir>`
- JSON parse `.agents/plugins/marketplace.json`, `.codex-plugin/plugin.json`, `.mcp.json`.

Po pushi:

- `codex plugin marketplace upgrade <codex-marketplace-id>`
- skontroluj `~/.codex/config.toml` `last_revision`,
- skontroluj cache `~/.codex/plugins/cache/<codex-marketplace-id>/<plugin-id>/<version>/`,
- pri MCP over, ze `.mcp.json` je v cache a plugin manifest ma `mcpServers`,
- pri private MCP over, ze `codex mcp list` ukazuje `Auth OAuth`; `Authentication complete` v browseri bez zadania hesla znamena uspesny MS365 SSO callback,
- private MCP E2E over cez novy chat alebo `codex exec`; nikdy nie citanim `.codex/.credentials.json` a rucnym bearer tokenom.
