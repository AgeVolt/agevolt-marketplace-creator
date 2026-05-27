# AgeVolt Marketplace Katalog

Toto je aktualny katalog AgeVolt Codex marketplaces. Cisty agent ma tento subor precitat pred vytvorenim alebo upravou marketplace, pluginu, skillu, KB alebo MCP.

## Root

SharePoint root:

```text
C:\Users\Ján Zuštiak\OneDrive - AgeVolt Slovakia, s.r.o\Dokumenty - Produkt\AI Agent
```

Marketplace root:

```text
marketplaces/
```

## Marketplaces

| Marketplace ID | Codex ID | Git repo | Stav | Scope |
| --- | --- | --- | --- | --- |
| `agevolt-creator-marketplace` | `creator` | `https://github.com/AgeVolt/agevolt-creator-marketplace` | active | Creator pre navrh, tvorbu a update AgeVolt AI artefaktov. |
| `agevolt-personal-productivity-marketplace` | `personal-productivity` | `https://github.com/AgeVolt/agevolt-personal-productivity-marketplace` | planned-empty | Osobne maily, kalendar, tasky, follow-upy, digesty, personal rules. |
| `agevolt-finance-admin-marketplace` | `finance-admin` | `https://github.com/AgeVolt/agevolt-finance-admin-marketplace` | active-build | Financie, administracia, SuperFaktura, faktury, pohladavky, reporty. |
| `agevolt-production-marketplace` | `production` | `https://github.com/AgeVolt/agevolt-production-marketplace` | planned-empty | Vyroba, servisna operativa, sklad, seriove cisla, kvalita, material. |
| `agevolt-support-helpdesk-marketplace` | `support-helpdesk` | `https://github.com/AgeVolt/agevolt-support-helpdesk-marketplace` | planned-empty | Support, helpdesk, incidenty, odpovede, eskalacie, support KB. |
| `agevolt-product-myagevolt-marketplace` | `product-myagevolt` | `https://github.com/AgeVolt/agevolt-product-myagevolt-marketplace` | planned-empty | myAgeVolt portal, portalova KB, produktove pravidla, developer tools. |
| `agevolt-product-chargers-marketplace` | `product-chargers` | `https://github.com/AgeVolt/agevolt-product-chargers-marketplace` | planned-empty | Nabijacky, cenniky, navody, produktova KB, sluzby okolo zariadeni. |
| `agevolt-public-user-tools-marketplace` | `public-user-tools` | `https://github.com/AgeVolt/agevolt-public-user-tools-marketplace` | planned-empty | Public-safe pouzivatelske navody, troubleshooting a verejne integracie. |

## Vyber Marketplace

- Ak ide o SuperFakturu, faktury, ponuky, objednavky, pohladavky, platby alebo financne reporty, pouzi `agevolt-finance-admin-marketplace`.
- Ak ide o myAgeVolt portal, portalovu KB, frontend/backend portal workflowy alebo repo pravidla k portalu, pouzi `agevolt-product-myagevolt-marketplace`.
- Ak ide o nabijacky, cenniky, produktove navody, servisne/sluzbove baliky alebo produktovu KB k zariadeniam, pouzi `agevolt-product-chargers-marketplace`.
- Ak ide o osobnu pracovnu rutinu jedneho cloveka, pouzi `agevolt-personal-productivity-marketplace`, iba ak to nie je osobny subor mimo firemneho marketplace.
- Ak ide o support alebo helpdesk, pouzi `agevolt-support-helpdesk-marketplace`.
- Ak ide o vyrobu, sklad alebo seriove cisla, pouzi `agevolt-production-marketplace`.
- Ak ma byt obsah verejny pre zakaznikov alebo public-safe pouzivatelov, pouzi `agevolt-public-user-tools-marketplace`.
- Novy marketplace vytvor iba ked ziaden existujuci marketplace nesedi alebo je potrebny iny pristup, citlivost alebo update kanal.
