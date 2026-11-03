# Corrigé — Lire un advisory et rédiger un avis d'exposition

> Corrigé de [l'exercice « Lire un advisory et rédiger un avis d'exposition
> »](../05-lire-advisory.md). Comparez d'abord votre travail aux **étapes**
> ci-dessous : la méthode compte autant que le résultat. Tous les chiffres
> employés ici sont ceux de l'avis reproduit dans l'énoncé.

---

## Partie 1 — Extraire les champs de l'avis

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

## Partie 2 — Lire la chronologie

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

## Partie 3 — Évaluer l'exposition du système

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

## Partie 4 — Rédiger l'avis d'exposition

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

## Points à retenir

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
