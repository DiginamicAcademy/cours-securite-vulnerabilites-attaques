# Corrigé — Lister les vulnérabilités et leur gravité sur un extrait de code

> Corrigé de [l'exercice « Lister les vulnérabilités et leur gravité sur un
> extrait de code »](../03-revue-de-code.md). Comparez d'abord votre travail aux
> **étapes** ci-dessous : la méthode compte autant que le résultat. Les familles
> citées sont celles du Top 10 de l'**OWASP** (Open Web Application Security
> Project).

---

## Partie 1 — Inventorier les constats

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

## Partie 2 — Évaluer la gravité

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

## Partie 3 — Ordonner les corrections

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

## Partie 4 — Les limites de la revue

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

## Points à retenir

- Une revue de code se mène en **inventaire** : chaque constat porte un endroit,
  une famille et une cause racine.
- Les constats les plus faciles à rater sont les **absences** — contrôle de rôle,
  jeton, échappement — pas les lignes visiblement dangereuses.
- La gravité se juge dans le **contexte** : exposition et données derrière, pas
  sophistication technique.
- L'ordre de correction affiche son **critère** ; les mesures retenues traitent la
  cause racine, pas le symptôme.
- La revue de code a des **limites** : les nommer fait partie du travail.
