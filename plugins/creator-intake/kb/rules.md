# Creator Intake Rules

## Decision Matrix

Vyber najmensi artefakt, ktory realne riesi poziadavku.

| Artefakt | Kedy ho vytvorit |
| --- | --- |
| Marketplace | Ina cielova skupina, viditelnost, citlivost alebo update kanal. |
| Plugin | Instalovatelny pracovny blok, ktory dava zmysel zapnut samostatne. |
| Skill | Opakovatelny agenticky postup s jasnym triggerom. |
| KB/reference | Hlavna hodnota su znalosti, schema, procesy, priklady alebo dokumenty. |
| MCP tool | Treba zive data, API, vypocet alebo akciu v inom systeme. |
| MCP resource | Server poskytuje citatelny dynamicky kontext. |
| MCP prompt | Server poskytuje pouzivatelom vyberatelny prompt template. |
| Firemne rule | Pravidlo plati pre viac ludi a ma ho zdedit novy clovek v rovnakej roli. |
| Personal rule | Preferencia jedneho cloveka. |
| Script | Deterministicka opakovatelna operacia. |

## Stop Rules

- Nevytvaraj firemny skill len preto, ze pouzivatel povedal "sprav skill".
- Neupravuj firemne marketplace/plugin/skill/KB/MCP/rule bez explicitnej poziadavky.
- Nekopiruj stare `AI/` priecinky cele.
- Nedavaj internu KB, customer data, secrets ani systemove exporty do public Git.
- Pri MCP write akcii vyzaduj read/preview a explicitne potvrdenie.

## Required Output Pri Nejasnej Poziadavke

Vrat artifact proposal alebo sa opytaj tri intake otazky.

Nepokracuj implementaciou, ak nie je jasne:

- kto to pouziva,
- ci ide o osobnu alebo firemnu vec,
- ci je potrebny zapis do systemu,
- ako to otestujeme.
