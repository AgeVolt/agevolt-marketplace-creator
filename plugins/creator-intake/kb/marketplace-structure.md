# AgeVolt Marketplace Struktura

Toto je aktualny standard pre AgeVolt AI Agent marketplaces.

## Co Je Co

| Vrstva | Uloha | Kedy ju vytvorit |
| --- | --- | --- |
| Marketplace | Katalog pluginov pre cielovu skupinu alebo domenu. | Ked sa lisi publikum, citlivost, instalacia alebo update kanal. |
| Plugin | Instalovatelny pracovny blok v marketplace. | Ked dava zmysel zapnut/vypnut celu oblast prace samostatne. |
| Skill | Agenticky postup alebo reference workflow s jasnym triggerom. | Ked sa opakuje postup, checklist, sposob rozhodovania alebo pouzitia nastrojov. |
| KB | Znalosti, pravidla, priklady, schema, procesy. | Ked hlavna hodnota nie je postup, ale obsah, ktory skill cita podla potreby. |
| MCP | Zive nastroje, dynamicke zdroje alebo prompt templates cez server. | Ked agent potrebuje API, databazu, vypocet, zapis alebo aktualne data. |
| Template | Vzor vystupu alebo suboru. | Ked chceme konzistentny format navrhu, dokumentu alebo konfiguracie. |
| Test | Minimalny scenar overenia. | Pri kazdom artefakte, ktory ma byt opakovane pouzivany alebo distribuovany. |

## Standardny Marketplace

```text
<marketplace-id>/
  README.md
  marketplace.yaml
  plugins/
```

Na urovni marketplace nedrz prazdne `kb/`, `mcp/`, `templates/` ani `tests/`. Marketplace iba popisuje cielovu skupinu a zoskupuje instalovatelne pluginy.

## Naming Standard

Marketplace musi mat tri zosuladene nazvy:

| Miesto | Standard | Priklad |
| --- | --- | --- |
| SharePoint priecinok | `agevolt-<oblast>-marketplace` | `agevolt-finance-admin-marketplace` |
| Git repo | `AgeVolt/agevolt-<oblast>-marketplace` | `AgeVolt/agevolt-creator-marketplace` |
| Codex marketplace ID | kratky slug bez `agevolt-` a bez `-marketplace` | `finance-admin` |
| Codex display name | slug bez finalneho `-marketplace`, citatelny Title Case | `AgeVolt Creator` |

Pravidla:

- Slug vzdy konci na `-marketplace`.
- Slug pouziva iba lowercase ASCII, cisla a pomlcky.
- SharePoint priecinok a Git repo musia mat rovnaky slug.
- `marketplace.yaml` `id` musi byt rovnaky slug.
- `marketplace.yaml` `codex_marketplace_id` musi byt kratky Codex marketplace ID.
- `.agents/plugins/marketplace.json` top-level `name` musi byt kratky Codex marketplace ID, nie dlhy Git/SharePoint slug.
- Kratky Codex marketplace ID sa zobrazuje vedla pluginu v Codex UI, preto nesmie obsahovat `agevolt` ani `marketplace`, ak to nie je nutne.
- Codex display name vznikne zo slugu bez posledneho `-marketplace`: `agevolt-creator-marketplace` -> `AgeVolt Creator`.
- Display name nikdy nekonci slovom `Marketplace`, ak je to iba technicky suffix slugu.
- `interface.displayName` nesmie byt rovnaky pre viac AgeVolt marketplaces, inak je Codex UI neprehladne.
- Nepouzivaj genericke display names ako `Built by AgeVolt`.
- Priklady: `agevolt-finance-admin-marketplace` -> Codex ID `finance-admin`, display `AgeVolt Finance Admin`; `agevolt-product-myagevolt-marketplace` -> Codex ID `product-myagevolt`, display `AgeVolt Product myAgeVolt`.

## Standardny Plugin

```text
plugins/<plugin-id>/
  README.md
  plugin.yaml
  skills/
  kb/
  mcp/
```

Aj v plugine vytvaraj iba priecinky s realnym obsahom. `skills/` vytvor, ked plugin obsahuje skill. `kb/` vytvor, ked skill potrebuje reference alebo pravidla. `mcp/` vytvor az ked existuje realny server alebo konfiguracia. V public Git Codex baliku musi mat plugin navyse `.codex-plugin/plugin.json`. Ak ma plugin MCP konfiguraciu pre Codex, pouzi Codex kompatibilny subor `.mcp.json` v plugin package.

## Plugin UI Standard

Kazdy AgeVolt plugin ma mat v `.codex-plugin/plugin.json`:

- kratke `interface.displayName`, idealne do 24 znakov,
- `interface.developerName: "AgeVolt"`,
- `interface.brandColor: "#280046"`,
- `interface.composerIcon: "./assets/icon.png"`,
- `interface.logo: "./assets/logo.png"`,
- realne subory `assets/icon.png` a `assets/logo.png` s AV logom.

Kazdy skill, ktory ma `agents/openai.yaml`, ma mat:

```yaml
interface:
  icon_small: "./assets/icon.png"
  icon_large: "./assets/logo.png"
  brand_color: "#280046"
```

Subory `assets/icon.png` a `assets/logo.png` drzte pri danom skille, aby cesty fungovali aj po instalacii do Codex cache. Nepouzivaj genericke emoji, nahodne ikonky alebo ine logo, ak pouzivatel explicitne neziada branded plugin tretej strany.

## Pravidlo Urovni

- Nedavaj skill pod MCP server.
- Nedavaj MCP server pod skill.
- V plugine su skill a MCP surodenci.
- Skill vysvetluje workflow a bezpecne pouzitie.
- MCP poskytuje nastroje, resources alebo prompts.
- KB drzi znalosti, ktore by zbytocne zatazovali `SKILL.md`.

## Pravidlo Jazyka A Uloziska

Vsetky `.md` subory vytvarane alebo upravovane v AgeVolt marketplaces pis po slovensky. Anglictina je povolena iba pre technicke identifikatory, nazvy suborov, prikazy, JSON/YAML kluce, frontmatter kluce, API/tool nazvy, presne citacie alebo explicitne vyziadany iny jazyk.

SharePoint `AI Agent/marketplaces/<marketplace-id>/...` je zdroj pravdy pre marketplace, pluginy, skilly, KB a MCP dokumentaciu. Public Git repo je mirror len pre public-safe distribuciu do Codexu. Pri kazdej zmene dodrz poradie:

1. uprav alebo vytvor SharePoint source,
2. oddel interny/private obsah od public-safe obsahu,
3. zosynchronizuj iba public-safe cast do Git repo,
4. bumpni verziu, validuj a az potom publikuj update.

Ak sa znalost pouziva vo viacerych skilloch v jednom plugine, patri na plugin-level KB. Ak sa pouziva vo viacerych pluginoch jedneho marketplace, patri na marketplace-level alebo spolocnu marketplace KB iba po explicitnom schvaleni struktury. Internu KB neduplikuj do Gitu; public Git skill ma odkazovat na lokalny `AI Agent` root alebo nahlasit access gap.

## Skill Boundary Standard

V jednom plugine nesmu vznikat skilly s rovnakym ucelom len preto, ze pouzivatel poziada o "novy skill".

Kazdy skill musi mat:

- jedinecny frontmatter `description`,
- jasny trigger,
- jasny scope a non-goals,
- vlastne rules alebo vlastny workflow,
- pomenovane rozdiely voci ostatnym skillom v tom istom plugine.

Pred `new-skill` alebo `update-skill` vzdy precitaj vsetky `skills/*/SKILL.md` v danom plugine. Ak sa novy obsah prekryva s existujucim skillom, preferuj:

1. upravit existujuci skill,
2. vytiahnut spolocne pravidla do `kb/`,
3. zlucit podobne skilly,
4. vytvorit novy skill az ked rozdiel ostava jasny.

Duplicitny signal:

- description sa spusta na tie iste poziadavky,
- oba skilly citaju rovnaku KB a pouzivaju rovnake tooly,
- lisia sa iba nazvom oddelenia alebo jednou vetou,
- jeden skill je iba uzsia cast druheho bez vlastnych pravidiel.

## MCP Tool Naming Standard

MCP server moze interne volat lubovolne API endpointy, ale tooly vystavene do Codexu musia mat nazvy, ktore model vie priamo volat.

Pravidla:

- Tool name: iba `A-Z`, `a-z`, `0-9`, `_`, `-`, najviac 64 znakov.
- Nepouzivaj bodky. `sf.documents.list` je zly tool name pre Codex; pouzi `sf_documents_list`.
- Ak kvoli kompatibilite existuju stare HTTP endpointy s bodkami, nech zostanu ako HTTP endpointy, ale MCP `tools/list` ma vracat underscore aliasy.
- Skill ma pomenovavat priamo MCP tooly, ktore ma agent pouzit.
- Skill nesmie odporucat obchadzanie MCP cez shell, `curl`, `Invoke-RestMethod`, HTTP URL fallback, citanie `.codex/.credentials.json` alebo rucny bearer token.
- Ak tooly nie su viditelne, pouzivatelsky workflow sa ma zastavit a nesmie prejst na HTTP/token fallback. Najprv over MCP registraciu a OAuth login; pri private AgeVolt MCP pouzi `codex mcp login <mcp-server-id> --scopes MCP.Access`, potom novy chat/refresh/restart Codexu. Upgrade/reinstall ries az po tom.

Pri skille, ktory zavisi na MCP, pridaj do `skills/<skill-id>/agents/openai.yaml`:

```yaml
dependencies:
  tools:
    - type: "mcp"
      value: "<mcp-server-id>"
      description: "<human-readable server>"
      transport: "streamable_http"
      url: "https://..."
```

Pri plugine, ktory zavisi na MCP, musi existovat:

```text
plugins/<plugin-id>/.mcp.json
plugins/<plugin-id>/.codex-plugin/plugin.json
```

`plugin.json` ma obsahovat `mcpServers: "./.mcp.json"` iba ked `.mcp.json` existuje.

## MCP OAuth Onboarding Standard

Pre private AgeVolt MCP nepouzivaj priame Entra metadata ako authorization server. MCP protected resource metadata ma ukazovat na shared broker:

```text
https://documents.agevolt.com/mcp/auth
```

Po instalacii pluginu over login:

```text
codex mcp login <mcp-server-id> --scopes MCP.Access
codex mcp list
```

`Auth OAuth` v `codex mcp list` znamena, ze Codex ma ulozene OAuth credentials. Browser moze ukazat len `Authentication complete`, ak pouzivatel uz ma aktivny MS365 SSO session; to je uspesny stav. Po prvom login treba otvorit novy chat alebo refreshnut/restartovat Codex, aby sa MCP tooly nacitali do aktualneho tool surface.

Neoveruj produkcny private MCP citanim `%USERPROFILE%\.codex\.credentials.json`. End-to-end overenie rob cez novy chat alebo izolovany `codex exec`, aby sa potvrdilo, ze tooly su realne vystavene do Codex tool surface.

## MCP Transport Handshake Standard

Pri HTTP/streamable HTTP MCP serveri nestaci, ze endpoint funguje cez `curl` alebo REST fallback. Server musi prejst realnym MCP JSON-RPC handshake:

- `initialize` s `id` vracia validnu JSON-RPC response.
- `notifications/initialized` bez `id` je JSON-RPC notification a nesmie vratit JSON-RPC response s `id: null`; vrat prazdne telo s HTTP `202` alebo `204`.
- `tools/list` po handshake vracia Codex kompatibilne nazvy toolov.
- `tools/call` musi fungovat priamo s tymi istymi nazvami, ktore vracia `tools/list`.
- Ak server odpoveda na notification ako na request, Codex nemusi MCP v chate vobec vystavit, aj ked plugin vyzera nainstalovany.

## Kedy Vytvorit Novy Marketplace

Novy marketplace vytvor iba ked aspon jedna vec plati:

- ina cielova skupina ho ma instalovat samostatne,
- obsah ma inu citlivost alebo pristup,
- update kanal ma byt oddeleny,
- pluginy by bez rozdelenia zbytocne zahltili nepovolane role,
- public-safe a interne veci by sa miesali.

Ak ide iba o dalsi pracovny blok pre rovnaku skupinu, vytvor plugin v existujucom marketplace.

## Minimalny Postup Pri Novom Marketplace

1. Najprv priprav artifact proposal.
2. Over publikum, citlivost a public/private hranicu.
3. V SharePointe vytvor minimalnu strukturu marketplace.
4. V public Git repozitari vytvor iba Codex marketplace katalog: README a `.agents/plugins/marketplace.json`.
5. Pluginy pridavaj az po schvaleni prveho minimalneho scenara.
6. Pri kazdej firemnej zmene zapis `revision-history.md`, ak marketplace uz ma revision history.
