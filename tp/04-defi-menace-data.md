# Défi — Modéliser les vulnérabilités d'un pipeline de données

> 🔴 **Défi** · **Sans limite de temps** · **Lien :** chapitre [Autres familles de
> failles — le même raisonnement, d'autres portes](../05-autres-familles.md)
> **Objectif :** modéliser, étape par étape, les vulnérabilités propres à une
> chaîne de traitement de données — et en tirer une note de priorités. Ce défi est
> **facultatif** : il ne bloque jamais la progression de la journée.

---

## Contexte

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

## Consignes

### Partie 1 — Cartographier les menaces

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

### Partie 2 — Repérer les points de confiance

Listez les endroits de la chaîne où un élément **croit** un autre sans le
vérifier : un fichier cru sur parole, un identifiant partagé, un réseau réputé
sûr. Une phrase par point de confiance.

### Partie 3 — La note de priorités

En dix lignes maximum, adressez-vous à la direction : quels sont les trois
risques à traiter en premier, pourquoi ceux-là, et quelle serait la première
mesure pour chacun. Votre critère de priorité doit être **dit**.

---

## ✅ Auto-évaluation

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

## Pour aller plus loin

*Facultatif.* La chaîne décrite grandit encore : un partenaire supplémentaire
déposera bientôt ses fichiers, et un tableau de bord sera ouvert aux fournisseurs.
Qu'est-ce que ces deux évolutions ajoutent à votre modélisation — et quel point de
confiance nouveau créent-elles ?
