<p align="center">
  <picture>
    <source media="(prefers-color-scheme: dark)" srcset="assets/Logo-diginamic-color-blk.png">
    <img src="assets/Logo-diginamic-color.svg" alt="Diginamic Formation" width="358">
  </picture>
</p>

# Les vulnérabilités et techniques d'attaques

> Cours d'**autoformation** · **RNCP40573** Expert en informatique et systèmes d'information · brique transversale de **sécurité**, adossée au bloc **BC03** (compétence C25)
> 2 journées · supports **rédigés par IA**, relus et validés.

---

## Pourquoi ce cours

La fromagerie **DIGICHEESE** (l'entreprise d'étude du parcours) vient de mettre sa
boutique en ligne. Sa recherche produits tient en une ligne de Python. Ça marche,
c'est même rapide. Et c'est exactement ça, le problème : les failles les plus
graves tiennent souvent dans les lignes les plus simples à écrire.

Les ateliers se déroulent dans le lab Docker partagé du parcours ([DiginamicAcademy/lab-digicheese](https://github.com/DiginamicAcademy/lab-digicheese)).

Ce cours couvre les **journées 2 et 3** du parcours de sécurité applicative :

- **J2 — Vulnérabilités applicatives** : apprendre à *voir* une faille. Cause
  racine, exposition, exploitation, impact — puis les grandes familles du Top 10
  de l'**OWASP** (Open Web Application Security Project).
- **J3 — Techniques d'attaque** : suivre un attaquant de bout en bout. Profils,
  chaîne de compromission, course entre correctif et exploitation, puis les
  mesures qui coupent la chaîne.

L'enjeu n'est pas de devenir un attaquant. C'est de savoir, devant un extrait de
code ou une configuration, dire **où est la faille, pourquoi elle existe, et ce
qu'elle coûterait** — pour décider quoi corriger en premier.

## À l'issue de ce cours

À l'issue de ce cours, vous serez capable de :

- **Retracer** les quatre étapes de la vie d'une faille : cause racine, exposition, exploitation, impact.
- **Distinguer** une cause racine d'un symptôme, et dire si un correctif ferme vraiment la faille.
- **Identifier** les dix familles du Top 10 de l'OWASP dans un scénario concret.
- **Relier** la gravité d'une faille à son exposition et à son contexte métier.
- **Décrire** les étapes d'une attaque : reconnaissance, accès initial, escalade, mouvement, exfiltration.
- **Relier** chaque mode opératoire à la mesure de prévention ou de détection qui le coupe.
- **Lire** un avis de faille connue, identifiée par une **CVE** (Common Vulnerabilities and Exposures), et en tirer un avis d'exposition pour un système d'information.
- **Repérer** les signes d'une vulnérabilité dans un extrait de code ou de configuration.
- **Justifier** les priorités de défense d'une application selon les attaques les plus probables.

## Sommaire

Les chapitres se lisent dans l'ordre ; chaque notion est pratiquée juste après,
par l'exercice intégré au chapitre (sa correction se déplie sous l'énoncé).

| # | Chapitre | Durée de lecture |
| --- | --- | --- |
| 01 | [Anatomie d'une vulnérabilité — Vie d'une faille](01-anatomie-vulnerabilite.md) | 30 min |
| 02 | [Les injections — quand les données deviennent des instructions](02-owasp-injections.md) | 40 min |
| 03 | [Les failles web — quand le navigateur se retourne contre vous](03-owasp-web.md) | 40 min |
| 04 | [Configuration, dépendances et failles connues](04-owasp-config-deps.md) | 35 min |
| 05 | [Autres familles de failles — le même raisonnement, d'autres portes](05-autres-familles.md) | 25 min |
| 06 | [Lire du code vulnérable — la méthode des trois passes](06-lecture-de-code-vulnerable.md) | 30 min |
| 07 | [Modèles d'attaquant — Qui attaque, et avec quoi](07-modeles-attaquant.md) | 35 min |
| 08 | [La kill chain — Une attaque en sept étapes](08-kill-chain.md) | 35 min |
| 09 | [Failles connues et course contre la montre — La vie d'une CVE](09-cve-et-course.md) | 40 min |
| 10 | [Attaques appliquées — Trois scénarios, un même raisonnement](10-attaques-appliquees.md) | 40 min |
| 11 | [Attaques d'authentification — Le premier rempart, et ses failles](11-attaques-authentification.md) | 30 min |
| 12 | [Prévention et détection — Prévenir, détecter, répondre](12-prevention-et-detection.md) | 30 min |
| 13 | [Tests d'intrusion — Une preuve, pas un exercice de hacking](13-tests-intrusion.md) | 25 min |
| A | [Glossaire et ressources](A-Glossaire-et-ressources.md) | — |

**Travaux pratiques**

| | Sujet | Durée |
| --- | --- | --- |
| 🟢 | Classer des scénarios dans le Top 10 de l'OWASP — dans le chapitre [01 — Anatomie d'une vulnérabilité](01-anatomie-vulnerabilite.md), Exercice 1 | 25 min |
| 🟡 | Repérer une injection SQL dans un extrait Python/SQL — dans le chapitre [02 — Les injections](02-owasp-injections.md), Exercice 1 | 40 min |
| 🔵 | Lister les vulnérabilités et leur gravité sur un extrait de code — dans le chapitre [04 — Configuration, dépendances et failles connues](04-owasp-config-deps.md), Exercice 2 | 55 min |
| 🔴 | Modéliser les vulnérabilités d'un pipeline de données — dans le chapitre [05 — Autres familles de failles](05-autres-familles.md), Exercice bonus | libre |
| 🟡 | Lire un advisory et rédiger un avis d'exposition — dans le chapitre [04 — Configuration, dépendances et failles connues](04-owasp-config-deps.md), Exercice 1 | 35 min |
| 🟢 | Reconstituer les étapes d'une attaque — dans le chapitre [07 — Modèles d'attaquant](07-modeles-attaquant.md), Exercice 1 | 25 min |
| 🟡 | Reconstituer la course patch / exploit — dans le chapitre [09 — Failles connues et course contre la montre](09-cve-et-course.md), Exercice 1 | 40 min |
| 🔵 | Exploiter, documenter, corriger une faille connue — dans le chapitre [09 — Failles connues et course contre la montre](09-cve-et-course.md), Exercice 2 | 70 min |
| 🔵 | Écrire une analyse menace / prévention — dans le chapitre [12 — Prévention et détection](12-prevention-et-detection.md), Exercice 1 | 55 min |
| 🔴 | Modéliser la kill chain d'une fuite de données — dans le chapitre [08 — La kill chain](08-kill-chain.md), Exercice bonus | libre |
| 🛠️ | [TP intégrateur — Dossier d'analyse et de sécurisation](tp/TP-INTEGRATEUR.md) · **facultatif, noté s'il est rendu** | 3 h 10 |

---

## ✅ Checkpoints de fin de journée

À cocher avant de passer à la journée suivante. En cas de doute : ne pas
avancer, revenir au concept concerné, refaire l'exercice.

### Jour 2 — chapitres 01 à 06

- [ ] J'ai lu les chapitres 01 à 06 et les points clés sont clairs.
- [ ] J'ai fait au moins les exercices 🟢 et 🟡 (partie principale), et vérifié mes réponses avec le bloc « Voir la correction » placé sous chaque énoncé, dans le chapitre. Les extensions « Pour aller plus loin » sont facultatives.
- [ ] J'ai répondu au QCM formatif, lorsqu'il est disponible, et identifié mes notions à revoir.
- [ ] Je sais expliquer, avec mes mots, les 2-3 idées principales de la journée.
- [ ] Je peux appliquer ces idées à un cas simple (un extrait de code, une config, un flux).

### Jour 3 — chapitres 07 à 13

- [ ] J'ai lu les chapitres 07 à 13 et les points clés sont clairs.
- [ ] J'ai fait au moins les exercices 🟢 et 🟡 (partie principale), et vérifié mes réponses avec le bloc « Voir la correction » placé sous chaque énoncé, dans le chapitre. Les extensions « Pour aller plus loin » sont facultatives.
- [ ] J'ai répondu au QCM formatif, lorsqu'il est disponible, et identifié mes notions à revoir.
- [ ] Je sais expliquer, avec mes mots, les 2-3 idées principales de la journée.
- [ ] Je peux appliquer ces idées à un cas simple (un extrait de code, une config, un flux).

---

## Aller plus loin

*Ces ressources sont facultatives : elles ne sont jamais exigées pour terminer
le cours.*

- [Le Top 10 de l'OWASP](https://owasp.org/www-project-top-ten/) — la
  classification de référence des failles web, révisée régulièrement.
- [La base des vulnérabilités](https://nvd.nist.gov/) — **NVD** (National
  Vulnerability Database) : rechercher une **CVE**, lire son score **CVSS**
  (Common Vulnerability Scoring System).
- [Les guides d'hygiène](https://cyber.gouv.fr/) de l'**ANSSI** (Agence nationale
  de la sécurité des systèmes d'information) — ce que ce cours résume en quatre
  étapes, détaillé sur un système réel.
- [Les avis de sécurité](https://cert.europa.eu/) du **CERT** (Computer Emergency
  Response Team) de l'Union européenne — failles connues, gravité, correctifs.
