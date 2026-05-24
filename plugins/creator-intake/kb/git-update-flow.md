# Git Update Flow

Tento subor definuje, co sa musi zmenit v SharePointe a v Git marketplace, aby si zmenu vedeli ostatni pouzivatelia aktualizovat v Codexe.

## Zaklad

Codex vie automaticky upgradovat iba Git marketplace. SharePoint lokalny priecinok je interny source of truth, ale samotny Codex upgrade pre beznych pouzivatelov ide cez public Git repo marketplace.

Preto kazdy public-safe artefakt musi mat:

1. SharePoint source v `AI Agent/marketplaces/<marketplace-id>/`.
2. Git publikovatelny obsah v `https://github.com/AgeVolt/<marketplace-id>`.
3. Commit a push do Git repozitara.
4. Overenie cez `codex plugin marketplace upgrade <marketplace-id>`.

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
  "name": "<marketplace-id>",
  "interface": {
    "displayName": "<Human Name>"
  },
  "plugins": []
}
```

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

V SharePoint `marketplace.yaml` pridaj plugin do `plugins`.

## Pri Update Existujuceho Pluginu

Pri zmene skillu, KB, MCP alebo plugin manifestu:

1. Uprav SharePoint source.
2. Uprav Git repo public-safe obsah.
3. Bumpni `plugins/<plugin-id>/.codex-plugin/plugin.json` `version` semverom.
4. Ak pribudol MCP server, pridaj `mcpServers` do `plugin.json` a `.mcp.json` do plugin rootu.
5. Ak MCP server zmizol, odstran `mcpServers` aj `.mcp.json`.
6. Commitni a pushni Git repo.
7. Spusti `codex plugin marketplace upgrade <marketplace-id>`.
8. Over, ze cache obsahuje novu verziu pluginu a novy skill/KB/MCP.

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

- `codex plugin marketplace upgrade <marketplace-id>`
- skontroluj `~/.codex/config.toml` `last_revision`,
- skontroluj cache `~/.codex/plugins/cache/<marketplace-id>/<plugin-id>/<version>/`,
- pri MCP over, ze `.mcp.json` je v cache a plugin manifest ma `mcpServers`.
