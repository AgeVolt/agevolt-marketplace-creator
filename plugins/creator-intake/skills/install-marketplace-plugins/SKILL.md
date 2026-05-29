---
name: install-marketplace-plugins
description: Pouzi po pridani alebo upgrade AgeVolt marketplace, ked treba nainstalovat vsetky default pluginy z marketplace manifestu. Nepouzivaj na tvorbu pluginu, zmenu skillu ani Git push.
---

# Install Marketplace Plugins

Tento skill zabezpeci AgeVolt standard: ked pouzivatel prida marketplace, nemusi
rucne vyklikavat kazdy plugin. Skill nacita marketplace manifest, nainstaluje
vsetky pluginy oznacene ako `INSTALLED_BY_DEFAULT` a overi vysledok.

## Hranica Skillu

Pouzi tento skill iba ked:

- pouzivatel prave pridal alebo upgradol AgeVolt marketplace,
- treba nainstalovat vsetky pluginy z jedneho marketplace,
- Creator alebo iny update workflow potrebuje overit, ze default pluginy su
  realne `installed, enabled`.

Nepouzivaj ho na:

- vytvorenie alebo upravu marketplace, pluginu, skillu, KB alebo MCP,
- rozhodovanie, do ktoreho marketplace obsah patri,
- Git commit alebo `git push`,
- rucne OAuth/token fallbacky.

## Postup

1. Zisti Codex marketplace ID, napriklad `creator`, `product-chargers` alebo
   `public-user-tools`.
2. Spusti `codex plugin marketplace list` a over, ze marketplace je pridany.
3. Nacitaj `.agents/plugins/marketplace.json` z nakonfigurovaneho marketplace
   rootu.
4. Vyber iba pluginy, ktore maju:

```json
"policy": {
  "installation": "INSTALLED_BY_DEFAULT"
}
```

5. Pre kazdy takyto plugin spusti:

```text
codex plugin add <plugin-name>@<marketplace-id>
```

6. Po instalacii spusti `codex plugin list` a over, ze kazdy default plugin je
   `installed, enabled`.
7. Vysledok zhrn: nainstalovane, uz nainstalovane, zlyhane, a pluginy vynechane
   preto, ze nemaju `INSTALLED_BY_DEFAULT`.

## Pravidla

- Samotne `codex plugin marketplace add` nestaci. Codex CLI po pridani
  marketplace nenainstaluje pluginy automaticky ani pri `INSTALLED_BY_DEFAULT`.
- `INSTALLED_BY_DEFAULT` je AgeVolt zamer v manifestoch; realnu instalaciu robi
  tento workflow cez `codex plugin add`.
- Skilly sa neinstaluju samostatne. Spristupnia sa instalaciou pluginu.
- Ak plugin pouziva MCP a ma `authentication: "ON_USE"`, nevyzaduj login pocas
  instalacie. Login ries az pri prvom realnom pouziti MCP workflowu.
- Ak plugin pouziva `authentication: "ON_INSTALL"` a Codex vyziada prihlasenie,
  pouzi standardny Codex/app auth flow. Necitaj tokeny, cookies ani
  `.codex/.credentials.json`.
- Neobchadzaj MCP cez `curl`, browser URL alebo shell HTTP fallback.
- Nerob `git push`. Tento skill meni iba lokalnu Codex instalaciu pluginov.

## Vystup

V odpovedi pouzivatelovi uveď:

- marketplace ID,
- pocet default pluginov,
- zoznam pluginov, ktore su `installed, enabled`,
- zoznam zlyhani s kratkym dovodom,
- ci treba otvorit novy chat alebo refreshnut Codex, aby sa nove skilly nacitali.
