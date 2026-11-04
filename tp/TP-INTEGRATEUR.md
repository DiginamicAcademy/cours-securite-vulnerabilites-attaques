# TP intégrateur — Sécuriser une application de bout en bout

> **Nature :** TP intégrateur noté · **Durée :** 3 h · **Notion :** le cours entier, appliqué de bout en bout à une application réelle

> ⚖️ **Cadre légal.** Tout ce TP se pratique sur **vos propres conteneurs** (lab Docker du cours) et sur des données fictives. Accéder ou se maintenir sans droit dans un système d'autrui est un délit (articles 323-1 et suivants du Code pénal), même « pour tester ». Les tests d'intrusion réels se font sous contrat, dans un périmètre défini.

---

## En bref

| Rubrique | Détail |
| --- | --- |
| **Nature** | TP intégrateur **noté** — un seul pour ce cours de deux jours |
| **Durée** | 3 h, en fin de J3 |
| **Travail** | individuel |
| **Livrable final unique** | le **Dossier d'analyse et de sécurisation** — un seul document |
| **Notation** | sur 20, grille ci-dessous |

Ce TP est **noté** : contrairement aux exercices de ce dossier, il n'a pas de corrigé publié ici.

---

## Contexte

**Bocage** est une PME (petite et moyenne entreprise) de vente en ligne de matériel de randonnée. Son application tourne depuis deux ans et a grandi par ajouts successifs. Elle se compose de cinq pièces :

1. **La boutique web** — écrite en Python avec FastAPI, avec des pages HTML : catalogue, panier, comptes clients, commandes.
2. **L'API partenaire** — un point d'entrée JSON qui transmet les commandes au fournisseur.
3. **La base de commandes** — en SQL : comptes, commandes, moyens de paiement.
4. **La chaîne de reporting** — chaque nuit, des exports CSV sont déposés dans un stockage objet, un planificateur déclenche les traitements, et un tableau de bord expose les ventes aux responsables de magasin.
5. **Les dépendances** — un fichier de dépendances figé il y a deux ans, jamais remis à jour.

Autour de l'application : deux développeurs, un administrateur système, les clients de la boutique, et le partenaire fournisseur.

La direction vient de lancer un appel clair : *« Dites-nous par où cette application peut être atteinte, ce que cela coûterait, comment l'en empêcher — et comment le voir si cela arrive quand même. »* C'est exactement votre mission.

---

## Objectifs

À l'issue de ce TP, vous êtes capable de :

- **cartographier** les vulnérabilités d'une application et les rattacher aux familles du Top 10 de l'**OWASP** (Open Web Application Security Project) ;
- **évaluer** la gravité d'un constat dans son contexte, et **ordonner** les corrections ;
- **modéliser** une attaque de bout en bout — la kill chain — et **nommer** le profil d'attaquant ;
- **exploiter** une faille sur un environnement de test et **expliquer** son mécanisme ;
- **corriger** à la cause racine et **vérifier** la correction par un test de non-régression ;
- **concevoir** la détection : signaux, règles d'alerte, premiers gestes de réponse ;
- **rédiger** un dossier technique défendable, dans le cadre légal.

---

## Étapes

Le TP se déroule de bout en bout : chaque étape s'appuie sur la précédente, et toutes alimentent le même livrable.

### 1. Cadrer — 20 min

- Délimitez le périmètre : composants, acteurs, données en jeu.
- Dressez la **surface d'attaque** : les points d'entrée, ce qui est exposé, à qui.
- Rappelez le cadre légal qui autorise vos tests.

*Attendu — partie 1 du dossier : une fiche de cadrage d'une page.*

### 2. Cartographier les vulnérabilités — 50 min

- Passez le code, la configuration et les dépendances en revue. Les dépendances n'ont pas été vérifiées depuis deux ans : c'est à vous de voir ce qu'elles contiennent.
- Pour chaque constat : l'endroit concerné, la famille OWASP, la **cause racine**, la gravité dans ce contexte.
- Ordonnez les corrections, et justifiez les deux premières.

*Attendu — partie 2 : le tableau des constats, priorisé.*

### 3. Modéliser l'attaque — 30 min

- Choisissez un profil d'attaquant et justifiez-le.
- Retracez une **kill chain** de bout en bout, de la reconnaissance à l'exfiltration : à chaque étape, ce que l'attaquant fait, le composant visé, ce qu'il obtient pour l'étape suivante.
- Repérez les **points de confiance** de la chaîne.

*Attendu — partie 3 : la kill chain commentée et le profil d'attaquant.*

### 4. Prouver — 30 min

- Sur le lab local **uniquement**, exploitez **au moins une** faille de votre cartographie.
- Enregistrez la requête dans un script réutilisable, et expliquez le mécanisme en une dizaine de lignes.

*Attendu — partie 4 : le script, l'explication du mécanisme, la preuve constatée.*

### 5. Prévenir — 30 min

- Corrigez sur les deux plans : la cause racine dans le code ou la configuration, **et** la dépendance obsolète.
- Reconstruisez le service, relancez votre script : il ne doit plus fonctionner. Notez ce que fait alors la page.
- Écrivez le **test de non-régression** qui empêchera la faille de revenir.

*Attendu — partie 5 : les corrections, la constatation, le test.*

### 6. Détecter et répondre — 20 min

- Pour les scénarios retenus, formulez l'événement qui les rendrait visibles, comme une règle : « quand X arrive, alerter ».
- Donnez les premiers gestes de réponse, dans l'ordre : conserver, isoler, qualifier, informer, corriger.

*Attendu — partie 6 : la table de détection et les premiers gestes.*

---

## Livrable final unique

**Le Dossier d'analyse et de sécurisation** : un seul document, structuré en six parties — une par étape — suivi d'annexes numérotées (scripts, captures, extraits de journaux). Tout le reste — brouillons, notes, captures intermédiaires — reste hors du livrable.

Le dossier doit se lire sans vous : un lecteur qui n'a pas assisté au TP doit comprendre ce que vous avez trouvé, pourquoi c'est grave, ce que vous avez prouvé, et ce que vous proposez.

1. Cadrage
2. Cartographie des vulnérabilités
3. Modélisation de l'attaque
4. Preuve d'exploitation
5. Prévention
6. Détection et réponse
7. Annexes

---

## Grille d'évaluation

| Critère | Points |
| --- | --- |
| Cadrage : périmètre, acteurs, données, surface d'attaque | 2 |
| Cartographie : familles, causes racines, gravité justifiée, priorisation | 5 |
| Modélisation : kill chain complète, profil d'attaquant, points de confiance | 4 |
| Preuve : exploitation sur le lab et explication du mécanisme | 3 |
| Prévention : correction à la cause racine et test de non-régression | 3 |
| Détection et réponse : signaux, règles, premiers gestes | 2 |
| Dossier : clarté, cadre légal, limites honnêtes | 1 |
| **Total** | **20** |

---

## Durée

**3 h**, réparties comme indiqué en tête de chaque étape : 20 + 50 + 30 + 30 + 30 + 20 minutes. Le lab est supposé démarré avant le début du TP.

---

## Ce qui est fourni

- Le code source de l'application `bocage` et son lab Docker (`lab/` du cours) : boutique, API, base de commandes, chaîne de reporting.
- Le fichier de dépendances de l'application.
- Des extraits de journaux des différents composants.

Rien d'autre à apporter : le cours suffit.

---

## ✅ Auto-évaluation

Avant de rendre votre dossier, vérifiez que :

- [ ] Mon dossier contient les six parties, dans l'ordre, et se lit sans moi.
- [ ] Chaque constat nomme une famille OWASP, une cause racine et une gravité justifiée par l'exposition et par ce que l'entreprise y perd.
- [ ] Ma kill chain va de la reconnaissance à l'exfiltration sans sauter d'étape, et chaque étape dit ce que l'attaquant obtient pour la suivante.
- [ ] J'ai exploité au moins une faille sur le lab, avec un script réutilisable, et j'explique le mécanisme.
- [ ] Mes corrections portent sur la cause racine, et mon test de non-régression le vérifie.
- [ ] Mes signaux de détection sont formulés comme des règles : « quand X arrive, alerter ».
- [ ] Mes premiers gestes de réponse suivent un ordre, et commencent par la conservation des traces.
- [ ] Je cite le cadre légal, et je dis honnêtement ce que mon analyse ne couvre pas.

---

## Travaux préparatoires

Ces travaux du dossier `tp/` préparent le TP — ils ne s'y substituent pas :

| Travail | Étape du TP qu'il prépare |
| --- | --- |
| [Lister les vulnérabilités et leur gravité sur un extrait de code](03-revue-de-code.md) | Étape 2 — cartographier |
| [Modéliser les vulnérabilités d'un pipeline de données](04-defi-menace-data.md) | Étape 2 — les points de confiance |
| [Exploiter, documenter, corriger une faille connue](10-td-exploiter-et-corriger.md) | Étapes 4 et 5 — prouver et corriger |
| [Écrire une analyse menace / prévention](12-analyse-menace-prevention.md) | Étape 6 — détecter et répondre |
| [Modéliser la kill chain d'une fuite de données](13-defi-kill-chain-data.md) | Étape 3 — modéliser l'attaque |

---

## Suivi de production

| Phase | État |
| --- | --- |
| Conception (ce document) | ✅ fait le 04/11/2026 |
| Rédaction de l'énoncé apprenant | ⬜ à faire |
| Corrigé (hors dépôt public) | ⬜ à faire |
| Relecture | ⬜ à faire |

**Journal** — 04/11/2026 : v1, conception initiale.
