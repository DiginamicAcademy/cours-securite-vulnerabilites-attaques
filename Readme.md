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

La plupart des incidents de sécurité commencent par une faille laissée passer :
une donnée jamais validée, une dépendance jamais mise à jour, un réglage par
défaut oublié. En parler avec les quatre étapes d'une faille — cause racine,
exposition, exploitation, impact — donne au développeur, à l'administrateur et au
responsable métier le langage commun qui permet de décider quoi corriger en
premier.

Ce cours couvre les deux journées centrales du volet sécurité : les
vulnérabilités applicatives d'abord, avec les familles du Top 10 de l'**OWASP**
(Open Web Application Security Project) et le cycle de vie des failles
connues ; les techniques d'attaque ensuite, avec la chaîne d'une compromission,
les modes opératoires réels et la course entre correctif et exploitation.

Dans une spécialisation Big Data et intelligence artificielle (IA), l'enjeu est
direct : un composant compromis expose toute la chaîne de traitement, des sources
jusqu'aux modèles. La pratique se fait dans un cadre éthique strict — étudier les
modes opératoires pour se défendre, jamais pour attaquer des systèmes tiers — sur
un environnement de laboratoire local, avec le fil rouge **DIGICHEESE** (la
fromagerie du fil rouge).

## À l'issue de ce cours

Vous serez capable de :

- **Identifier** les familles de vulnérabilités du Top 10 de l'OWASP (version 2021) et les situer dans un système d'information.
- **Distinguer** une vulnérabilité, une faille exploitable et un risque métier.
- **Relier** chaque faille à sa cause racine : validation, confiance, configuration, dépendances.
- **Reconnaître** les signes d'une vulnérabilité dans un extrait de code ou de configuration.
- **Évaluer** la gravité d'une faille à partir de son exposition, de son contexte métier et d'un score CVSS (Common Vulnerability Scoring System).
- **Décrire** les étapes d'une attaque : reconnaissance, exploitation, pivot, exfiltration.
- **Retracer** la course entre correctif et exploitation d'une faille connue, identifiée par une **CVE** (Common Vulnerabilities and Exposures).
- **Relier** chaque mode opératoire d'attaque aux mesures de prévention et de détection correspondantes.
- **Analyser** un scénario d'attaque multi-étapes pour identifier les points de rupture.
- **Justifier** les priorités de défense d'une application selon les attaques les plus probables.

## Sommaire

| # | Chapitre | Durée de lecture |
| --- | --- | --- |
| 01 | [Anatomie d'une vulnérabilité — Vie d'une faille](01-anatomie-vulnerabilite.md) | 45 min |

---

## Aller plus loin

*Ces approfondissements sont facultatifs : ils ne sont jamais exigés pour
terminer la journée.*

- **[OWASP Top 10:2021](https://owasp.org/Top10/2021/)** — les dix familles de failles web étudiées dans le cours, chacune avec ses causes, ses exemples et sa prévention.
- **[ATT (Adversarial Tactics, Techniques) & CK (Common Knowledge)](https://attack.mitre.org/)** — la base de référence des modes opératoires d'attaquants, décrite à partir d'incidents réels.
- **[NVD (National Vulnerability Database)](https://nvd.nist.gov/)** — la base américaine des failles connues : chaque CVE y est décrite, notée en CVSS et référencée.
- **[Guide d'hygiène informatique de l'ANSSI (Agence nationale de la sécurité des systèmes d'information)](https://messervices.cyber.gouv.fr/guides/guide-dhygiene-informatique)** — les 42 mesures d'hygiène à mettre en œuvre sur un système réel.
- **[CERT (Computer Emergency Response Team) de l'Union européenne](https://cert.europa.eu/)** — le service de cybersécurité des institutions de l'Union européenne (CERT-EU), qui publie des alertes de sécurité lors de la découverte de failles massivement exploitées.
