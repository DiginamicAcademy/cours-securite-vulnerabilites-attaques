[← Chapitre précédent](04-owasp-config-deps.md) · [Sommaire](Readme.md) · [Chapitre suivant →](06-lecture-de-code-vulnerable.md)

# Autres familles de failles — le même raisonnement, d'autres portes

> Un document PDF reçu d'un fournisseur fait planter le parseur — puis exécuter
> du code qui n'a rien à faire là. Rien à voir avec SQL, ni avec le web : et
> pourtant le mécanisme est le même. Faire exécuter quelque chose qui n'était pas
> prévu.

**Durée de lecture :** 25 min (noyau exigé, atelier compris) · **Aller plus
loin :** +20 min (facultatif, pas exigé pour finir la journée) · **Prérequis :**
le chapitre « Les injections — quand les données deviennent des instructions »

---

## Objectifs

À l'issue de ce chapitre, vous serez capable de :

- **Identifier** les mécanismes de **SSRF** (Server-Side Request Forgery), d'**XXE**
  (XML External Entity) et de désérialisation non fiable.
- **Relier** chacune de ces failles à sa cause commune : trop de confiance, pas
  assez de validation.
- **Repérer**, dans un scénario de données, la famille de risque qui s'applique à
  un flux entrant.
- **Décider** ce qu'il faut vérifier en premier sur un fichier ou une requête
  venant de l'extérieur.

---

## I. SSRF, XXE, désérialisation — trois portes de derrière

Les injections vues précédemment arrivent par les formulaires. Ces trois familles
arrivent ailleurs : une adresse à aller chercher, un fichier à ouvrir, un objet à
reconstruire. Le raisonnement, lui, ne change pas.

### SSRF — le serveur qui va chercher ce qu'on lui demande

Une plateforme de vente en ligne permet de prévisualiser une image à partir de son
adresse. Le serveur va la chercher lui-même, puis l'affiche. Un vendeur soumet
ceci comme adresse d'image :

```text
http://169.254.169.254/latest/meta-data/
```

Cette adresse n'est pas celle d'une image : c'est le service de métadonnées de
l'hébergeur, joignable **depuis le serveur** et depuis lui seul. Le serveur a fait
la requête à la place du vendeur — c'est la famille **A10** — SSRF (Server-Side
Request Forgery), du Top 10 de l'**OWASP** (Open Web Application Security Project).

Ce que permet ce mécanisme : parcourir le réseau interne, atteindre des services
sans authentification — bases de données, consoles d'administration, services
d'infrastructure — ou lire des fichiers locaux selon la configuration.

### XXE — le fichier de données qui demande plus que ce qu'il contient

Un cabinet comptable importe des factures au format **XML** (eXtensible Markup
Language), fournies par ses clients. Le parseur — le programme qui lit le fichier —
accepte par défaut les **entités externes** : une déclaration placée en tête du
fichier lui demande d'aller lire ailleurs, puis d'insérer le résultat dans le
document.

```xml
<?xml version="1.0"?>
<!DOCTYPE facture [
  <!ENTITY interne SYSTEM "file:///etc/passwd">
]>
<facture><client>&interne;</client></facture>
```

La facture s'affiche alors avec, à la place du nom du client, le contenu du
fichier demandé. C'est la faille dite **XXE** (XML External Entity) : le fichier
de données devient une instruction d'accès au système. Selon la configuration du
parseur, elle peut aussi déclencher une requête vers un serveur choisi par
l'attaquant — le même effet que le SSRF, par un autre chemin.

### Désérialisation — quand les octets redeviennent des objets

Une plateforme de calcul distribué échange des résultats entre ses nœuds sous
forme d'objets Python sérialisés — avec le module `pickle`. Un fichier reçu d'un
partenaire est chargé tel quel :

```python
import pickle

# chargement d'un résultat de calcul reçu d'un nœud partenaire
resultat = pickle.loads(donnees_recues)
```

Or un objet sérialisé contient la **recette** de sa reconstruction : en le
chargeant, le programme exécute ce que le fichier lui dicte. Une désérialisation
de données non fiables est donc une exécution de code déguisée — bien plus grave
qu'une simple corruption de données.

| Famille | Ce qui arrive | Ce qui est exécuté ou lu |
| --- | --- | --- |
| SSRF | une adresse à aller chercher | une requête vers le réseau interne |
| XXE | un fichier de données à parser | une lecture de fichier, une requête sortante |
| Désérialisation | un objet à reconstruire | du code au moment du chargement |

> 🧪 **Atelier rapide (5 min)**
>
> Un éditeur de logiciels de gestion vous envoie chaque nuit un fichier XML de
> facturation, déposé sur un serveur d'échange puis importé automatiquement. Une
> alerte signale que ce partenaire a été compromis. Parmi les risques vus
> ci-dessus, lequel s'applique à ce flux, et par quelle étape exacte
> entre-t-il dans votre système ? Une phrase de réponse suffit.

---

## II. Prototype pollution — pour les applications JavaScript

Quand une application JavaScript fusionne un objet reçu de l'extérieur dans ses
propres objets, un visiteur peut insérer une clé qui modifie le **prototype**
partagé par tous les objets de la page. L'effet est discret et étendu : non pas
exécuter du code, mais **redéfinir les règles** selon lesquelles l'application
fonctionne. On la rattache à **A03** — Injection ou à **A08** — Software and
Data Integrity Failures. L'aperçu suffit ici : fusionner une donnée externe
dans ses propres structures est un geste à examiner de près.

> [!WARNING]
> Ces trois familles se repèrent moins facilement qu'une injection dans un
> formulaire : aucune ne se voit dans l'interface. Elles se voient dans les
> **points de contact avec l'extérieur** — une adresse à résoudre, un fichier à
> charger, un objet à reconstruire.

---

## III. Le même raisonnement partout

Derrière la diversité des mécanismes, une seule question revient : **qu'est-ce
que ce flux me demande d'exécuter, et l'ai-je décidé ?**

- le SSRF demande au serveur de **faire une requête** qu'il n'a pas choisie ;
- l'XXE demande au parseur de **lire un fichier** qu'il n'a pas choisi ;
- la désérialisation demande au programme de **reconstruire un objet** selon une
  recette qu'il n'a pas écrite ;
- la prototype pollution demande à l'application de **changer une règle** qu'elle
  n'a pas prévue.

Dans les quatre cas, la cause racine est la même : **trop de confiance accordée à
une donnée entrante**. Et les barrières se ressemblent :

1. **Restreindre ce qui est accepté** — liste autorisée d'adresses, de formats, de
   clés, de types.
2. **Désactiver ce qui n'est pas nécessaire** — entités externes du parseur,
   modules de sérialisation puissants, fonctions d'évaluation.
3. **Exécuter avec le moins de droits possible** — un importeur de factures n'a
   pas besoin de lire les fichiers du système, ni d'atteindre le réseau interne.
4. **Surveiller les écarts** — un import qui se met à faire des requêtes sortantes
   n'est pas normal.

Pour un flux de données entrant — un fichier reçu d'un tiers, une API partenaire,
un export importé — la question pratique est donc : *ce fichier peut-il me faire
faire quelque chose ?* Si oui, il faut savoir quoi, et l'empêcher.

### Exercice bonus — Modéliser les vulnérabilités d'un pipeline de données (libre, facultatif)

#### Contexte

Une chaîne de magasins d'équipement sportif exploite une plateforme de données
qui produit ses statistiques de ventes et ses prévisions de stock. La plateforme
a grandi par ajouts successifs ; personne n'a jamais fait le tour de ses risques.
La direction demande une **modélisation des menaces** : par où cette plateforme
peut-elle être atteinte, et que coûterait chaque voie ?

Voici la chaîne, telle qu'elle fonctionne :

1. **Réception** — chaque partenaire dépose la nuit un fichier d'export au format
   CSV (comma-separated values) sur un serveur d'échange, récupéré ensuite par
   SFTP (protocole de transfert de fichiers sur SSH (Secure Shell)).
2. **Stockage** — les fichiers bruts sont conservés dans un stockage objet de
   type MinIO, accessibles à toute l'équipe data.
3. **Orchestration** — un planificateur Airflow déclenche les traitements, avec
   des identifiants de connexion écrits dans les fichiers de définition des
   tâches.
4. **Transformation** — des scripts Python et Spark nettoient, joignent et
   agrègent les données ; certains chargent des objets Python sérialisés produits
   par des traitements antérieurs.
5. **Restitution** — une application web interne, puis un portail public,
   affichent les tableaux de bord ; une API (Application Programming Interface)
   alimente les partenaires.

Votre mission : **modéliser**, pas corriger.

---

#### Consignes

##### Partie 1 — Cartographier les menaces

Pour chacune des cinq étapes, identifiez :

- les **familles de vulnérabilités** qui s'appliquent — en vous appuyant sur le
  Top 10 de l'**OWASP** (Open Web Application Security Project) et sur les
  familles vues dans ce cours ;
- le **mécanisme** en une phrase : ce que l'attaquant fait exactement, à quelle
  étape ;
- l'**impact métier** : ce que l'entreprise y perd — données, activité,
  confiance.

Présentez votre travail sous forme de tableau. Une étape peut admettre plusieurs
familles : donnez-les toutes.

##### Partie 2 — Repérer les points de confiance

Listez les endroits de la chaîne où un élément **croit** un autre sans le
vérifier : un fichier cru sur parole, un identifiant partagé, un réseau réputé
sûr. Une phrase par point de confiance.

##### Partie 3 — La note de priorités

En dix lignes maximum, adressez-vous à la direction : quels sont les trois
risques à traiter en premier, pourquoi ceux-là, et quelle serait la première
mesure pour chacun. Votre critère de priorité doit être **dit**.

---

#### ✅ Auto-évaluation

Avant de passer au corrigé, vérifiez que :

- [ ] J'ai traité **les cinq étapes**, y compris celles qui me semblaient
      techniques et sans risque.
- [ ] Chaque menace nomme une **famille**, un **mécanisme** et un **impact
      métier** — pas seulement un danger vague.
- [ ] J'ai listé les **points de confiance** : là où un élément en croit un autre
      sans vérifier.
- [ ] Ma note de priorités contient **trois risques**, un critère de tri **dit**,
      et une première mesure par risque.
- [ ] Je peux relire mon tableau et dire, pour chaque ligne, **par quelle étape**
      l'attaquant entre.

---

#### Prolongement de l'exercice

*Facultatif.* La chaîne décrite grandit encore : un partenaire supplémentaire
déposera bientôt ses fichiers, et un tableau de bord sera ouvert aux fournisseurs.
Qu'est-ce que ces deux évolutions ajoutent à votre modélisation — et quel point de
confiance nouveau créent-elles ?

<details>
<summary>Voir la correction</summary>

> Un défi ne se corrige pas pas-à-pas : ce document donne une **grille de vérification** et des exemples de constats, pour confronter votre modélisation — jamais à recopier. Les familles citées sont celles du Top 10 de l'**OWASP** (Open Web Application Security Project).

#### Grille de vérification

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

#### Exemples de constats

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

#### Les points de confiance

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

#### La note de priorités

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

#### Points à retenir

- Une chaîne de données se modèle **étape par étape** — la réception et
  l'orchestration comptent autant que la restitution.
- Les menaces se décrivent en **gestes concrets** : une requête, un fichier, un
  identifiant — jamais en intentions.
- Les points de confiance sont des **vérifications manquantes** : les nommer fait
  la moitié du travail.
- Les risques propres aux données sont ceux que l'on oublie : fichiers crus sur
  parole, secrets partagés, objets reconstruits.
- Une note de priorités vaut par son **critère affiché**, pas par sa longueur.

</details>

---

## IV. Aller plus loin

*Contenu facultatif, pas exigé pour finir la journée : ces approfondissements
ne sont jamais exigés pour terminer le cours.*

- **La prototype pollution en détail** : les applications web modernes
  construisent leur page côté navigateur, en JavaScript. Certaines acceptent de
  fusionner un objet reçu du serveur — une préférence d'affichage, un thème,
  une configuration — dans leurs propres objets. Si la fusion accepte des clés
  choisies par l'extérieur, un visiteur peut insérer une clé qui modifie le
  **prototype** partagé par tous les objets de la page. Une valeur modifiée à
  cet endroit se retrouve partout dans l'application — y compris dans des
  chemins de code que personne n'avait pensés dangereux.

- **Les vulnérabilités de la chaîne logicielle** (supply chain) : ce n'est plus
  l'application qui est visée, mais ce qui la construit — un paquet compromis, un
  script d'intégration modifié, une image de conteneur altérée. Le raisonnement
  reste le même : une donnée — ici, du code tiers — a été crue sur parole.
- **Les parseurs XML en détail** : les options de sécurité des bibliothèques
  courantes, et pourquoi « le fichier vient d'un partenaire » n'est pas un motif
  de confiance.
- **La sérialisation sans exécution** : les formats d'échange qui séparent les
  données du code — JSON (JavaScript Object Notation), CSV (comma-separated
  values) — et quand les choisir plutôt qu'un format qui transporte des objets.

---

## Points clés

- SSRF, XXE, désérialisation et prototype pollution arrivent par des portes
  différentes : une adresse, un fichier, un objet, une configuration.
- Le mécanisme commun : une **donnée entrante** fait exécuter, lire ou modifier
  quelque chose que le programme n'avait pas décidé.
- La cause racine est partout la même : **trop de confiance, pas assez de
  validation**.
- Les barrières se ressemblent : restreindre ce qui est accepté, désactiver
  l'inutile, exécuter avec peu de droits, surveiller les écarts.
- Devant un flux entrant, la question à poser est : *ce fichier peut-il me faire
  faire quelque chose ?*

---

## 🧠 Vérifiez vos acquis

*Formatif, jamais noté. Répondez sans relire le chapitre, puis dépliez « Vérifier ses réponses » pour vous corriger : ce qui compte est de repérer où vous vacillez, pas de faire 5 sur 5.*

**1.** Une plateforme télécharge elle-même l'adresse d'image soumise par un vendeur, et celui-ci soumet `http://169.254.169.254/latest/meta-data/`. Pourquoi cette adresse est-elle dangereuse ?

- A. Parce qu'elle est invalide et fait planter le service de prévisualisation, ce qui coupe la plateforme
- B. Parce que ce service de métadonnées de l'hébergeur est joignable depuis le serveur, et depuis lui seul : c'est le serveur qui fait la requête à la place du vendeur
- C. Parce que la réponse contient du **XML** (eXtensible Markup Language) que le parseur va lire et interpréter
- D. Parce que l'image récupérée est trop volumineuse : le disque du serveur sature et le service tombe

**2.** Une facture importée commence par `<!ENTITY interne SYSTEM "file:///etc/passwd">`, et le nom du client s'affiche avec le contenu du fichier. Quelle faille, et quel réglage la rend possible ?

- A. Une SSRF (Server-Side Request Forgery) : le serveur n'a pas restreint la liste des adresses autorisées
- B. Une **XXE** (XML External Entity) : le parseur accepte les entités externes par défaut
- C. Une désérialisation non fiable : le fichier est reconstruit en objet et sa recette s'exécute au chargement
- D. Une prototype pollution : la clé insérée modifie le prototype partagé et redéfinit les règles de la page

**3.** `resultat = pickle.loads(donnees_recues)` sur un fichier reçu d'un partenaire. Pourquoi est-ce plus grave qu'une simple corruption de données ?

- A. Parce que le module est lent sur les gros fichiers et bloque les nœuds de traitement pendant la lecture
- B. Parce que le fichier peut contenir une condition SQL qui réécrit la requête du traitement
- C. Parce qu'un objet sérialisé contient la recette de sa reconstruction : le programme exécute ce que le fichier lui dicte
- D. Parce que le format ne vérifie pas l'encodage des caractères et que le texte importé arrive corrompu

**4.** En quoi la prototype pollution se distingue-t-elle des trois familles précédentes, telle que la décrit le chapitre ?

- A. Elle ne s'attaque qu'aux applications écrites en Python, dont les objets partagent un modèle commun
- B. Elle ne cherche pas à exécuter du code mais à redéfinir les règles selon lesquelles l'application fonctionne
- C. Elle est la seule des quatre visible dans l'interface, donc la seule à chercher en revue d'écran
- D. Elle nécessite un fichier binaire pour être déclenchée, ce qui la rend invisible en revue de code

**5.** *Question ouverte (2-3 lignes).* Reprenez la question du chapitre — « ce fichier peut-il me faire faire quelque chose ? » — sur le flux de l'atelier : un fichier XML de facturation déposé chaque nuit par un partenaire désormais compromis, importé automatiquement. Nommez la famille de risque, l'étape exacte où elle entre dans le système, et une barrière du chapitre qui la coupe.

---

<details>
<summary>Vérifier ses réponses</summary>

### Réponses

**1. B.** C'est le mécanisme du SSRF, famille A10 : le serveur fait une requête qu'il n'a pas choisie, vers un service atteint depuis l'intérieur seulement.
**Pourquoi :** par ce chemin on parcourt le réseau interne, on atteint des services sans authentification (bases, consoles d'administration, services d'infrastructure) et parfois des fichiers locaux.

**2. B.** La déclaration en tête du fichier demande au parseur d'aller lire ailleurs puis d'insérer le résultat : le fichier de données devient une instruction d'accès au système.
**Pourquoi :** c'est le réglage par défaut du parseur qui ouvre la porte — selon la configuration, la même faille peut aussi déclencher une requête sortante, comme en SSRF.

**3. C.** Charger l'objet, c'est suivre la recette qu'il apporte : une désérialisation de données non fiables est une exécution de code déguisée.
**Pourquoi :** les trois familles du chapitre reviennent à la même cause — trop de confiance accordée à une donnée entrante.

**4. B.** L'effet est discret et étendu : une clé insérée modifie le prototype partagé par tous les objets de la page, donc le comportement de l'application.
**Pourquoi :** le chapitre la rattache à A03 — Injection ou A08 — Software and Data Integrity Failures, et retient le geste à examiner : fusionner une donnée externe dans ses propres structures.

**5.** Attendu : famille XXE ; l'entrée se fait à l'étape de lecture du fichier par le parseur, qui accepte les entités externes ; barrière attendue : désactiver les entités externes du parseur (ou restreindre les formats acceptés, exécuter l'importeur avec le moins de droits possible, surveiller les requêtes sortantes inhabituelles).
**Pourquoi :** aucune de ces failles ne se voit dans l'interface — elles se voient aux points de contact avec l'extérieur, ici un fichier déposé par un tiers.

</details>

---

[← Chapitre précédent](04-owasp-config-deps.md) · [Sommaire](Readme.md) · [Chapitre suivant →](06-lecture-de-code-vulnerable.md)
