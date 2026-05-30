# Creator Intake

Creator Intake je prvy plugin marketplace `agevolt-creator-marketplace`.

Pouzi ho pri poziadavkach typu:

- sprav skill,
- vytvor marketplace,
- urob plugin,
- potrebujem MCP,
- vytvor knowledge base,
- premigruj nieco zo stareho AI,
- daj to pre vsetkych.

Hlavne pravidlo: nevytvaraj artefakt naslepo. Najprv rozhodni, ci poziadavka
patri do marketplace, pluginu, skillu, KB, MCP, pravidla, osobneho pravidla
alebo skriptu.

## Struktura Pluginu

```text
creator-intake/
  README.md
  plugin.yaml
  skills/
    create-marketplace/
    update-marketplace/
    install-marketplace-plugins/
    mcp-websupport/
  kb/
    rules.md
    marketplace-structure.md
    marketplace-catalog.md
    git-update-flow.md
    distribution-feedback-model.md
    mcp-build-runbook.md
    artifact-proposal.md
```

`skills/` obsahuje spustitelne agenticke postupy. `kb/` obsahuje dlhsie pravidla, strukturu a vystupne formaty. `mcp/` sa vytvori az vtedy, ked plugin realne potrebuje zive API alebo externy system.

Pri tvorbe alebo oprave MCP najprv citaj `kb/mcp-build-runbook.md`. Ten definuje
strukturu MCP pluginu, WebSupport server code, auth cez zdielany AgeVolt OAuth
Broker, zakaz manualnych token fallbackov a povinne E2E testy.

Pri pridani alebo upgrade marketplace pouzi `install-marketplace-plugins`, aby
sa vsetky pluginy oznacene ako `INSTALLED_BY_DEFAULT` nainstalovali cez
`codex plugin add <plugin>@<marketplace>`. Samotne pridanie marketplace cez
Codex CLI pluginy nenainstaluje a skilly sa spristupnia az s pluginom.

Pri beznych pouzivateloch rozlisuj dve veci:

- Git marketplace je install/update balik pre Codex.
- SharePoint `AI Agent/marketplaces/**` je admin source of truth.

Ak pouzivatel hlasi problem, navrhuje zmenu alebo je frustrovany z vystupu AI,
neved ho k uprave ostreho marketplace source. Feedback ma ist do
`AI Agent/feedback/inbox/` a az po admin triage sa zapracuje do KB, skillu,
pluginu alebo MCP.

## Jazyk A Ulozisko

Vsetky nove alebo upravovane `.md` subory, ktore Creator vytvara pre AgeVolt marketplaces, pluginy, skilly, KB alebo MCP dokumentaciu, pis po slovensky. Technicke identifikatory, nazvy suborov, prikazy, JSON/YAML kluce, frontmatter kluce, API/tool nazvy a presne citacie mozu ostat v povodnom tvare.

UI nazvy a kratke UI texty pre marketplaces, pluginy a skilly pis po anglicky.
Patri sem `displayName`, `display_name`, `shortDescription`,
`short_description`, `defaultPrompt` a `default_prompt`.

Pri implementacnej zmene najprv uprav SharePoint zdroj v
`AI Agent/marketplaces/<marketplace-id>/...`. Verejny Git pouzi az potom ako
verejne bezpecnu kopiu a aktualizacny kanal pre Codex. Git nie je miesto pre
internu KB, surove exporty, zakaznicke data ani produkcne podklady.

Nikdy nerob `git push` do `main`, `master`, release branchu ani inej zdielanej vetvy bez explicitneho potvrdenia pouzivatela v aktualnom chate. Bez potvrdenia priprav iba diff alebo lokalny commit na review a zastav sa pred pushom.

Ak chyba kontext, poloz najviac tri otazky:

1. Kto to bude pouzivat a kto to nema vidiet?
2. Ake su 2-3 realne poziadavky, ktore ma agent zvladnut?
3. Ma agent iba navrhovat, alebo aj citat/zapisovat do systemov?
