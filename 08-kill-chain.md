# La kill chain — Une attaque en sept étapes

> En 2013, la chaîne de compromission d'un grand distributeur américain
> commence par un fournisseur de climatisation : un accès prestataire, puis un
> accès au réseau interne, puis le vol de dizaines de millions de cartes de
> paiement. Chaque étape de cette chaîne laissait une occasion de l'arrêter.
> Une attaque réussie n'est presque jamais un coup de génie : c'est une série
> d'échecs de défense qui s'enchaînent.

**Durée de lecture :** 60 min · **Prérequis :** le chapitre « Modèles d'attaquant »

---

## Objectifs

À l'issue de ce chapitre, vous serez capable de :

- **Décrire** les étapes d'une chaîne de compromission, de la reconnaissance à
  l'exfiltration.
- **Situer** une étape d'attaque dans la kill chain à partir d'un scénario
  réel.
- **Associer** à chaque étape la mesure de défense qui la coupe.
- **Expliquer** pourquoi plusieurs échecs doivent s'enchaîner pour qu'un
  incident grave ait lieu.

---

## I. La chaîne

Le terme **kill chain** (« chaîne d'élimination ») vient du vocabulaire
militaire : il décrit la succession d'étapes nécessaires pour atteindre une
cible. En sécurité informatique, on parle aussi de **chaîne de compromission**.
Son intérêt est simple : une attaque réussie n'est pas un événement, c'est un
**processus**, et un processus se coupe à plusieurs endroits.

Les étapes classiques, dans l'ordre :

| Étape | La question qu'elle pose | Exemple concret |
| --- | --- | --- |
| **Reconnaissance** | Que vaut la cible, et comment y entrer ? | balayage des adresses, recherche d'adresses de courriel publiques |
| **Accès initial** | Par quelle porte on entre ? | un compte prestataire oublié, une faille d'une bibliothèque obsolète |
| **Escalade de privilèges** | Comment obtenir plus de droits ? | un compte utilisateur transformé en compte d'administration |
| **Mouvement latéral** | Où aller ensuite dans le réseau ? | d'un poste de travail vers le serveur de fichiers |
| **Persistance** | Comment rester après le redémarrage ? | une tâche planifiée, un compte créé en douce |
| **Actions sur les objectifs** | Ce que l'attaquant est venu faire | vol de données, chiffrement rançongiciel, espionnage |
| **Exfiltration** | Comment faire sortir les données ? | un transfert vers un stockage en ligne, déguisé en trafic normal |

Deux précisions sur ce vocabulaire. L'**accès initial** est souvent la partie la
plus médiatisée (« une faille a été exploitée »), alors que les étapes
suivantes — escalade, mouvement, persistance — sont celles qui transforment un
incident mineur en catastrophe. Et l'**exfiltration** est l'étape la plus
discrète : les données sortent parfois pendant des semaines, à petites doses,
dans un trafic qui ressemble à n'importe quelle sauvegarde.

> [!INFO]
> Toutes les attaques ne remplissent pas les sept étapes. Un vol de mot de
> passe réutilisé peut aller de l'accès initial à l'action finale en quelques
> minutes. La kill chain n'est pas un scénario à suivre : c'est une **grille de
> lecture** pour savoir où l'on se trouve.

> 🧪 **Atelier rapide (5 min)**
>
> Reprenez l'affaire du distributeur évoquée en introduction, et placez chacun
> de ces faits sur la kill chain :
>
> - les coordonnées du prestataire de maintenance sont récupérées ;
> - le réseau interne du distributeur est atteint depuis le réseau du
>   prestataire ;
> - les droits d'un compte sont élargis pour atteindre les serveurs de caisse ;
> - les numéros de cartes sont copiés depuis les terminaux de paiement ;
> - les données sortent du réseau pendant plusieurs jours.

---

## II. Où couper la chaîne

L'intérêt pratique de la kill chain apparaît dès qu'on la lit à l'envers :
**pour chaque étape, il existe une mesure qui l'empêche ou qui la révèle**. Et
plus on coupe tôt, moins ça coûte cher.

| Étape | Couper (prévenir) | Voir (détecter) |
| --- | --- | --- |
| Reconnaissance | réduire la surface exposée, ne rien publier d'inutile | surveillance des balayages, pages pièges |
| Accès initial | mises à jour, authentification forte, filtrage des accès distants | alertes sur les connexions inhabituelles |
| Escalade de privilèges | moindre privilège, comptes d'administration cloisonnés | journalisation des changements de droits |
| Mouvement latéral | segmentation du réseau, micro-segmentation | détection des connexions internes atypiques |
| Persistance | contrôle des tâches planifiées et des comptes créés | revue périodique des comptes et des services |
| Actions sur les objectifs | cloisonnement et chiffrement des données sensibles | surveillance des accès en masse aux données |
| Exfiltration | limitation des volumes sortants, filtrage réseau | alerte sur les transferts volumétriques inhabituels |

Le cas du distributeur de 2013 l'illustre bien : la porte d'entrée était un
accès prestataire — une mesure banale (un accès distant limité, ou une
segmentation entre le réseau du fournisseur et le réseau interne) aurait coupé
la chaîne à sa deuxième étape, avant toute atteinte aux caisses.

Deux idées à retenir ici.

**La défense en profondeur** n'est pas un empilement de produits : c'est la
présence d'au moins une barrière à **chaque** étape de la chaîne. Une barrière
unique, même excellente, cède dès qu'elle est contournée.

**La détection vaut la prévention** quand on ne peut pas empêcher. Un accès
prestataire est parfois nécessaire à l'activité : on ne peut pas toujours le
supprimer. En revanche, on peut savoir qu'il a été utilisé, quand, et pour
aller où — et c'est ce qui transforme une compromission silencieuse en incident
géré.

> [!TIP]
> Pour une application concrète, listez les sept étapes dans un tableau et
> remplissez une seule colonne : « qu'est-ce qui, chez nous, empêche ou révèle
> cette étape ? ». Les cases vides sont votre plan de travail — pas les cases
> remplies.

---

## IV. Aller plus loin

*Ces approfondissements sont facultatifs : ils ne sont jamais exigés pour terminer
la journée.*

- **MITRE ATT&CK** : une base de connaissances publique qui recense les modes
  opératoires observés sur le terrain, classés par tactique (l'objectif
  poursuivi) et par technique (la manière). Là où la kill chain décrit un
  déroulé général, ATT&CK décrit des gestes précis — par exemple « comment un
  attaquant se fait passer pour un service légitime ». C'est un vocabulaire de
  travail pour les équipes de détection.
- **La chaîne de compromission côté données** : dans un système data (entrepôt,
  lac de données, chaîne de traitement), les étapes sont les mêmes mais les
  portes diffèrent — un compte de service, un accès de stockage, un export
  planifié. La grille de lecture reste valable.
- **Les retours d'expérience publiés** : les rapports d'incident détaillés
  (ANSSI, éditeurs, CERT) décrivent des chaînes complètes. En lire un en entier
  vaut mieux que dix schémas : on y voit les délais entre chaque étape.

---

## Points clés

- Une attaque réussie est un **processus** en sept étapes — reconnaissance,
  accès initial, escalade, mouvement, persistance, actions sur les objectifs,
  exfiltration — pas un événement isolé.
- La kill chain est une **grille de lecture** : elle sert à se demander, étape
  par étape, où la défense peut agir.
- **Plusieurs échecs doivent s'enchaîner** pour qu'un incident grave ait lieu —
  donc plusieurs occasions existent de l'interrompre.
- Couper tôt coûte moins cher que nettoyer tard : la porte d'entrée est rarement
  l'endroit le plus coûteux de la chaîne.
- Quand on ne peut pas empêcher une étape, on peut presque toujours la
  **détecter** : c'est le rôle des journaux et de la surveillance.
