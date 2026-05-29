# Postup Git Aktualizacie

Tento subor definuje, co sa musi zmenit v SharePointe a v Git marketplace, aby si zmenu vedeli ostatni pouzivatelia aktualizovat v Codexe.

## Zaklad

Codex vie automaticky upgradovat iba Git marketplace. SharePoint lokalny priecinok je interny zdroj pravdy, ale samotny Codex upgrade pre beznych pouzivatelov ide cez public Git repo marketplace.

Preto kazdy public-safe artefakt musi mat:

1. SharePoint source v `AI Agent/marketplaces/<marketplace-id>/`.
2. Git publikovatelny obsah v `https://github.com/AgeVolt/<marketplace-id>`.
3. Validovany Git working tree alebo lokalny commit pripraveny na review.
4. `git push` az po explicitnom potvrdeni pouzivatela v aktualnom chate.
5. Overenie cez `codex plugin marketplace upgrade <codex-marketplace-id>` po schvalenom pushi.

Poradie je zavazne: najprv SharePoint source, potom public-safe Git mirror. Ak agent vytvori alebo upravi iba Git bez SharePoint source, zmena je nekompletna.

Vsetky nove alebo upravovane `.md` subory pis po slovensky v SharePointe aj v Gite. Vynimky su technicke identifikatory, nazvy suborov, prikazy, JSON/YAML kluce, frontmatter kluce, API/tool nazvy, presne citacie alebo explicitna poziadavka pouzivatela na iny jazyk.

UI nazvy a kratke UI texty pre marketplace, plugin a skill su vynimka: pis ich
po anglicky. Tykaju sa hlavne `.agents/plugins/marketplace.json`
`interface.displayName`, `.codex-plugin/plugin.json` `interface.displayName`,
`shortDescription`, `defaultPrompt`, `plugin.yaml` `display_name` a
`skills/*/agents/openai.yaml` `display_name`, `short_description`,
`default_prompt`.

Git repo nesluzi ako interny zdroj pravdy ani sklad internych podkladov. Do Gitu zapisuj iba public-safe manifesty, public-safe skill workflowy, public-safe KB, tool mapping, instalacne/testovacie pokyny a odkazy na to, ako najst lokalny `AI Agent` root.

## Povinna SharePoint/Git Kontrola

Pri kazdom update existujuceho marketplace, pluginu, skillu alebo KB sprav
dvojstrannu kontrolu:

1. Pred editom vypis alebo precitaj relevantny SharePoint source v
   `AI Agent/marketplaces/<marketplace-id>/...`.
2. Pred editom vypis alebo precitaj zodpovedajuci Git checkout
   `repos/<marketplace-id>`.
3. Zisti rozdiel: subory iba v SharePointe, subory iba v Gite a subory na oboch
   stranach s inym obsahom.
4. Rozhodni pre kazdy rozdiel, ci ide o:
   - public-safe subor, ktory sa musi zosuladit do Gitu,
   - SharePoint-only private podklad, ktory nesmie ist do Gitu,
   - Git-only distribucny subor ako `.agents/plugins/marketplace.json`.
5. Po edite zopakuj porovnanie public-safe suborov a skontroluj, ze Git kopia je
   zhodna so SharePoint zdrojom alebo ma zdokumentovanu public-safe projekciu.
6. Skontroluj vsetky zmenene `.md` subory na jazyk. Text ma byt po slovensky;
   anglictina je povolena iba pri technickych identifikatoroch, klucoch, prikazoch,
   API/tool nazvoch, presnych citaciach alebo explicitnej poziadavke pouzivatela.
   UI nazvy a kratke UI texty musia byt po anglicky.
7. Skontroluj referencie zo skillov na KB/subory. Public Git skill nesmie odkazovat
   na chybajuci Git subor bez toho, aby jasne oznacil SharePoint-only private zdroj
   a access-gap spravanie.

Ak tato kontrola odhali, ze Git neobsahuje public-safe KB alebo skill, ktory je v
SharePointe a skill na neho odkazuje, dopln ho do Gitu. Ak by doplnenie odhalilo
internu KB, ponechaj ju iba v SharePointe a uprav skill tak, aby si ju nevymyslal.

## Push Approval Gate

`git push` do `main`, `master`, release branchu alebo inej zdielanej vetvy nikdy nerob automaticky. Pred pushom zastav a vypytaj si explicitne potvrdenie pouzivatela pre konkretnu zmenu a konkretny repo/branch.

Za potvrdenie sa pocita iba jasna veta v aktualnom chate, napriklad:

- "pushni to do main",
- "potvrdzujem push",
- "mozes to publikovat",
- "pushni tento commit".

Za potvrdenie sa nepocita vseobecne zadanie "oprav to", "implementuj plan", "sprav update", "dokonci to" ani to, ze GitHub remote funguje. Bez potvrdenia priprav diff alebo lokalny commit na review a v zavere napis, ze push caka na schvalenie.

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
    "installation": "INSTALLED_BY_DEFAULT",
    "authentication": "ON_INSTALL"
  },
  "category": "Productivity"
}
```

`authentication: "ON_USE"` pouzi iba pri pluginoch s user-data alebo MCP OAuth,
kde sa login ma vyziadat az pri prvom realnom pouziti.

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

1. Sprav povinnu SharePoint/Git kontrolu pred editom.
2. Uprav SharePoint source.
3. Uprav Git repo public-safe obsah.
4. Sprav povinnu SharePoint/Git kontrolu po edite vratane jazyka `.md` suborov.
5. Bumpni `plugins/<plugin-id>/.codex-plugin/plugin.json` `version` semverom.
6. Ak pribudol MCP server, pridaj `mcpServers` do `plugin.json` a `.mcp.json` do plugin rootu.
7. Ak MCP server zmizol, odstran `mcpServers` aj `.mcp.json`.
8. Priprav lokalny commit alebo diff na review.
9. Zastav a vypytaj si explicitne potvrdenie pred `git push`.
10. Po schvalenom pushi spusti `codex plugin marketplace upgrade <codex-marketplace-id>`.
11. Over, ze cache obsahuje novu verziu pluginu a novy skill/KB/MCP.
12. Pri private MCP over `codex mcp login <mcp-server-id> --scopes MCP.Access`, potom `codex mcp list` = `Auth OAuth` a novy chat/refresh Codexu.

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

Pred ziadostou o schvalenie pushu:

- porovnanie SharePoint source vs Git public-safe projekcia,
- kontrola, ze public-safe KB/skilly zmenene v SharePointe su aj v Gite,
- kontrola, ze private KB je iba v SharePointe a Git skill ma access-gap spravanie,
- kontrola slovenciny vo vsetkych novych alebo upravenych `.md` suboroch,
- `python <skill-creator>/scripts/quick_validate.py <skill-dir>`
- `python <plugin-creator>/scripts/validate_plugin.py <plugin-dir>`
- JSON parse `.agents/plugins/marketplace.json`, `.codex-plugin/plugin.json`, `.mcp.json`.

Po schvalenom pushi:

- `codex plugin marketplace upgrade <codex-marketplace-id>`
- `codex plugin add <plugin-id>@<codex-marketplace-id>` pre kazdy plugin s `INSTALLED_BY_DEFAULT`, alebo pouzi skill `install-marketplace-plugins`
- skontroluj `~/.codex/config.toml` `last_revision`,
- skontroluj cache `~/.codex/plugins/cache/<codex-marketplace-id>/<plugin-id>/<version>/`,
- pri MCP over, ze `.mcp.json` je v cache a plugin manifest ma `mcpServers`,
- pri private MCP over, ze `codex mcp list` ukazuje `Auth OAuth`; `Authentication complete` v browseri bez zadania hesla znamena uspesny MS365 SSO callback,
- private MCP E2E over cez novy chat alebo `codex exec`; nikdy nie citanim `.codex/.credentials.json` a rucnym bearer tokenom.
