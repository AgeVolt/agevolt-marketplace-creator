# Postup Git Aktualizacie

Tento subor definuje, co sa musi zmenit v SharePointe a v Git marketplace, aby si zmenu vedeli ostatni pouzivatelia aktualizovat v Codexe.

## Zaklad

Codex vie automaticky upgradovat iba Git marketplace. SharePoint lokalny priecinok je interny zdroj pravdy, ale samotny Codex upgrade pre beznych pouzivatelov ide cez verejny Git repo marketplace.

Preto kazdy verejne bezpecny artefakt musi mat:

1. SharePoint zdroj v `AI Agent/marketplaces/<marketplace-id>/`.
2. Git publikovatelny obsah v `https://github.com/AgeVolt/<marketplace-id>`.
3. Validovany Git working tree alebo lokalny commit pripraveny na kontrolu.
4. `git push` az po explicitnom potvrdeni pouzivatela v aktualnom chate.
5. Overenie cez `codex plugin marketplace upgrade <codex-marketplace-id>` po schvalenom pushi.

Poradie je zavazne: najprv SharePoint zdroj, potom verejne bezpecna Git kopia. Ak agent vytvori alebo upravi iba Git bez SharePoint zdroja, zmena je nekompletna.

Vsetky nove alebo upravovane `.md` subory pis po slovensky v SharePointe aj v Gite. Vynimky su technicke identifikatory, nazvy suborov, prikazy, JSON/YAML kluce, frontmatter kluce, API/tool nazvy, presne citacie alebo explicitna poziadavka pouzivatela na iny jazyk.

Git repo nesluzi ako interny zdroj pravdy ani sklad internych podkladov. Do Gitu zapisuj iba verejne bezpecne manifesty, verejne bezpecne skillove postupy, verejne bezpecnu KB, mapovanie toolov, instalacne/testovacie pokyny a odkazy na to, ako najst lokalny `AI Agent` root.

## Povinna SharePoint/Git Kontrola

Pri kazdej implementacnej zmene v existujucom marketplace kontroluj obidve vrstvy:

1. Najdi SharePoint zdroj v `AI Agent/marketplaces/<marketplace-id>/...`.
2. Najdi realny Git checkout, ktory pouzivatel kontroluje. Ak pouzivatel uviedol konkretnu cestu, pracuj v nej.
3. Pred editom precitaj relevantne README, manifesty, `SKILL.md`, `agents/openai.yaml` a KB v SharePointe aj v Gite.
4. Po edite over, ze verejne bezpecne casti su zosuladene v oboch miestach.
5. Over, ze sukromna KB alebo raw exporty ostali iba v SharePointe a Git ma iba access-gap pravidlo alebo verejne bezpecny suhrn.
6. Over jazyk: nove alebo upravene `.md` texty maju byt po slovensky okrem technickych identifikatorov a povolenych vynimiek.
7. Over referencie: kazdy skill musi odkazovat iba na existujuce KB/assets/MCP subory alebo musi vysvetlit, ze ide o sukromny access gap.

## Push Approval Gate

`git push` do `main`, `master`, release branchu alebo inej zdielanej vetvy nikdy nerob automaticky. Pred pushom zastav a vypytaj si explicitne potvrdenie pouzivatela pre konkretnu zmenu a konkretny repo/branch.

Za potvrdenie sa pocita iba jasna veta v aktualnom chate, napriklad:

- "pushni to do main",
- "potvrdzujem push",
- "mozes to publikovat",
- "pushni tento commit".

Za potvrdenie sa nepocita vseobecne zadanie "oprav to", "implementuj plan", "sprav update", "dokonci to" ani to, ze GitHub remote funguje. Bez potvrdenia priprav diff alebo lokalny commit na kontrolu a v zavere napis, ze push caka na schvalenie.

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

V Git repozitari `<marketplace-id>` pridaj verejne bezpecny plugin:

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

1. Najdi a precitaj SharePoint zdroj aj realny Git checkout.
2. Uprav SharePoint zdroj.
3. Rozdel obsah na sukromny a verejne bezpecny.
4. Uprav Git repo verejne bezpecny obsah v tom istom update.
5. Skontroluj, ze jazyk upravenych `.md` suborov je slovensky.
6. Skontroluj, ze `SKILL.md`, `agents/openai.yaml`, KB a manifesty na seba odkazuju bez chybajucich suborov.
7. Bumpni `plugins/<plugin-id>/.codex-plugin/plugin.json` `version` semverom.
8. Ak pribudol MCP server, pridaj `mcpServers` do `plugin.json` a `.mcp.json` do plugin rootu.
9. Ak MCP server zmizol, odstran `mcpServers` aj `.mcp.json`.
10. Validuj skill, plugin a JSON/YAML manifesty.
11. Priprav lokalny commit alebo diff na kontrolu.
12. Zastav a vypytaj si explicitne potvrdenie pred `git push`.
13. Po schvalenom pushi spusti `codex plugin marketplace upgrade <codex-marketplace-id>`.
14. Over, ze cache obsahuje novu verziu pluginu a novy skill/KB/MCP.
15. Pri sukromnom MCP over `codex mcp login <mcp-server-id> --scopes MCP.Access`, potom `codex mcp list` = `Auth OAuth` a novy chat/refresh Codexu.

Bez version bumpu moze byt tazsie overit, ci sa pouzivatelovi naozaj refreshol plugin cache.

## Verejne Bezpecna Hranica

Do verejneho Git repozitara patri:

- plugin manifest,
- verejne bezpecne skill instrukcie,
- verejne bezpecna KB a mapovanie toolov,
- `.mcp.json` s URL bez secretov,
- README bez internych dat.

Do verejneho Git repozitara nepatri:

- zakaznicke data,
- realne doklady,
- tokeny, API kluce, FTP pristupy,
- `config.local.php`,
- exporty zo SharePointu, ClickUpu, Teams, mailov alebo SuperFaktury,
- interne KB, ktore nie su schvalene ako verejne bezpecne.

Ak je KB interna iba pre SharePoint, skill v Gite musi vediet najst lokalny `AI Agent` root a precitat ju odtial. Ak root nenajde, musi povedat, ze interna KB nie je dostupna.

## Minimalne Overenie

Pred ziadostou o schvalenie pushu:

- `python <skill-creator>/scripts/quick_validate.py <skill-dir>`
- `python <plugin-creator>/scripts/validate_plugin.py <plugin-dir>`
- JSON parse `.agents/plugins/marketplace.json`, `.codex-plugin/plugin.json`, `.mcp.json`.
- porovnanie upravenych verejne bezpecnych suborov medzi SharePoint zdrojom a Git checkoutom,
- kontrola chybajucich KB/assets/MCP referencii zo `SKILL.md`, `agents/openai.yaml` a manifestov,
- kontrola jazyka v upravenych `.md` suboroch.

Po schvalenom pushi:

- `codex plugin marketplace upgrade <codex-marketplace-id>`
- skontroluj `~/.codex/config.toml` `last_revision`,
- skontroluj cache `~/.codex/plugins/cache/<codex-marketplace-id>/<plugin-id>/<version>/`,
- pri MCP over, ze `.mcp.json` je v cache a plugin manifest ma `mcpServers`,
- pri sukromnom MCP over, ze `codex mcp list` ukazuje `Auth OAuth`; `Authentication complete` v browseri bez zadania hesla znamena uspesny MS365 SSO callback,
- sukromne MCP E2E over cez novy chat alebo `codex exec`; nikdy nie citanim `.codex/.credentials.json` a rucnym bearer tokenom.
