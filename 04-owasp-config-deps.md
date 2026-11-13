[← Chapitre précédent](03-owasp-web.md) · [Sommaire](Readme.md) · [Chapitre suivant →](05-autres-familles.md)

# Configuration, dépendances et failles connues

> Une bibliothèque Python téléchargée dix millions de fois par mois est
> compromise. Votre application l'utilise — vous ne l'avez jamais écrite, vous ne
> l'avez peut-être jamais lue. Est-ce votre problème ? Oui : c'est exactement ce
> que décrit une **CVE** (Common Vulnerabilities and Exposures).

**Durée de lecture :** 35 min (noyau exigé, atelier compris) · **Aller plus
loin :** +30 min (facultatif, pas exigé pour finir la journée) · **Prérequis :**
le chapitre « Anatomie d'une vulnérabilité »

---

## Objectifs

À l'issue de ce chapitre, vous serez capable de :

- **Repérer** une configuration de développement laissée en production, et dire ce
  qu'elle expose.
- **Décrire** le cycle de vie d'une faille connue : identifiant CVE, avis de
  sécurité, score de gravité, correctif.
- **Lire** un advisory de sécurité et en extraire les informations qui décident
  d'une action.
- **Relier** la chaîne de dépendances d'une application à sa surface d'attaque.

---

## I. Security misconfiguration — les réglages qu'on oublie

Un hôpital met en ligne son portail de rendez-vous. L'équipe garde le mode debug
du framework activé « pour voir les erreurs ». Quelques semaines plus tard, une
adresse inattendue répond : `/debug/config` affiche la configuration complète du
service — y compris les identifiants de la base de données.

Personne n'a forcé quoi que ce soit. La configuration **est** la faille : c'est la
famille **A05** — Security Misconfiguration, du Top 10 de l'**OWASP** (Open Web
Application Security Project).

### Les formes les plus fréquentes

- **Le mode debug en production** : pages d'erreur détaillées, traces de pile,
  adresses de diagnostic — autant de renseignements offerts.
- **Les réglages par défaut** : comptes d'administration livrés avec le produit,
  clés de démonstration, ports ouverts, listes de mots de passe connus.
- **Les pages inutiles** : une documentation technique, un ancien service, un
  tableau de bord oublié — chaque page oubliée est une porte de plus.
- **Les en-têtes de sécurité absents** : le serveur ne dit pas au navigateur ce
  qu'il a le droit d'exécuter ou de conserver.

> 🧪 **Atelier rapide (8 min)**
>
> Le service `vulnerable` du lab expose volontairement sa configuration. Après
> `cd lab-securite` puis `docker compose up --build -d`, interrogez-le :
>
> ```bash
> curl http://localhost:8080/debug/config
> ```
>
> Listez les informations qui fuient : identifiants, chemins, versions. Pour
> chacune, notez en une ligne ce qu'un attaquant en ferait.

### Les défaillances cryptographiques

La famille **A02** — Cryptographic Failures, est la voisine directe : les données
sont là, mais mal protégées — mots de passe stockés en clair ou avec un condensat
faible comme `MD5` (Message-Digest Algorithm) sans sel, données transmises sans
chiffrement, secrets écrits dans un fichier de configuration versionné. Le point
commun avec la configuration : la cause racine est un **choix par défaut jamais
revu**, pas une erreur de programmation isolée.

> [!WARNING]
> Un identifiant de base de données affiché par une page de debug n'est pas un
> détail technique : c'est l'accès à tout ce que la base contient. La gravité se
> mesure à ce qui est derrière, pas à l'élégance de la faille.

---

## II. Vulnerable components — le code qu'on n'a pas écrit

Le code d'une application moderne est majoritairement fait de **dépendances** :
bibliothèques, frameworks, images de conteneurs, briques d'infrastructure. Une
faille dans l'une d'elles est une faille de l'application — c'est la famille
**A06** — Vulnerable and Outdated Components.

### Ce qu'est une CVE

Quand une faille est découverte dans un produit utilisé ailleurs, elle reçoit un
identifiant normalisé : **CVE** (Common Vulnerabilities and Exposures), au format
`CVE-année-numéro`. Cet identifiant est le point de rendez-vous de toute la
communauté : éditeur, chercheurs, équipes de sécurité parlent de la même faille.

Autour de l'identifiant, un **advisory** — un avis de sécurité — rassemble ce
qu'il faut savoir :

| Élément | Ce qu'il apporte |
| --- | --- |
| le produit et les versions touchées | savoir si vous êtes concerné |
| le mécanisme de la faille | comprendre ce qui peut arriver |
| le score **CVSS** (Common Vulnerability Scoring System) | une gravité comparable entre les failles |
| le correctif, ou le contournement | ce qu'il faut faire, et dans quel délai |
| les références | l'annonce de l'éditeur, la base **NVD** (National Vulnerability Database), l'avis **GHSA** (GitHub Security Advisory) |

### Le cycle de vie d'une faille connue

Une faille connue vit en quatre temps :

1. **Découverte** — un chercheur, un client, ou l'éditeur lui-même trouve le
   problème. Pendant cette phase, l'information est tenue secrète pour laisser le
   temps de préparer un correctif.
2. **Publication** — l'identifiant CVE et l'advisory sortent. Tout le monde apprend
   l'existence de la faille — y compris ceux qui vont l'exploiter.
3. **Correctif** — l'éditeur publie une version corrigée. Entre la publication et
   le déploiement de cette version chez vous, la faille est ouverte.
4. **Exploitation** — des preuves de concept circulent, puis des outils
   d'attaque automatisés. Le délai entre la publication et l'exploitation massive
   se compte en heures, parfois en minutes.

C'est la **course correctif / exploitation** : le défenseur doit produire,
tester et déployer plus vite que l'attaquant ne s'adapte. Un correctif installé
mais jamais redéployé ne corrige rien.

> [!TIP]
> La question n'est pas « avons-nous des dépendances vulnérables ? » — la réponse
> est oui, tôt ou tard. C'est « savons-nous lesquelles tournent en production, et
> en combien de temps les mettons-nous à jour ? ».

### D'où viennent les alertes

- les avis des éditeurs et des projets open source ;
- les bases publiques comme **NVD** (National Vulnerability Database) ;
- les avis de sécurité comme ceux du **CERT** (Computer Emergency Response Team)
  de l'Union européenne ;
- les outils d'analyse des dépendances — **SCA** (Software Composition Analysis),
  qui comparent la liste de vos dépendances aux failles connues : `pip-audit`,
  `trivy`, ou les alertes intégrées aux forges.

---

## III. Lire un advisory réel

Savoir qu'une CVE existe ne suffit pas : encore faut-il décider **ce que cela
change pour votre système**. Un advisory se lit comme une fiche — produit,
versions, gravité, correctif — puis se traduit en avis d'exposition : êtes-vous
concerné, où, et dans quel délai agir.

L'exercice qui suit vous
fait travailler sur un avis réel du
CERT de l'Union européenne, et sur le cas d'un système d'information data qui
doit décider quoi traiter en premier.

### Exercice 1 — Lire un advisory et rédiger un avis d'exposition (35 min)

#### Contexte

Le 20 octobre 2026, le **CERT** (Computer Emergency Response Team) de l'Union
européenne publie l'avis de référence **2026-018** sur **PHP** (PHP: Hypertext
Preprocessor), le langage qui fait tourner une bonne part des applications web.
Voici l'essentiel de l'avis, tel qu'il a été relayé aux équipes :

> **Advisory CERT-EU 2026-018 — PHP**
>
> - **Produit** : PHP (PHP: Hypertext Preprocessor).
> - **Versions concernées** : les versions antérieures au correctif du 21 octobre
> 2026.
> - **20 octobre 2026** : deux failles d'exécution de code à distance non
> authentifiées sont annoncées — score **CVSS** (Common Vulnerability Scoring
> System) de 9.2.
> - **21 octobre 2026, 13:30 UTC** (temps universel coordonné) : le correctif est
> publié.
> - **Dans la foulée** : des preuves de concept — les premiers exploits
> fonctionnels — circulent publiquement.
> - **23 octobre 2026** : une troisième faille est annoncée sur le même produit.
> - **Recommandation** : appliquer le correctif dès que possible ; à défaut,
> restreindre l'accès aux serveurs concernés.

Vous travaillez pour un opérateur de mobilité, sur son système d'information
**data** — la plateforme qui produit les statistiques de fréquentation et les
tableaux de bord ouverts aux collectivités. Voici l'inventaire tenu par l'équipe :

| Élément du système | Technologie | Exposition |
| --- | --- | --- |
| portail de reporting interne | PHP | intranet, accès authentifié |
| tableaux de bord publics | PHP | internet, derrière un reverse-proxy |
| orchestration des traitements | Airflow | intranet, accès authentifié |
| stockage objet des exports | MinIO | intranet |
| API de données | Python | internet |

---

#### Consignes

##### Partie 1 — Extraire les champs de l'avis

À partir du seul extrait ci-dessus, remplissez une fiche en cinq lignes : produit,
versions concernées, gravité, date du correctif, recommandation. Ne recopiez pas
l'avis : reformulez chaque champ en une phrase.

##### Partie 2 — Lire la chronologie

Placez les quatre dates de l'avis sur une frise, puis répondez :

- combien de temps sépare l'annonce des failles de la publication du correctif ;
- ce que signifie, pour une équipe qui exploite du PHP, la phrase « des preuves de
  concept circulent dans la foulée » ;
- ce que change l'annonce d'une troisième faille le 23 octobre, alors que le
  correctif du 21 n'est peut-être pas encore déployé.

##### Partie 3 — Évaluer l'exposition du système

Pour chacun des cinq éléments de l'inventaire, répondez **concerné**, **non
concerné** ou **à vérifier**, avec une justification d'une ligne. Puis classez les
éléments concernés par ordre de traitement, en explicitant votre critère.

##### Partie 4 — Rédiger l'avis d'exposition

En une dizaine de lignes destinées au responsable du système d'information,
rédigez l'avis d'exposition : ce qui est touché, ce que cela peut coûter, ce qui
doit être fait, dans quel ordre et pour quand. Un lecteur non technique doit
pouvoir décider en le lisant.

---

#### ✅ Auto-évaluation

Avant de passer au corrigé, vérifiez que :

- [ ] Ma fiche de l'avis contient les **cinq champs**, reformulés, sans copier le
      texte d'origine.
- [ ] Ma chronologie donne les **quatre dates** et ce que chacune impose à
      l'équipe.
- [ ] J'ai donné un verdict pour **les cinq éléments** de l'inventaire, avec une
      justification — y compris pour ceux que je juge non concernés.
- [ ] Mon classement affiche un **critère explicite** (exposition, données
      touchées, nombre de personnes concernées…).
- [ ] Mon avis d'exposition se lit **sans connaître l'advisory** : un décideur
      peut trancher en le lisant.

---

#### Prolongement de l'exercice

*Facultatif.* L'avis annonce une troisième faille après la publication du
correctif. Décrivez la procédure que vous mettriez en place pour que la prochaine
annonce de ce type soit traitée en une journée, sans réunion de crise.

<details>
<summary>Voir la correction</summary>

> Comparez d'abord votre travail aux **étapes** ci-dessous : la méthode compte autant que le résultat. Tous les chiffres employés ici sont ceux de l'avis reproduit dans l'énoncé.

#### Partie 1 — Extraire les champs de l'avis

**Éléments attendus :**

| Champ | Ce qui est attendu |
| --- | --- |
| Produit | PHP (PHP: Hypertext Preprocessor), le langage qui exécute les applications web côté serveur |
| Versions concernées | toutes les versions antérieures au correctif du 21 octobre 2026 |
| Gravité | très élevée : exécution de code à distance, sans authentification, score **CVSS** (Common Vulnerability Scoring System) de 9.2 |
| Date du correctif | 21 octobre 2026, 13:30 UTC (temps universel coordonné) |
| Recommandation | appliquer le correctif dès que possible ; à défaut, restreindre l'accès aux serveurs concernés |

**Ce qu'il faut retenir :** les deux mentions « non authentifiée » et « exécution
de code à distance » disent l'essentiel : n'importe quel visiteur d'un service
exposé peut y faire exécuter ce qu'il veut. Le score CVSS de 9.2 confirme la
gravité, mais c'est le mécanisme qui décide de la conduite à tenir.

**Erreurs fréquentes :** recopier l'avis au lieu de reformuler — une fiche
reformulée prouve qu'elle a été lue. Oublier la mention des versions : sans elle,
on ne sait pas si l'on est concerné.

---

#### Partie 2 — Lire la chronologie

**Éléments attendus :**

| Date | Ce qui se passe | Ce que cela impose |
| --- | --- | --- |
| 20 octobre 2026 | deux failles annoncées | partir du principe que les systèmes exposés sont visés |
| 21 octobre 2026, 13:30 UTC | correctif publié | la course au déploiement commence |
| dans la foulée | preuves de concept en circulation | le délai de grâce est écoulé : l'exploitation est à la portée de tous |
| 23 octobre 2026 | troisième faille annoncée | le correctif du 21 ne couvre pas tout : reprendre l'avis |

Le temps séparant l'annonce des failles de la publication du correctif est celui
d'une journée. Ce que signifie « des preuves de concept circulent dans la
foulée » : les premiers exploits fonctionnels sont publics, donc l'attaque n'est
plus réservée à ceux qui savent la construire — elle devient un outil que l'on
exécute.

L'annonce d'une troisième faille le 23 octobre, alors que le correctif du 21 n'est
peut-être pas déployé, change la nature du travail : il ne s'agit plus d'appliquer
une mise à jour et de clore le sujet, mais de **rester à jour** — reprendre l'avis,
vérifier ce que couvre le correctif installé, et tenir une veille active.

**Ce qu'il faut retenir :** une advisory n'est pas un document à lire une fois.
C'est un fil d'actualité sur un produit que l'on exploite.

**Erreurs fréquentes :** ne retenir que la date du correctif. Les dates qui
comptent pour l'exploitation sont celle de la publication — le début de la course —
et celle de la circulation des preuves de concept — la fin du délai de grâce.

---

#### Partie 3 — Évaluer l'exposition du système

**Éléments attendus :**

| Élément | Verdict | Justification |
| --- | --- | --- |
| portail de reporting interne (PHP) | **concerné** | il tourne sur le produit touché ; son exposition est limitée à l'intranet |
| tableaux de bord publics (PHP) | **concerné en priorité** | même produit, mais joignable depuis internet |
| orchestration des traitements (Airflow) | **à vérifier** | pas de PHP dans l'inventaire, mais des composants web ou des extensions peuvent en embarquer |
| stockage objet des exports (MinIO) | **non concerné** | aucun composant PHP déclaré |
| API de données (Python) | **non concerné** | autre langage, autre chaîne d'exécution |

Classement attendu, avec critère explicite — deux critères se défendent et donnent
le même ordre ici :

- **par exposition** : les tableaux de bord publics d'abord, puis le portail
  interne, parce que la distance à l'attaquant décide de la probabilité
  d'exploitation ;
- **par impact métier** : les tableaux de bord publics d'abord, parce qu'ils
  portent l'image de l'opérateur et sont consultés par les collectivités.

**Ce qu'il faut retenir :** « non concerné » se justifie autant que « concerné ».
Un verdict sans vérification de l'inventaire est une supposition — d'où le verdict
**à vérifier** pour Airflow, dont l'inventaire ne dit pas tout.

**Erreurs fréquentes :** classer par score CVSS seul. Le score est le même pour
les deux serveurs PHP ; c'est l'exposition et le contexte métier qui tranchent
entre eux.

---

#### Partie 4 — Rédiger l'avis d'exposition

**Éléments attendus** — un avis qui tient en une dizaine de lignes et contient les
cinq rubriques suivantes :

| Rubrique | Ce qu'elle doit dire |
| --- | --- |
| Contexte | l'avis **CERT** (Computer Emergency Response Team) de l'Union européenne, référence 2026-018, sur PHP, deux failles d'exécution de code à distance non authentifiées |
| Exposition | les tableaux de bord publics et le portail de reporting tournent sur le produit touché |
| Gravité | score CVSS de 9.2, preuves de concept déjà en circulation |
| Action | appliquer le correctif du 21 octobre 2026 sur les deux serveurs, vérifier les composants d'Airflow |
| Échéance | traiter les serveurs exposés à internet en premier, sans attendre la prochaine fenêtre de maintenance |

**Ce qu'il faut retenir :** l'avis d'exposition est un document de décision. Il
ne raconte pas la faille : il dit ce que la faille change **ici**, et ce qu'il
faut faire **maintenant**.

**Erreurs fréquentes :** rédiger un résumé technique de l'advisory. Le lecteur
visé n'a pas besoin du mécanisme d'exploitation : il a besoin de savoir quoi
décider.

---

#### Points à retenir

- Un advisory se lit en champs : produit, versions, gravité, correctif,
  recommandation — reformulés, jamais recopiés.
- Les dates racontent une course : publication du correctif, puis circulation des
  preuves de concept. Le délai de grâce est court.
- L'exposition se juge élément par élément, à partir d'un inventaire — et « non
  concerné » se justifie comme « concerné ».
- Le classement de traitement affiche son **critère** : exposition ou impact
  métier, mais toujours dit.
- L'avis d'exposition est un document de **décision** : contexte, exposition,
  gravité, action, échéance.

</details>

---

### Exercice 2 — Lister les vulnérabilités et leur gravité sur un extrait de code (55 min)

#### Contexte

L'association qui organise le festival « Les Nuits Sonores » fait évoluer son
application d'inscription. Le développeur historique est parti ; le nouveau
mainteneur reçoit le fichier principal, écrit en Python avec FastAPI, et le
fichier de dépendances. Avant toute évolution, l'association demande une **revue
de code sécurité** : qu'est-ce qui, dans ce code, expose les inscrits ou
l'application ?

Votre mission n'est pas de corriger : c'est de **lister**, **classer** et
**ordonner**. Les corrections viendront ensuite.

Pour vous repérer, voici les familles du Top 10 de l'**OWASP** (Open Web
Application Security Project) que cette revue est susceptible de rencontrer :

| Code | Famille | Ce qu'elle recouvre |
| --- | --- | --- |
| **A01** | Broken Access Control | faire ce qu'on n'a pas le droit de faire |
| **A02** | Cryptographic Failures | des données mal protégées, chiffrées mal ou pas |
| **A03** | Injection | confondre les données et les instructions |
| **A05** | Security Misconfiguration | réglages par défaut, debug laissé actif, secrets exposés |
| **A06** | Vulnerable and Outdated Components | une dépendance connue pour être fragile |
| **A07** | Identification and Authentication Failures | des comptes et des mots de passe mal protégés |

##### Le code à relire — `application.py`

```python
# application.py — inscriptions au festival « Les Nuits Sonores »
import hashlib
import sqlite3

from fastapi import FastAPI, Request
from fastapi.responses import HTMLResponse

app = FastAPI(debug=True)
db = sqlite3.connect("festival.db", check_same_thread=False)

# clé de service utilisée pour appeler la billetterie partenaire
CLES_API = {"billetterie": "CLE_A_PLACER_EN_VARIABLE_D_ENVIRONNEMENT"}


@app.get("/recherche")
def recherche(q: str):
    # recherche d'un artiste dans la programmation
    lignes = db.execute(
        "SELECT artiste, scene FROM concerts WHERE artiste LIKE '%" + q + "%'"
    ).fetchall()
    return lignes


@app.get("/concerts/{id_concert}", response_class=HTMLResponse)
def fiche(id_concert: int):
    artiste, description = db.execute(
        "SELECT artiste, description FROM concerts WHERE id = ?",
        (id_concert,),
    ).fetchone()
    return f"<h1>{artiste}</h1><div>{description}</div>"


@app.post("/connexion")
def connexion(identifiant: str, mot_de_passe: str):
    empreinte = hashlib.md5(mot_de_passe.encode()).hexdigest()
    ligne = db.execute(
        "SELECT id, role FROM comptes WHERE identifiant = ? AND empreinte = ?",
        (identifiant, empreinte),
    ).fetchone()
    return {"compte": ligne}


@app.get("/admin/inscriptions")
def inscriptions():
    # réservé à l'organisation : l'interface n'affiche ce lien qu'aux admins
    return db.execute("SELECT nom, courriel FROM inscriptions").fetchall()


@app.post("/inscriptions/annuler")
def annuler(id_inscription: int, request: Request):
    # le cookie de session suffit à prouver l'identité du demandeur
    db.execute("DELETE FROM inscriptions WHERE id = ?", (id_inscription,))
    return {"annulee": id_inscription}
```

##### Le fichier de dépendances — `requirements.txt`

```text
fastapi==0.95.0
requests==2.19.0
pillow==8.0.0
```

L'équipe n'a jamais vérifié si des failles publiques concernaient ces versions.

---

#### Consignes

##### Partie 1 — Inventorier les constats

Listez **tous** les constats de sécurité que vous repérez. Pour chacun, donnez :

- l'endroit concerné (la fonction ou la construction de code, pas un numéro de
  ligne) ;
- la famille OWASP correspondante, par son code ;
- la cause racine, en une phrase.

##### Partie 2 — Évaluer la gravité

Pour chaque constat, évaluez la gravité **dans ce contexte** : l'application
contient des noms, des courriels et des paiements d'inscrits au festival, et elle
est accessible depuis internet. Appuyez votre évaluation sur deux éléments :
l'exposition et ce que l'organisation y perd.

##### Partie 3 — Ordonner les corrections

Classez les constats par ordre de traitement, et justifiez les deux premiers en
deux lignes chacun : quelle mesure de cause racine traitez-vous, et pourquoi
celle-là d'abord ?

##### Partie 4 — Les limites de la revue

Une revue de code ne voit pas tout. Citez deux points que ce code ne permet pas
de trancher et qui demandent un autre type de vérification (test, configuration
du serveur, environnement d'exécution…).

---

#### ✅ Auto-évaluation

Avant de passer au corrigé, vérifiez que :

- [ ] J'ai listé **au moins six constats**, et je peux dire pourquoi chacun en est
      un.
- [ ] Chaque constat porte une **famille OWASP** et une **cause racine**, pas
      seulement une description du risque.
- [ ] Mon évaluation de gravité s'appuie sur **l'exposition** et sur **ce que
      l'organisation y perd**, pour chaque constat.
- [ ] Mon ordre de traitement est **justifié**, et les deux premiers sont traités
      par une mesure de cause racine.
- [ ] J'ai nommé **deux limites** de la revue de code — ce que le texte du code
      ne dit pas.

---

#### Prolongement de l'exercice

*Facultatif.* Parmi vos constats, l'un d'eux ne se voit pas dans le code de
l'application mais dans ce qui l'entoure. Lequel, et quelle vérification
l'écarterait ou le confirmerait ?

<details>
<summary>Voir la correction</summary>

> Comparez d'abord votre travail aux **étapes** ci-dessous : la méthode compte autant que le résultat. Les familles citées sont celles du Top 10 de l'**OWASP** (Open Web Application Security Project).

#### Partie 1 — Inventorier les constats

**Éléments attendus :** huit constats sont repérables dans ce code. Six
constituent le minimum attendu ; les deux derniers font la différence.

| Endroit | Famille | Cause racine |
| --- | --- | --- |
| la fonction `recherche` | **A03** — Injection | la saisie `q` est concaténée dans la requête SQL |
| la fonction `fiche` | **A03** — Injection | la description de l'artiste est insérée dans la page HTML sans être échappée |
| la fonction `inscriptions` | **A01** — Broken Access Control | aucune vérification de rôle côté serveur |
| la fonction `annuler` | **A01** — Broken Access Control | l'action est prouvée par le seul cookie, sans jeton d'origine ni vérification du propriétaire |
| la fonction `connexion` | **A07** — Identification and Authentication Failures | le mot de passe est réduit à un condensat MD5 sans sel |
| la constante `CLES_API` | **A05** — Security Misconfiguration | un secret est écrit dans le code versionné |
| la création de l'application | **A05** — Security Misconfiguration | le mode debug est activé |
| le fichier `requirements.txt` | **A06** — Vulnerable and Outdated Components | des dépendances vieilles de plusieurs années, jamais auditées |

**Ce qu'il faut retenir :** les deux fonctions `recherche` et `fiche` relèvent de
la même famille pour des raisons différentes — l'une mélange données et
instructions, l'autre mélange texte et code dans une page. Le critère reste le
mécanisme, pas la syntaxe.

**Erreurs fréquentes :** s'arrêter aux deux injections visibles et manquer les
absences — un contrôle de rôle manquant ne se « voit » pas comme une ligne
dangereuse, il se voit à ce qui manque. De même, le mode debug et la clé en clair
ne plantent rien : ils exposent.

---

#### Partie 2 — Évaluer la gravité

**Éléments attendus** — l'application contient des noms, des courriels et des
paiements, et elle est joignable depuis internet :

| Constat | Gravité ici | Exposition et perte |
| --- | --- | --- |
| injection SQL dans `recherche` | **critique** | ouverte à tous les visiteurs ; la base entière peut être lue |
| contrôle d'accès absent sur `inscriptions` | **critique** | la liste des inscrits — noms, courriels — est livrée à quiconque la demande |
| secret en clair dans `CLES_API` | **élevée** | le code circule (versions, sauvegardes, recrutements) ; la clé ouvre la billetterie partenaire |
| **XSS** (Cross-Site Scripting) stocké dans `fiche` | **élevée** | chaque visiteur de la fiche exécute le script ; un compte d'organisation touché change tout |
| mot de passe en MD5 sans sel | **élevée** | une fuite de la base livre les mots de passe réutilisés ailleurs |
| action `annuler` sans jeton ni propriétaire | **moyenne à élevée** | annulation d'inscriptions par un tiers, ou par simple piègeage du navigateur |
| mode debug activé | **moyenne** | les traces d'erreur renseignent l'attaquant sur le code et l'environnement |
| dépendances obsolètes | **à qualifier** | la gravité dépend des failles publiques réellement affectant ces versions |

**Ce qu'il faut retenir :** la gravité se lit à l'intersection de l'exposition et
de ce que l'organisation y perd. Le condensat MD5 et le mode debug sont
techniquement « petits » ; dans une application qui traite des paiements, ils ne
le sont pas.

**Erreurs fréquentes :** attribuer la même gravité à tous les constats, ou juger
sur la seule sophistication technique. Un constat « à qualifier » est une réponse
honnête : tant que les dépendances n'ont pas été auditées, la gravité reste
inconnue.

---

#### Partie 3 — Ordonner les corrections

**Éléments attendus** — deux ordres se défendent, à condition d'afficher le
critère :

- **en priorité d'exposition** : le contrôle d'accès de `inscriptions` d'abord —
  la route livre immédiatement les données personnelles, sans rien exploiter ;
  puis l'injection SQL de `recherche`, qui ouvre la base entière.
- **en priorité de potentiel d'exploitation** : l'injection SQL d'abord — elle
  permet d'aller au-delà de la table visée ; puis le contrôle d'accès.

Dans les deux cas, les mesures de cause racine attendues sont les mêmes :

| Constat | Mesure de cause racine |
| --- | --- |
| injection SQL | requête préparée avec un emplacement réservé pour `q` |
| contrôle d'accès absent | vérification du rôle côté serveur, à chaque requête, refus par défaut |

**Ce qu'il faut retenir :** l'ordre importe moins que la visibilité du critère.
Une équipe qui sait pourquoi elle traite l'un avant l'autre peut réviser son
ordre quand le contexte change ; une équipe qui suit une liste ne le peut pas.

**Erreurs fréquentes :** proposer « ajouter un filtre sur les caractères
spéciaux » ou « masquer le lien d'administration » — deux mesures qui traitent le
symptôme et laissent la cause en place.

---

#### Partie 4 — Les limites de la revue

**Éléments attendus** — deux points au minimum, parmi :

- **la configuration du serveur** : en-têtes HTTP, chiffrement des échanges,
  version du serveur — rien de tout cela n'apparaît dans le code de
  l'application ;
- **les droits du compte de base de données** : le code ne dit pas ce que le
  compte `sqlite` peut faire au-delà de cette application ;
- **la surveillance** : rien n'indique si les connexions et les actions sensibles
  sont tracées, ni si quelqu'un les lit ;
- **le comportement en production** : un test d'intrusion local ne révèle ni les
  proxies, ni les redirections, ni les services voisins ;
- **les dépendances réelles** : le fichier de dépendances liste ce qui est
  déclaré, pas ce qui est installé et chargé à l'exécution.

**Ce qu'il faut retenir :** une revue de code éclaire une partie du système. La
dire complète est la meilleure façon de manquer ce qu'elle ne couvre pas.

---

#### Points à retenir

- Une revue de code se mène en **inventaire** : chaque constat porte un endroit,
  une famille et une cause racine.
- Les constats les plus faciles à rater sont les **absences** — contrôle de rôle,
  jeton, échappement — pas les lignes visiblement dangereuses.
- La gravité se juge dans le **contexte** : exposition et données derrière, pas
  sophistication technique.
- L'ordre de correction affiche son **critère** ; les mesures retenues traitent la
  cause racine, pas le symptôme.
- La revue de code a des **limites** : les nommer fait partie du travail.

</details>

---

## IV. Aller plus loin

*Contenu facultatif, pas exigé pour finir la journée : ces approfondissements
ne sont jamais exigés pour terminer le cours.*

- **Les formes détaillées des défaillances cryptographiques (A02)** : des mots
  de passe stockés en clair, ou avec un condensat faible comme `MD5`
  (Message-Digest Algorithm) sans sel ; des données personnelles transmises ou
  stockées sans chiffrement ; un chiffrement maison, ou un algorithme obsolète
  conservé « par compatibilité » ; des secrets — clés, jetons, mots de passe de
  service — écrits dans le code ou dans un fichier de configuration versionné.

- **Le registre des dépendances** (**SBOM**, Software Bill of Materials) : la
  liste formalisée de ce qui compose une application, dans un format échangeable.
  Quand une faille sort, c'est ce registre qui répond en quelques minutes à la
  question « sommes-nous concernés ? ».
- **Le durcissement de configuration** : les référentiels de configuration sûre —
  une configuration de référence, versionnée, appliquée à chaque déploiement, au
  lieu d'un serveur réglé à la main.
- **La réponse à incident côté dépendances** : qui décide, qui déploie, qui
  vérifie — la procédure se prépare avant la faille, pas pendant.

---

## Points clés

- Une configuration de développement laissée en production expose autant qu'une
  faille de code — et se corrige en quelques réglages.
- Les défaillances cryptographiques et les mauvaises configurations partagent une
  cause : un **choix par défaut jamais revu**.
- Une **CVE** identifie une faille connue ; l'advisory qui l'accompagne donne le
  produit, les versions, la gravité et le correctif.
- Le cycle découverte → publication → correctif → exploitation se joue en heures :
  la course se gagne à la **veille** et à la vitesse de déploiement.
- La question utile n'est pas « avons-nous des failles ? » mais « connaissons-nous
  nos dépendances, et en combien de temps les corrigeons-nous ? ».

---

## 🧠 Vérifiez vos acquis

*Formatif, jamais noté. Répondez sans relire le chapitre, puis dépliez « Vérifier ses réponses » pour vous corriger : ce qui compte est de repérer où vous vacillez, pas de faire 5 sur 5.*

**1.** Le portail de rendez-vous d'un hôpital laisse `/debug/config` répondre avec la configuration complète, y compris les identifiants de la base. De quelle famille du Top 10 de l'**OWASP** (Open Web Application Security Project) s'agit-il ?

- A. A06 — Vulnerable and Outdated Components, puisque la configuration vient du framework
- B. A05 — Security Misconfiguration, puisque la configuration est la faille
- C. A02 — Cryptographic Failures, puisque des identifiants circulent
- D. A01 — Broken Access Control, puisque personne n'a forcé la porte

**2.** Que partagent les réglages par défaut laissés en production et les mots de passe stockés avec un condensat faible comme `MD5` (Message-Digest Algorithm) sans sel ?

- A. D'être des erreurs de programmation isolées
- B. De ne devenir graves qu'avec une exposition internet
- C. De se corriger avec un analyseur de dépendances
- D. D'avoir la même cause : un choix par défaut jamais revu

**3.** Dans un advisory, quel élément vous dit d'abord si **vous** êtes concerné ?

- A. Le score **CVSS** (Common Vulnerability Scoring System)
- B. Les références vers la base **NVD** (National Vulnerability Database)
- C. Le produit et les versions touchées
- D. Le mécanisme de la faille

**4.** L'éditeur publie une version corrigée lundi ; votre service tourne toujours sur la version de vendredi, et des preuves de concept circulent depuis mardi. Où en êtes-vous dans le cycle de vie de la faille ?

- A. En phase de découverte : l'éditeur n'a prévenu personne, l'information est encore tenue secrète
- B. La faille est ouverte : entre la publication du correctif et son déploiement chez vous, rien n'est corrigé
- C. La faille est refermée : le correctif existe et la version corrigée est publiée, donc le risque est passé
- D. C'est sans importance : entre un correctif publié et une exploitation massive, il se passe toujours des mois

**5.** *Question ouverte (2-3 lignes).* Le chapitre remplace la question « avons-nous des dépendances vulnérables ? » par une autre. Quelle est-elle, et qu'implique-t-elle concrètement pour une équipe ?

---

<details>
<summary>Vérifier ses réponses</summary>

### Réponses

**1. B.** Personne n'a forcé quoi que ce soit : le mode debug du framework était resté activé, la configuration **est** la faille — famille A05.
**Pourquoi :** les autres familles décrivent d'autres mécanismes ; ici, la gravité se mesure à ce qui est derrière la page, et un identifiant de base affiché vaut l'accès à tout son contenu.

**2. D.** Les deux viennent d'un réglage par défaut accepté puis oublié, pas d'une faute écrite ligne à ligne.
**Pourquoi :** c'est le point commun que le chapitre établit entre A05 et A02 — et c'est pour cela que ces failles se corrigent par des choix assumés et revus, pas par un correctif ponctuel.

**3. C.** Sans le produit et les versions concernées, rien ne permet de dire si votre système est dans la liste.
**Pourquoi :** le CVSS compare des gravités entre failles, les références renvoient aux annonces, le mécanisme explique ce qui peut arriver — mais la première question est « sommes-nous concernés ? », et c'est aussi le rôle de l'avis d'exposition.

**4. B.** Le correctif existe mais n'est pas déployé : la fenêtre d'exposition reste ouverte, et le délai entre publication et exploitation se compte en heures, parfois en minutes.
**Pourquoi :** c'est la course correctif / exploitation — un correctif installé mais jamais redéployé ne corrige rien.

**5.** Attendu : « savons-nous quelles dépendances vulnérables tournent en production, et en combien de temps nous les mettons à jour ? ». Implications attendues : connaître ses dépendances (analyse type **SCA** — Software Composition Analysis, alertes des éditeurs et des forges) et être capable de déployer vite.
**Pourquoi :** une faille dans une dépendance est une faille de l'application ; la course se gagne à la veille et à la vitesse de déploiement, pas en espérant n'être jamais concerné.

</details>

---

[← Chapitre précédent](03-owasp-web.md) · [Sommaire](Readme.md) · [Chapitre suivant →](05-autres-familles.md)
