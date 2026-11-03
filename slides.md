---
marp: true
theme: default
RNCPBloc: [RNCP40573BC03]
style: |
  section {background-color: #121114}
  h1,h2,h3 {color: #8393f0}
  p,ul,li,td,th {color: #ccc}
  section table {font-size: 0.85em}
  section table td {background-color: #121114}
  section table th {background-color: #272133; font-weight: bolder}
  pre {background-color: #17151a; color: #ccc}
  blockquote {border: 2px green solid; border-left-width: 15px; padding: 0.5em 15px;font-style: italic }
paginate: true
header: "Les vulnérabilités et techniques d'attaques"
footer: "![height:20px](https://raw.githubusercontent.com/DiginamicInternal/PublicAssets/refs/heads/main/Logo-diginamic-color-blk.png)"
---

# Les vulnérabilités et techniques d'attaques

**J2 — Vulnérabilités applicatives · J3 — Techniques d'attaque**

**Cours d'autoformation · RNCP40573 — brique transversale de sécurité, adossée au bloc BC03**

<!-- Diapositive d'ouverture. Ce commentaire est une note de présentateur : il
     n'apparaît pas à la projection, mais reste visible dans le PPTX. -->

---

## Objectifs pédagogiques

À la fin de ces deux journées, vous saurez :

- **Retracer** les quatre étapes de la vie d'une faille
- **Distinguer** une cause racine d'un symptôme
- **Identifier** les familles du Top 10 de l'**OWASP** (Open Web Application Security Project) dans un scénario concret
- **Relier** la gravité d'une faille à son exposition et à son contexte métier
- **Décrire** les étapes d'une attaque, de la reconnaissance à l'exfiltration
- **Relier** chaque mode opératoire à la mesure qui le coupe
- **Lire** un avis de faille connue et en tirer un avis d'exposition
- **Repérer** les signes d'une vulnérabilité dans un extrait de code
- **Justifier** les priorités de défense d'une application

---

## Le parcours : deux journées

| Journée | Thème | L'idée à retenir |
|---|---|---|
| **J2** | Vulnérabilités applicatives | avant de corriger une faille, il faut la **voir** |
| **J3** | Techniques d'attaque | suivre le mode opératoire pour savoir **où couper la chaîne** |

> J2 pose le langage des vulnérabilités ; leur correction pratique vient plus
> tard, dans les journées défensives du parcours.

---

## Le rythme d'une journée

| Étape | Activité |
|---|---|
| 1 | **Découverte** — lire un chapitre |
| 2 | **Mise en pratique** — faire l'exercice tout de suite |
| 3 | **Auto-correction** — confronter ses réponses au corrigé |
| 4 | **Checkpoint** — cocher ses acquis en fin de journée |

> Chaque notion entre par une histoire, se manipule dans un atelier de 5 à 10
> minutes, puis se pratique dans l'exercice du bloc.

---

## Le fil rouge : DIGICHEESE

Tout au long du parcours, une même entreprise : **DIGICHEESE** (la fromagerie du
fil rouge).

- une boutique en ligne, une recherche produits, des commentaires clients
- un espace d'administration, une base de données, des factures
- une petite équipe, peu de temps, et les habitudes du développement pressé

> Les failles que nous cherchons sont les siennes. Les correctifs que nous posons
> sont les siens aussi.

---

## J2 — Vulnérabilités applicatives

**Voir la faille.**

Avant de corriger, il faut savoir ce qu'on regarde : où est la faille, pourquoi
elle existe, et ce qu'elle coûte.

---

## Une ligne mal écrite, et tout l'édifice tombe

En 2011, le service de jeux en ligne de Sony est mis hors service pendant
plusieurs semaines : **77 millions de comptes** exposés.

L'origine, selon les rapports de l'époque : une injection **SQL** (Structured
Query Language) — une requête construite en collant ce que l'utilisateur tape,
sans jamais le vérifier.

<!-- Récit d'accroche du chapitre 1. -->

---

## La vie d'une faille : quatre étapes

| Étape | La question qu'elle pose | Dans l'histoire de Sony |
|---|---|---|
| **Cause racine** | pourquoi la faille existe-t-elle ? | une requête écrite par concaténation |
| **Exposition** | où est-elle atteignable, et par qui ? | une page de recherche ouverte à tous |
| **Exploitation** | comment l'attaquant s'en sert-il ? | il fait réécrire la requête avec ses propres critères |
| **Impact** | qu'est-ce que l'organisation y perd ? | des comptes, de la confiance, des semaines d'activité |

---

## Où couper la chaîne

Ces quatre étapes se coupent à plusieurs endroits :

- une **requête paramétrée** supprime la cause racine
- un **filtre en amont** réduit l'exposition
- une **surveillance** réduit la durée de l'exploitation
- une **procédure de crise** réduit l'impact

> Aucune de ces barrières ne remplace les autres : c'est leur succession qui fait
> la sécurité.

---

## Atelier rapide (5 min) — le formulaire sans captcha

Un formulaire d'inscription n'a pas de captcha : un script peut y créer des
comptes par milliers.

Retrouvez les quatre étapes de cette faille :

1. cause racine
2. exposition
3. exploitation
4. impact

Une ligne par étape suffit.

---

## Cause racine ou symptôme ?

La recherche produits de DIGICHEESE renvoie tout le catalogue dès qu'on y tape
une apostrophe. Trois correctifs sont sur la table :

- bloquer les requêtes qui contiennent une apostrophe ;
- afficher une page d'erreur neutre, sans détail technique ;
- écrire la requête avec des paramètres, au lieu de coller les chaînes.

Le premier traite un **symptôme** : il cache un cas précis, et il en bloquera
d'autres — dont des clients qui cherchent un cœur de Comté.

Le troisième traite la **cause racine** : la requête ne mélange plus les données
et les instructions.

---

## La question à se poser

> Si je corrige cela, la faille peut-elle encore être exploitée autrement ?

- **Oui** → vous traitez un symptôme.
- **Non** → vous traitez la cause racine.

C'est la question qui sépare un correctif utile d'un correctif rassurant.

---

## Toutes les failles ne se valent pas

Une faille critique sur un poste isolé peut être moins grave qu'une faille
moyenne sur la page de paiement d'un site fréquenté.

Deux notions font la différence :

- la **surface d'attaque** : ce qui est joignable de l'extérieur, et par qui ;
- le **contexte métier** : ce qui est derrière — données de paiement, dossiers,
  comptes administrateurs.

---

## Noter une faille : le CVSS

L'industrie s'appuie sur des grilles de notation. La plus répandue est le
**CVSS** (Common Vulnerability Scoring System) : un score qui combine ce que
l'attaquant doit faire pour réussir et ce que l'organisation y perd.

> Un score est un **point de départ**, pas une décision : un score élevé sur un
> système sans valeur se traite après un score moyen sur la caisse.

---

## Le Top 10 de l'OWASP : une carte, pas une liste

| Code | Famille | Ce qu'elle recouvre |
|---|---|---|
| **A01** | Broken Access Control | faire ce qu'on n'a pas le droit de faire |
| **A02** | Cryptographic Failures | des données mal protégées |
| **A03** | Injection | confondre les données et les instructions |
| **A04** | Insecure Design | un choix de conception dangereux |
| **A05** | Security Misconfiguration | réglages par défaut, debug laissé actif |
| **A06** | Vulnerable and Outdated Components | une dépendance connue pour être fragile |
| **A07** | Identification and Authentication Failures | comptes et mots de passe mal protégés |
| **A08** | Software and Data Integrity Failures | des mises à jour jamais vérifiées |
| **A09** | Security Logging and Monitoring Failures | ne pas voir l'attaque arriver |
| **A10** | SSRF (Server-Side Request Forgery) | faire demander au serveur ce qu'il ne devrait pas |

> Ce n'est pas une liste à apprendre par cœur : c'est une carte pour se parler.

---

## TP — Classer des scénarios dans le Top 10

La fromagerie DIGICHEESE remonte six observations :

1. la recherche produits renvoie tout le catalogue avec `' OR '1'='1`
2. un commentaire client exécute une balise de script dans le navigateur
3. `/admin/users` est accessible à un simple client
4. `/debug/config` affiche les identifiants de la base de données
5. les factures sont produites par une bibliothèque vieille de plusieurs années
6. le formulaire de connexion n'a aucune limite de tentatives

> 🟢 25 min — rattacher chaque observation à une famille, retracer l'anatomie de
> l'une d'elles, dire quel correctif traite la cause racine.

---

## Les injections : confondre données et instructions

`' OR '1'='1` — sept caractères, et la base de données renvoie tous les
enregistrements.

Ce n'est pas de la magie, c'est de la grammaire : la requête a été **réécrite**
par l'attaquant.

Un même mécanisme, trois syntaxes : injection SQL, injection NoSQL, injection de
commandes.

> La cause racine est toujours la même : des données traitées comme des
> instructions.

---

## Les failles web : XSS, CSRF, contrôle d'accès

- **XSS** (Cross-Site Scripting) — la confusion entre ce que l'utilisateur voit
  et ce que le navigateur exécute. Un commentaire devient un cheval de Troie.
- **CSRF** (Cross-Site Request Forgery) — faire faire une action à l'utilisateur
  sans qu'il le sache.
- **Broken Access Control** — accéder à `/admin/users` sans être administrateur.

> Un même fil : trop de confiance dans ce qui vient de l'utilisateur.

---

## Configuration et dépendances

**Security misconfiguration** — le mode debug laissé actif en production, des mots
de passe par défaut, des pages inutiles : `/debug/config` affiche les identifiants
de la base.

**Vulnerable components** — le code qu'on écrit n'est qu'une fraction de ce qui
tourne. Une bibliothèque tierce est une surface d'attaque à part entière.

---

## Anatomie d'une faille connue

Une **CVE** (Common Vulnerabilities and Exposures) est l'identifiant public d'une
faille connue. Un avis de sécurité l'accompagne.

| Élément | Ce qu'il dit |
|---|---|
| identifiant | quelle faille, dans quel produit |
| score CVSS | à quel point elle est grave |
| versions affectées | ce qui doit être corrigé |
| correctif | la version qui ferme la faille |

> Le cycle : découverte → publication → correctif → exploitation. La course se
> joue en heures, parfois en minutes.

---

## Les autres familles de failles

Certaines failles sortent du Top 10, mais le raisonnement est le même : **trop de
confiance, pas assez de validation**.

- **SSRF** — faire demander au serveur ce qu'il ne devrait pas
- les **entités externes XML** (eXtensible Markup Language) — un document
  malveillant qui prend le contrôle du parseur ; c'est ce qu'on appelle une
  faille **XXE** (XML External Entity)
- **désérialisation** — reconstruire un objet depuis des données non fiables
- **prototype pollution** — pour les applications JavaScript

---

## Lire du code vulnérable

Un relecteur expérimenté passe 20 minutes sur un diff de 40 lignes et trouve une
injection. Comment ? Il ne lit pas le code : il **cherche les confusions**.

La méthode en trois passes :

1. **les entrées** — d'où viennent les données ?
2. **les sorties** — où vont-elles ?
3. **les appels système** — que fait le programme avec ?

> Les marqueurs à chercher : une requête construite par concaténation,
> `format()`, `eval`, un fichier ouvert sans validation.

---

## J3 — Techniques d'attaque

**Suivre l'attaquant.**

Ce n'est pas parce qu'on ignore l'attaquant qu'il nous ignore : on suit une
compromission de bout en bout, puis on revient sur chaque étape pour dire où la
défense pouvait agir.

---

## Qui attaque ?

Un ancien employé mécontent, un adolescent qui teste un outil trouvé en ligne,
une cellule financée par un État : trois attaquants, trois budgets, trois
objectifs.

| Profil | Motivation | Moyens |
|---|---|---|
| opportuniste | argent, gloire | outils trouvés en ligne, peu de temps |
| initié | revanche, gain | connaissance du système d'information |
| organisé | espionnage, rançon | temps, budget, persévérance |

> Connaître l'adversaire aide à choisir la bonne défense.

---

## La kill chain

En 2013, la compromission de Target : un fournisseur de climatisation compromis →
accès au réseau → vol de 40 millions de cartes bancaires.

La chaîne d'une compromission :

**reconnaissance → accès initial → escalade → mouvement latéral → exfiltration**

> Plusieurs échecs doivent s'enchaîner pour qu'une catastrophe ait lieu — donc
> plusieurs occasions de l'empêcher.

---

## Où couper la kill chain

| Étape | La mesure qui la coupe |
|---|---|
| reconnaissance | réduire la surface exposée |
| accès initial | corriger les failles connues, filtrer les entrées |
| escalade | moindres privilèges, comptes séparés |
| mouvement latéral | segmentation, surveillance des flux |
| exfiltration | contrôle des sorties, détection des volumes anormaux |

---

## La course correctif / exploitation

**PHP** (PHP: Hypertext Preprocessor), octobre 2026 — l'avis du **CERT**
(Computer Emergency Response Team) de l'Union européenne, référence 2026-018 :

- le 20 octobre, deux failles d'exécution de code à distance non authentifiées
  sont annoncées (score CVSS 9.2)
- le 21 à 13:30 **UTC** (temps universel coordonné), le correctif sort
- dans la foulée, des preuves de concept circulent
- le 23, une **troisième** faille est annoncée

> De la publication à l'exploitation massive, il s'écoule parfois des heures —
> parfois des minutes.

---

## L'effet de l'IA sur la course

Les outils d'**IA** accélèrent la génération de preuves de concept : le décalage
entre le correctif et l'exploitation se réduit.

Ce qu'il impose au défenseur : la **veille**.

- suivre les avis de sécurité de ses dépendances
- savoir ce qui tourne réellement en production
- pouvoir corriger et redéployer vite

> Un correctif installé mais jamais redéployé ne corrige rien.

---

## Trois scénarios, un même raisonnement

| Scénario | Ce qui se passe | La mesure qui le coupe |
|---|---|---|
| injection SQL → vol de données | la requête est réécrite | requête paramétrée |
| XSS → vol de session | un commentaire devient du code | échappement systématique |
| API → base de données | les droits ne sont pas contrôlés | contrôle d'accès côté serveur |

> On relie les notions de J2 à des faits concrets : la chronologie, le point de
> rupture, le correctif.

---

## Les attaques d'authentification

2012, LinkedIn : 117 millions de mots de passe volés. Des années plus tard, des
comptes piratés — parce que les gens réutilisent leurs mots de passe.

- **credential stuffing** — rejouer les couples identifiant / mot de passe volés
  ailleurs
- **force brute** — essayer jusqu'à trouver
- **hameçonnage** — un faux site suffit à récolter un mot de passe valide
- **vol de session** — un cookie de session dérobé remplace le mot de passe

> L'authentification est le premier rempart ; ses faiblesses sont souvent
> humaines.

---

## Prévenir, détecter, répondre

Une attaque passe inaperçue pendant des mois en moyenne. La question n'est pas
« comment ne jamais être attaqué » mais « comment s'en rendre compte vite ».

| Temps | Ce qu'on fait |
|---|---|
| **prévenir** | corriger les causes racines, réduire la surface exposée |
| **détecter** | journaliser, surveiller, alerter |
| **répondre** | un plan d'incident, même sommaire |

> Vérifiez ce que vos journaux contiennent réellement : une attaque invisible à
> la lecture est une attaque qui recommencera.

---

## Les tests d'intrusion

Une entreprise commande un test d'intrusion « pour cocher une case », sans
périmètre ni suivi des recommandations. Un an plus tard, la même faille est
exploitée.

Un test d'intrusion n'est pas un exercice de hacking : c'est une **preuve** de
l'état de sécurité, avec un rapport.

- **périmètre et règles d'engagement** — ce qui est testé, quand, et jusqu'où
- **déroulé** — reconnaissance, exploitation, rapport
- **limites** — ce qui n'a pas été testé n'est pas protégé

---

## Le lab de pratique

Un environnement local, à vous, dans lequel les failles étudiées sont
volontairement présentes.

```bash
cd lab && docker compose up --build -d
```

> **Cadre éthique** : ces manipulations se font **uniquement** sur le lab local.
> Les étudier sert à se défendre — jamais à tester un système tiers sans
> autorisation écrite.

---

## Points clés

- Une faille suit quatre étapes : **cause racine, exposition, exploitation,
  impact** — et la chaîne se coupe à chacune d'elles.
- Corriger un symptôme donne l'impression d'avancer : seule la cause racine rend
  l'attaque impossible à rejouer.
- La gravité dépend de **l'exposition et du contexte métier**, pas de la seule
  complexité technique.
- Une attaque est une **chaîne** : plusieurs échecs doivent s'enchaîner, donc
  plusieurs occasions d'agir.
- La course correctif / exploitation se joue en heures : la **veille** fait
  partie du travail de développement.

---

## Des questions ?

Le dépôt reste la référence : chapitres, travaux pratiques et corrigés.

Pour toute question [adressez-vous au service pédagogie](mailto:pedagogie@diginamic.fr)
