<p align="center">
  <picture>
    <source media="(prefers-color-scheme: dark)" srcset="assets/Logo-diginamic-color-blk.png">
    <img src="assets/Logo-diginamic-color.svg" alt="Diginamic Formation" width="358">
  </picture>
</p>

# Les vulnérabilités et techniques d'attaques

> Cours d'**autoformation** · **RNCP40573** Expert en informatique et systèmes d'information · brique transversale de **sécurité**, adossée au bloc **BC03** (compétence C25)
> 2 journée(s) · supports **rédigés par IA**, relus et validés.

---

## Pourquoi ce cours

La fromagerie **DIGICHEESE** (la fromagerie du fil rouge) vient de mettre sa
boutique en ligne. Sa recherche produits tient en une ligne de Python. Ça marche,
c'est même rapide. Et c'est exactement ça, le problème : les failles les plus
graves tiennent souvent dans les lignes les plus simples à écrire.

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

Les chapitres se lisent dans l'ordre ; chaque notion est pratiquée juste après.

1. [Anatomie d'une vulnérabilité](01-anatomie-vulnerabilite.md) — la vie d'une
   faille : cause racine, exposition, exploitation, impact.

**Travaux pratiques**

- [Classer des scénarios dans le Top 10 de l'OWASP](tp/01-classer-owasp.md) —
  🟢 25 min, à faire juste après le chapitre 1.
- [Corrigé du travail pratique](tp/corrige/01-classer-owasp.md) — à ouvrir après
  avoir fait l'exercice.

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
