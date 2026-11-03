# Corrigé — Modéliser les vulnérabilités d'un pipeline de données

> Corrigé de [le défi « Modéliser les vulnérabilités d'un pipeline de données
> »](../04-defi-menace-data.md). Un défi ne se corrige pas pas-à-pas : ce
> document donne une **grille de vérification** et des exemples de constats, pour
> confronter votre modélisation — jamais à recopier. Les familles citées sont
> celles du Top 10 de l'**OWASP** (Open Web Application Security Project).

---

## Grille de vérification

Vérifiez votre travail critère par critère. Un critère non couvert indique où
reprendre ; il ne s'agit pas d'obtenir « la bonne réponse », mais une
modélisation défendable.

| Critère | Ce qui doit être vérifiable dans votre travail |
| --- | --- |
| Couverture | les cinq étapes sont traitées, y compris la réception et l'orchestration |
| Mécanisme | chaque menace décrit un geste d'attaquant concret — une requête, un fichier, un identifiant — pas une intention |
| Familles | les familles citées existent et correspondent au mécanisme décrit |
| Impact | chaque menace dit ce que l'entreprise y perd, en termes métier |
| Points de confiance | au moins trois endroits où une vérification manque sont nommés |
| Priorités | trois risques, un critère de tri explicite, une première mesure chacun |

---

## Exemples de constats

Ces exemples illustrent le niveau de détail attendu. Ils ne constituent pas une
liste exhaustive : une modélisation qui trouve d'autres voies, à condition de les
relier à une étape précise, est tout aussi valable.

| Étape | Exemple de menace | Famille | Ce qu'il faut retrouver |
| --- | --- | --- | --- |
| Réception | un fichier partenaire lu sans vérification | Software and Data Integrity Failures | le point de confiance « le partenaire » : le fichier est cru sur parole, alors qu'il provient d'un système compromettable |
| Stockage | un accès ouvert à toute l'équipe data | Broken Access Control | la différence entre « l'équipe a besoin des données » et « chaque personne a besoin de tout » |
| Orchestration | des identifiants écrits dans les définitions de tâches | Cryptographic Failures | un secret versionné fuit avec le dépôt ; l'impact dépasse la plateforme data |
| Transformation | le chargement d'objets Python sérialisés | Software and Data Integrity Failures | ce qui est exécuté au chargement — la recette de reconstruction de l'objet |
| Transformation | des scripts qui assemblent des requêtes ou des commandes | Injection | la confusion données / instructions, sur ce qui provient des fichiers reçus |
| Restitution | le portail public et son API | Broken Access Control, Injection | ce qui est joignable de l'extérieur : contrôle d'accès côté serveur, saisies traitées comme des données |
| Toute la chaîne | des composants jamais mis à jour | Vulnerable and Outdated Components | la chaîne de dépendances comme surface d'attaque, et l'absence d'audit |

**Ce qu'il faut retenir :** une chaîne de données n'est pas une application web
comme les autres. Ses portes d'entrée sont les **fichiers reçus**, les **secrets
partagés** et les **objets reconstruits** — autant de chemins qui ne passent pas
par un formulaire.

**Erreurs fréquentes :** traiter la chaîne comme une application web ordinaire et
manquer les risques propres aux flux de données ; décrire des scénarios
spectaculaires sans les relier à une étape précise ; oublier l'orchestration, qui
pourtant détient les accès.

---

## Les points de confiance

**Éléments attendus** — au moins trois, parmi :

- le **partenaire** : son fichier est accepté sans vérification d'intégrité ni de
  forme ;
- le **réseau interne** : la plateforme data est réputée sûre, donc peu protégée
  de l'intérieur ;
- les **identifiants partagés** : une seule paire d'identifiants sert à plusieurs
  usages, et personne ne sait qui l'utilise ;
- les **traitements antérieurs** : un objet sérialisé est chargé parce qu'il vient
  d'un calcul maison ;
- l'**équipe** : l'accès au stockage est accordé par appartenance à l'équipe, pas
  par besoin.

**Ce qu'il faut retenir :** un point de confiance est un endroit où une
vérification **manque** — pas une faute de programmation. C'est pour cela qu'ils
se voient mal dans le code et bien dans une modélisation.

---

## La note de priorités

**Éléments attendus** — trois risques, un critère dit, une première mesure par
risque. Deux critères se défendent :

- **par exposition** : traiter d'abord ce qui est joignable de l'extérieur — le
  portail public et son API — puis les secrets partagés, puis les flux reçus ;
- **par impact en cascade** : traiter d'abord ce qui ouvre le plus de portes —
  les identifiants écrits dans les définitions de tâches, qui donnent accès à
  toute la chaîne — puis la restitution publique, puis les fichiers reçus.

Premières mesures attendues, à titre d'exemple :

| Risque | Première mesure |
| --- | --- |
| identifiants dans les définitions de tâches | sortir les secrets du code, les injecter à l'exécution |
| portail public sans contrôle vérifié | vérifier le contrôle d'accès côté serveur, à chaque requête |
| fichiers reçus sans vérification | valider la forme et l'origine de chaque fichier avant traitement |

**Ce qu'il faut retenir :** la note de priorités est un document de décision. Elle
ne gagne pas en exhaustivité, elle gagne en **clarté** : trois risques, un
critère, une mesure.

---

## Points à retenir

- Une chaîne de données se modèle **étape par étape** — la réception et
  l'orchestration comptent autant que la restitution.
- Les menaces se décrivent en **gestes concrets** : une requête, un fichier, un
  identifiant — jamais en intentions.
- Les points de confiance sont des **vérifications manquantes** : les nommer fait
  la moitié du travail.
- Les risques propres aux données sont ceux que l'on oublie : fichiers crus sur
  parole, secrets partagés, objets reconstruits.
- Une note de priorités vaut par son **critère affiché**, pas par sa longueur.
