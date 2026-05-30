---
name: update-marketplace
description: Pouzi iba ked sa pridava alebo upravuje plugin, skill, knowledge base, MCP, script alebo iny artefakt v uz existujucom AgeVolt marketplace. Tento skill robi implementacny update v SharePointe aj Git marketplace a overuje upgrade. Nepouzivaj na nejasny intake ani na vytvorenie noveho marketplace repo.
---

# Update Marketplace

Tento skill je pre update existujuceho AgeVolt marketplace. Pouzi ho pri poziadavkach typu "pridaj plugin", "pridaj skill", "dopln KB", "zapoj MCP", "uprav existujuci plugin", "publikuj update" alebo "nech si to vsetci vedia upgradnut".

## Hranica Skillu

Tento skill je implementacny update existujuceho marketplace:

- pridanie alebo uprava pluginu,
- pridanie alebo uprava skillu,
- pridanie alebo uprava KB,
- pridanie alebo uprava MCP,
- manifest update, version bump, priprava Git zmeny a Codex upgrade po schvalenom pushi.

Nepouzivaj ho, ked poziadavka este len rozhoduje, co ma vzniknut; vtedy pouzi `creator-intake`. Nepouzivaj ho na vytvorenie noveho marketplace repo; vtedy pouzi `create-marketplace`.

## Najprv Nacitaj Kontext

Najdi `AI Agent` root:

1. `AGEVOLT_AI_AGENT_ROOT`, ak existuje.
2. `%USERPROFILE%\OneDrive - AgeVolt Slovakia, s.r.o\Dokumenty - Produkt\AI Agent`.
3. Aktualny workspace alebo jeho rodic, ak sa vola `AI Agent`.

Ak root existuje, precitaj:

- `README.md`
- `marketplaces/agevolt-creator-marketplace/plugins/creator-intake/kb/rules.md`
- `marketplaces/agevolt-creator-marketplace/plugins/creator-intake/kb/marketplace-structure.md`
- `marketplaces/agevolt-creator-marketplace/plugins/creator-intake/kb/marketplace-catalog.md`
- `marketplaces/agevolt-creator-marketplace/plugins/creator-intake/kb/git-update-flow.md`
- `marketplaces/agevolt-creator-marketplace/plugins/creator-intake/kb/mcp-build-runbook.md`
- `marketplaces/agevolt-creator-marketplace/plugins/creator-intake/kb/distribution-feedback-model.md`

Ak root neexistuje, skus precitat bundlovane KB v tomto plugine:

- `../../kb/marketplace-structure.md`
- `../../kb/marketplace-catalog.md`
- `../../kb/git-update-flow.md`
- `../../kb/mcp-build-runbook.md`
- `../../kb/distribution-feedback-model.md`

Ak ani tie nie su dostupne, pouzi pravidla v tomto SKILL.md a povedz, ze interny SharePoint root sa nenasiel.

## Povinne Rozhodnutie

Najprv urci:

1. existujuci marketplace,
2. plugin,
3. typ zmeny: `new-plugin`, `new-skill`, `update-skill`, `new-kb`, `update-kb`, `new-mcp`, `update-mcp`, `manifest-update`,
4. ci je obsah public-safe pre Git.

Ak marketplace nesedi, nepresuvaj obsah nasilu. Pouzi `marketplace-catalog.md` a navrhni spravny marketplace alebo novy marketplace proposal.

## Povinne Poradie A Jazyk

Kazdy update existujuceho marketplace rob v tomto poradi:

1. Pred editom precitaj alebo vypis cielovy SharePoint source v
   `AI Agent/marketplaces/<marketplace-id>/...` aj zodpovedajuci Git checkout.
2. Porovnaj SharePoint a Git: subory iba v SharePointe, iba v Gite a rozdielne
   subory. Rozhodni, co je public-safe Git obsah a co je SharePoint-only private.
3. Uprav alebo vytvor SharePoint source v `AI Agent/marketplaces/<marketplace-id>/...`.
4. Rozdel obsah na interny/private a public-safe.
5. Do Git repozitara synchronizuj iba public-safe cast vratane public-safe KB,
   skillov, manifestov a README.
6. Po edite znova porovnaj SharePoint a Git public-safe cast. Git subory musia
   byt zhodne so SharePoint zdrojom alebo musia mat jasne zdokumentovanu
   public-safe projekciu.
7. Skontroluj jazyk vsetkych novych alebo upravenych `.md` suborov v SharePointe
   aj Gite; maju byt po slovensky okrem technickych identifikatorov a povolenych
   vynimiek.
8. Skontroluj, ze public Git skill neodkazuje na chybajucu KB bez access-gap
   pravidla. Ak skill odkazuje na public-safe KB, KB musi byt aj v Gite.
9. Bumpni verziu, validuj manifesty/skilly a az potom priprav lokalny commit alebo diff.
10. `git push` a marketplace upgrade ries az po explicitnom potvrdeni pouzivatela v aktualnom chate.

Git repo je distribucny/update kanal pre Codex, nie interny zdroj pravdy. Ak vznikne iba Git zmena bez SharePoint source, update je nekompletny.

Vsetky nove alebo upravovane `.md` subory pis po slovensky. Vynimky su technicke identifikatory, nazvy suborov, prikazy, JSON/YAML kluce, frontmatter kluce, API/tool nazvy, presne citacie alebo explicitna poziadavka pouzivatela na iny jazyk.

UI nazvy a kratke UI texty pre marketplace, plugin a skill pis po anglicky.
Toto plati pre `.agents/plugins/marketplace.json` `interface.displayName`,
`.codex-plugin/plugin.json` `interface.displayName`, `shortDescription`,
`defaultPrompt`, `plugin.yaml` `display_name` a `skills/*/agents/openai.yaml`
`display_name`, `short_description`, `default_prompt`.

Ak sa v SharePointe meni public-safe KB, README, skill alebo manifest, zodpovedajuca Git kopia musi byt aktualizovana v tej istej robote. Ak subor nesmie ist do Gitu, dopis alebo ponechaj jasne SharePoint-only/private oznacenie a uisti sa, ze Git skill pri jeho absencii nehada.

Pri kazdom novom alebo upravenom `.agents/plugins/marketplace.json` nastav realne instalovatelne AgeVolt pluginy defaultne na `policy.installation: "INSTALLED_BY_DEFAULT"`. Default `policy.authentication` je `ON_INSTALL`; `ON_USE` pouzi iba pri user-data alebo MCP OAuth pluginoch. Po schvalenom pushi a marketplace upgrade spusti `install-marketplace-plugins` alebo `codex plugin add <plugin>@<marketplace>` pre kazdy default plugin, lebo samotny `marketplace add/upgrade` pluginy cez CLI nenainstaluje.

Internu KB, raw exporty, zakaznicke data, produkcne dumpy, SQL exporty, slow logy, zmluvy a citlive podklady nedavaj do public Gitu. Ak public-safe skill potrebuje internu KB, musi ju hladat v lokalnom `AI Agent` roote a pri absencii nahlasit access gap.

Feedback od bezneho pouzivatela nikdy nezapisuj priamo do ostreho
`marketplaces/**` source. Najprv vytvor zaznam v `AI Agent/feedback/inbox/`.
Az admin triage rozhodne, ci ide o `update-kb`, `update-skill`, `new-skill`,
`update-mcp`, automatizaciu alebo ziadnu zmenu.

Bez potvrdenia pushu mozes pripravit SharePoint source, Git working tree, validacie, diff alebo lokalny commit na review. Nepovazuj vseobecne zadanie "oprav to", "implementuj plan" alebo "sprav update" za suhlas s pushom do `main`.

## SharePoint Struktura

Marketplace root musi ostat minimalny:

```text
marketplaces/<marketplace-id>/
  README.md
  marketplace.yaml
  plugins/
```

Plugin root:

```text
plugins/<plugin-id>/
  README.md
  plugin.yaml
```

Pridaj iba realne potrebne priecinky:

- skill -> `skills/<skill-id>/SKILL.md`
- skill UI -> `skills/<skill-id>/agents/openai.yaml`
- KB -> `kb/*.md`
- MCP -> `.mcp.json` plus `mcp/README.md`, ak treba vysvetlit server
- Codex plugin -> `.codex-plugin/plugin.json`

Nevytvaraj prazdne `templates/`, `tests/`, `mcp/`, `kb/`, `assets/` ani `scripts/`.

## Git Update

Kazdy public-safe update musi ist aj do Git repozitara marketplace:

```text
C:\AiAgent\repos\<marketplace-id>
```

Ak lokalny checkout neexistuje, naklonuj:

```powershell
gh repo clone AgeVolt/<marketplace-id> C:\AiAgent\repos\<marketplace-id>
```

Pri novom plugine uprav `.agents/plugins/marketplace.json` a pridaj entry s `source.path = "./plugins/<plugin-id>"`.

Pri update existujuceho pluginu bumpni `.codex-plugin/plugin.json` `version`.

## Skill Uniqueness Standard

Pri kazdom `new-skill` alebo `update-skill` v existujucom plugine:

1. Najprv precitaj vsetky `skills/*/SKILL.md` v cielovom plugine.
2. Porovnaj frontmatter `name`, `description`, trigger slova, scope, non-goals, povinne KB a pravidla.
3. Novy skill vytvor iba ked ma realne iny trigger, iny pouzivatelsky zamer, iny scope a ine pravidla ako existujuce skilly.
4. Ak je rozdiel iba v par vetach, dopln existujuci skill alebo presun spolocne pravidla do `kb/`.
5. Ak dva skilly hovoria skoro to iste, navrhni zlucenie a nevytvaraj treti podobny skill.
6. Pri update `SKILL.md` skontroluj, ci sa jeho `description` stale nelisi prilis malo od ostatnych skillov v plugine.
7. `description` musi byt pouzitelny ako trigger pre Codex: konkretne kedy skill pouzit, kedy ho nepouzit a cim sa lisi od susednych skillov.
8. Ak hranica medzi skillmi nie je jasna, zastav a vrat kratke porovnanie plus odporucanie `merge`, `split`, alebo `keep separate`.

Pri MCP pridaj alebo uprav:

```text
plugins/<plugin-id>/.mcp.json
plugins/<plugin-id>/.codex-plugin/plugin.json
```

`plugin.json` musi mat `mcpServers: "./.mcp.json"` iba ked `.mcp.json` realne existuje.

## MCP Update Standard

Pri kazdom `new-mcp`, `update-mcp`, `new-skill` alebo `update-skill`, ktory pouziva MCP:

- MCP tool names musia byt Codex/OpenAI kompatibilne: iba pismena, cisla, `_` alebo `-`, maximalne 64 znakov.
- Nepouzivaj bodky v MCP tool names. Z `sf.documents.list` urob `sf_documents_list`.
- Ak backend potrebuje stare bodkovane cesty, nech ostanu ako interne HTTP endpointy; MCP `tools/list` ma vracat Codex kompatibilne aliasy.
- Skill ma instruovat agenta, aby volal priamo MCP tooly a neobchadzal ich cez `curl`, `Invoke-RestMethod`, priame HTTP endpointy, `.codex/.credentials.json` alebo rucne bearer tokeny.
- Ak MCP tooly nie su v chate viditelne, skill ma zakazat HTTP/token fallback a najprv overit registraciu + OAuth login MCP servera. Pri private AgeVolt MCP pouzi `codex mcp login <mcp-server-id> --scopes MCP.Access`; po uspesnom login ma `codex mcp list` ukazat `Auth OAuth`. Potom otvor novy chat alebo sprav refresh/restart Codexu, aby sa tool surface nacital. Nepokracuj rucnym volanim MCP cez shell.
- Ak browser ukaze `Authentication complete` bez zadania hesla, ber to ako uspesny SSO login cez uz prihlaseny MS365 browser session.
- HTTP/streamable HTTP MCP server musi spravne ignorovat JSON-RPC notifications: `notifications/initialized` bez `id` nesmie vratit JSON-RPC response s `id: null`.
- `skills/<skill-id>/agents/openai.yaml` ma deklarovat MCP dependency:

```yaml
dependencies:
  tools:
    - type: "mcp"
      value: "<mcp-server-id>"
      description: "<human-readable server>"
      transport: "streamable_http"
      url: "https://..."
```

## Validacia

Pred ziadostou o schvalenie pushu spusti:

```powershell
python "C:\Users\Ján Zuštiak\.codex\skills\.system\skill-creator\scripts\quick_validate.py" "<skill-dir>"
python "C:\Users\Ján Zuštiak\.codex\skills\.system\plugin-creator\scripts\validate_plugin.py" "<plugin-dir>"
```

Pred tym este skontroluj:

- SharePoint source vs Git public-safe projekcia,
- zmenene `.md` subory su po slovensky,
- public Git neobsahuje private KB ani raw exporty,
- public Git skill neodkazuje na chybajuci public-safe KB subor,
- private SharePoint-only KB ma v skille jasne access-gap spravanie.

Pri MCP navyse over:

- `initialize` vrati validnu JSON-RPC response,
- `notifications/initialized` bez `id` vrati prazdne telo s HTTP `202` alebo `204`,
- `tools/list` vrati Codex kompatibilne nazvy bez bodiek,
- jeden read-only `tools/call` funguje priamo cez MCP,
- private MCP ma `codex mcp login <mcp-server-id> --scopes MCP.Access`, `codex mcp list` = `Auth OAuth` a novy chat alebo `codex exec` vidi tool bez shell fallbacku.

Skontroluj JSON:

```powershell
Get-Content "<json-path>" -Raw | ConvertFrom-Json
```

Potom:

```powershell
git -C "C:\AiAgent\repos\<marketplace-id>" add .
git -C "C:\AiAgent\repos\<marketplace-id>" commit -m "<kratky popis>"
```

Potom zastav a vypytaj si explicitne potvrdenie pouzivatela. Az po nom pokracuj:

```powershell
git -C "C:\AiAgent\repos\<marketplace-id>" push
codex plugin marketplace upgrade <codex-marketplace-id>
codex plugin add <plugin-id>@<codex-marketplace-id>
```

Ak marketplace obsahuje viac default pluginov, namiesto jednotlivych prikazov pouzi skill `install-marketplace-plugins` a over `codex plugin list`.

## Stop Pravidla

- Nemen firemny marketplace bez explicitnej poziadavky.
- Nepridavaj internu KB ani customer data do public Git.
- Nepridavaj MCP bez `.mcp.json` a bez validacie plugin manifestu.
- Nepridavaj plugin do marketplace JSON bez realneho `plugins/<plugin-id>`.
- Nevykonaj Git push bez explicitneho potvrdenia pouzivatela v aktualnom chate.
- Ak ma byt update dostupny ostatnym cez Codex, vysvetli, ze po review bude potrebny schvaleny Git push.
