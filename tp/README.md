# Travaux pratiques — mode d'emploi

Chaque énoncé se fait **juste après le chapitre** qu'il pratique. Le corrigé est
un **corrigé formatif** : il reste dans ce dépôt public et sert à s'auto-corriger.
Le TP noté du cours, lui, est évalué séparément ; son corrigé vit dans le dépôt
privé `cours-securite-vulnerabilites-attaques-Peda`.

> ⚖️ **Cadre légal.** Toutes les manipulations de ce cours se pratiquent sur
> **vos propres conteneurs** (lab Docker) et sur des données fictives. Accéder ou
> se maintenir sans droit dans un système d'autrui est un délit (articles 323-1
> et suivants du Code pénal), même « pour tester ». Les tests d'intrusion réels
> se font sous contrat, dans un périmètre défini.

## Table des exercices

| Niveau | Énoncé | Chapitre lié | Corrigé |
| --- | --- | --- | --- |
| 🟢 | [Classer des scénarios dans le Top 10 de l'OWASP](01-classer-owasp.md) | 01 — Anatomie d'une vulnérabilité | [corrigé](corrige/01-classer-owasp.md) |
| 🟡 | [Repérer une injection SQL dans un extrait Python/SQL](02-trouver-injection.md) | 02 — Les injections | [corrigé](corrige/02-trouver-injection.md) |
| 🔵 | [Lister les vulnérabilités et leur gravité sur un extrait de code](03-revue-de-code.md) | 03-05 — Failles web et configuration | [corrigé](corrige/03-revue-de-code.md) |
| 🔴 | [Modéliser les vulnérabilités d'un pipeline de données](04-defi-menace-data.md) | 06 — Lire du code vulnérable | [corrigé](corrige/04-defi-menace-data.md) |
| 🟡 | [Lire un advisory et rédiger un avis d'exposition](05-lire-advisory.md) | 09 — La vie d'une CVE | [corrigé](corrige/05-lire-advisory.md) |
| 🟢 | [Reconstituer les étapes d'une attaque](07-reconstituer-attaque.md) | 08 — La kill chain | [corrigé](corrige/07-reconstituer-attaque.md) |
| 🟡 | [Reconstituer la course patch / exploit](09-timeline-cve.md) | 09 — Failles connues et course | [corrigé](corrige/09-timeline-cve.md) |
| 🔵 | [Exploiter, documenter, corriger une faille connue](10-td-exploiter-et-corriger.md) | 10 — Attaques appliquées | [corrigé](corrige/10-td-exploiter-et-corriger.md) |
| 🔵 | [Écrire une analyse menace / prévention](12-analyse-menace-prevention.md) | 12 — Prévention et détection | [corrigé](corrige/12-analyse-menace-prevention.md) |
| 🔴 | [Modéliser la kill chain d'une fuite de données](13-defi-kill-chain-data.md) | 13 — Tests d'intrusion | [corrigé](corrige/13-defi-kill-chain-data.md) |

Les niveaux suivent la progression 🟢 découverte · 🟡 application · 🔵 intégration ·
🔴 défi facultatif. Un 🔴 ne bloque jamais la progression.
