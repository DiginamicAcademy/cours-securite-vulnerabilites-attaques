# Autres familles de failles — le même raisonnement, d'autres portes

> Un document PDF reçu d'un fournisseur fait planter le parseur — puis exécuter
> du code qui n'a rien à faire là. Rien à voir avec SQL, ni avec le web : et
> pourtant le mécanisme est le même. Faire exécuter quelque chose qui n'était pas
> prévu.

**Durée de lecture :** 45 min · **Prérequis :** le chapitre « Les injections —
quand les données deviennent des instructions »

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

Les applications web modernes construisent leur page côté navigateur, en
JavaScript. Certaines acceptent de fusionner un objet reçu du serveur — une
préférence d'affichage, un thème, une configuration — dans leurs propres objets.
Si la fusion accepte des clés choisies par l'extérieur, un visiteur peut insérer
une clé qui modifie le **prototype** partagé par tous les objets de la page.

L'effet est discret et étendu : une valeur modifiée à cet endroit se retrouve
partout dans l'application — y compris dans des chemins de code que personne
n'avait pensés dangereux. C'est la **prototype pollution** : non pas exécuter du
code, mais **redéfinir les règles** selon lesquelles l'application fonctionne.

On la rattache à la famille **A03** — Injection, quand elle permet d'injecter du
code, ou à **A08** — Software and Data Integrity Failures, quand elle corrompt des
données que l'application croit fiables. L'aperçu suffit ici : retenir que
fusionner une donnée externe dans ses propres structures est un geste à examiner
de près.

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
   modules de sérialisation puissants, fonctions d'évaluation.3. **Exécuter avec le moins de droits possible** — un importeur de factures n'a
   pas besoin de lire les fichiers du système, ni d'atteindre le réseau interne.
4. **Surveiller les écarts** — un import qui se met à faire des requêtes sortantes
   n'est pas normal.

Pour un flux de données entrant — un fichier reçu d'un tiers, une API partenaire,
un export importé — la question pratique est donc : *ce fichier peut-il me faire
faire quelque chose ?* Si oui, il faut savoir quoi, et l'empêcher.

---

## IV. Aller plus loin

*Ces approfondissements sont facultatifs : ils ne sont jamais exigés pour terminer
la journée.*

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
