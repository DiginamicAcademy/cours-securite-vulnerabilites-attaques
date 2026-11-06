# TP intégrateur — Sécuriser une application de bout en bout

> **Nature :** TP intégrateur **facultatif**, noté au bulletin s'il est rendu · **Durée :** 3 h 10 · **Notion :** le cours entier, appliqué de bout en bout à une application réelle

> ⚖️ **Cadre légal.** Tout ce TP se pratique sur **vos propres conteneurs** (le lab DIGICHEESE du cours) et sur des données fictives. Accéder ou se maintenir sans droit dans un système d'autrui est un délit (articles 323-1 et suivants du Code pénal), même « pour tester ». Les tests d'intrusion réels se font sous contrat, dans un périmètre défini.

---

## En bref

| Rubrique | Détail |
| --- | --- |
| **Nature** | TP intégrateur **noté** — un seul pour ce cours de deux jours |
| **Durée** | 3 h 10, en fin de J3 |
| **Travail** | individuel |
| **Livrable final unique** | le **Dossier d'analyse et de sécurisation** — un seul document |
| **Notation** | TP noté sur 20 |

Ce TP est **noté** : contrairement aux exercices de ce dossier, il n'a pas de corrigé publié ici.

---

## Contexte

**DIGICHEESE** (la fromagerie du fil rouge) est une TPE (très petite entreprise) fromagère qui a mis sa boutique en ligne il y a deux ans. L'application a grandi par ajouts successifs et personne n'est revenu sur sa sécurité depuis. Elle tient en quatre services — exactement ceux du lab Docker que vous avez reçu :

| Service | URL | Ce qu'il fait |
| --- | --- | --- |
| `vulnerable` | http://localhost:8080 | **l'application qui tourne** : FastAPI, pages HTML et réponses JSON — recherche du catalogue, avis clients, connexion, page d'administration |
| `secure` | http://localhost:8081 | la **même application corrigée** — une référence à comparer, pas un modèle à recopier |
| `db` | localhost:3306 | MySQL, base `fromagerie` : comptes, catalogue, avis, commandes |
| `cve` | http://localhost:8082 | une petite app Flask annexe (recherche catalogue) dont une dépendance n'a jamais été revue |

Ce que l'application manipule : des comptes (identifiant, mot de passe, rôle), le catalogue et les stocks, les avis publiés par les clients, la table des commandes. Autour d'elle : la gérante, deux salariés, les clients, et le développeur qui a tout écrit, sans revue depuis la mise en ligne.

La gérante lance un appel clair : *« Dites-nous par où cette application peut être atteinte, ce que cela coûterait, comment l'en empêcher — et comment le voir si cela arrive quand même. »* C'est exactement votre mission.

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

- Délimitez le périmètre : les quatre services du lab, les acteurs, les données en jeu.
- Dressez la **surface d'attaque** : les points d'entrée un par un — `/products/search`, `/login`, `/comments`, `/admin/users`, `/debug/config`, la page de recherche du service `cve` — mais aussi ce qui est exposé en dehors de l'application, comme la base de données sur `localhost:3306` ou les secrets de configuration des conteneurs.
- Rappelez le cadre légal qui autorise vos tests.

*Attendu — partie 1 du dossier : une fiche de cadrage d'une page.*

### 2. Cartographier les vulnérabilités — 50 min

- Passez en revue le **code** (`vulnerable/app.py`), la **configuration** (`docker-compose.yml`, les `Dockerfile`) et les **dépendances** (`cve/requirements.txt` ; et, pour l'application principale, l'absence totale de version figée — c'est un constat à faire figurer au tableau, pas un document que vous avez raté).
- Pour chaque constat : l'endroit concerné (fichier et ligne, ou endpoint), la famille OWASP, la **cause racine**, la gravité dans ce contexte.
- Ordonnez les corrections, et justifiez les deux premières.

*Attendu — partie 2 : le tableau des constats, priorisé.*

### 3. Modéliser l'attaque — 30 min

- Choisissez un profil d'attaquant et justifiez-le.
- Retracez une **kill chain** de bout en bout, de la reconnaissance à l'exfiltration : à chaque étape, ce que l'attaquant fait, le composant visé, ce qu'il obtient pour l'étape suivante. Ne vous arrêtez pas à la première faille : ce que livre une simple injection dans la recherche du catalogue — la liste complète des comptes et de leurs empreintes, la fuite de la configuration — nourrit l'étape d'après.
- Repérez les **points de confiance** de la chaîne.

*Attendu — partie 3 : la kill chain commentée et le profil d'attaquant.*

### 4. Prouver — 30 min

- Sur le lab local **uniquement**, exploitez **au moins une** faille de votre cartographie.
- Enregistrez la requête dans un script réutilisable, et expliquez le mécanisme en une dizaine de lignes.

*Attendu — partie 4 : le script, l'explication du mécanisme, la preuve constatée.*

### 5. Prévenir — 30 min

- Corrigez sur les deux plans : la cause racine dans le code ou la configuration, **et** la dépendance obsolète du service `cve`.
- Reconstruisez le service (`docker compose up --build -d`), relancez votre script : il ne doit plus fonctionner. Notez ce que fait alors la page.
- Écrivez le **test de non-régression** qui empêchera la faille de revenir.
- Le service `secure` est là pour la comparaison : vous pouvez le citer, pas le recopier. Un dossier qui aligne du code corrigé sans nommer la cause racine n'est pas recevable.

*Attendu — partie 5 : les corrections, la constatation, le test.*

### 6. Détecter et répondre — 20 min

- Le lab ne fournit pas de journaux tout prêts : **vous produisez les vôtres** en déroulant votre kill chain (`docker compose logs vulnerable`, `docker compose logs db`), et c'est sur ces extraits que vous formulez la détection, sous forme de règle : « quand X arrive, alerter ».
- Donnez les premiers gestes de réponse, dans l'ordre : conserver, isoler, qualifier, informer, corriger.

*Attendu — partie 6 : la table de détection appuyée sur vos extraits de journaux, et les premiers gestes.*

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

## Durée

**3 h 10**, réparties comme indiqué en tête de chaque étape : 20 + 50 + 30 + 30 + 30 + 20 minutes. Le lab est supposé démarré avant le début du TP.

---

## Ce qui est fourni

- Le lab Docker **DIGICHEESE** partagé avec le parcours (dossier `lab/`) : `docker compose up --build -d` démarre les quatre services. Son `README.md` donne les endpoints, les failles mises en scène et les comptes de démonstration.
- Le code source de l'application dans ses deux versions — `vulnerable/` (ce qui tourne) et `secure/` (la référence corrigée) — et la base de démonstration `db/init.sql`.
- Le fichier de dépendances verrouillé du service `cve` (`cve/requirements.txt`).
- Les journaux : ceux que vous produisez vous-même avec `docker compose logs`, pas un fichier fourni.

Rien d'autre à apporter : le cours suffit.

---

## ✅ Auto-évaluation

Avant de rendre votre dossier, vérifiez que :

- [ ] Mon dossier contient les six parties, dans l'ordre, et se lit sans moi.
- [ ] Chaque constat nomme une famille OWASP, une cause racine et une gravité justifiée par l'exposition et par ce que l'entreprise y perd.
- [ ] Ma kill chain va de la reconnaissance à l'exfiltration sans sauter d'étape, et chaque étape dit ce que l'attaquant obtient pour la suivante.
- [ ] J'ai exploité au moins une faille sur le lab, avec un script réutilisable, et j'explique le mécanisme.
- [ ] Mes corrections portent sur la cause racine, et mon test de non-régression le vérifie.
- [ ] Mes signaux de détection sont formulés comme des règles : « quand X arrive, alerter », et s'appuient sur des journaux que j'ai produits.
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
