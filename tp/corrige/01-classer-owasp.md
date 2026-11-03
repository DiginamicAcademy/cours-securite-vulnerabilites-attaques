# Corrigé — Classer des scénarios dans le Top 10 de l'OWASP

> Corrigé de [l'exercice « Classer des scénarios dans le Top 10 de l'OWASP »](../01-classer-owasp.md).
> Comparez d'abord votre travail aux **étapes** ci-dessous : la méthode compte
> autant que le résultat. Les familles citées sont celles du Top 10 de
> l'**OWASP** (Open Web Application Security Project).

---

## Partie 1 — Rattacher chaque observation à une famille

**Éléments attendus :**

| Observation | Famille | Pourquoi |
| --- | --- | --- |
| La recherche produits | **A03** — Injection | la saisie est traitée comme une instruction |
| Les commentaires clients | **A03** — Injection | même confusion, dans le navigateur : le texte saisi devient du code |
| L'espace administration | **A01** — Broken Access Control | l'accès n'est pas vérifié côté serveur |
| La configuration | **A05** — Security Misconfiguration | un réglage de développement laissé actif en production |
| La génération de factures | **A06** — Vulnerable and Outdated Components | une dépendance connue pour être fragile, jamais mise à jour |
| Le formulaire de connexion | **A07** — Identification and Authentication Failures | rien ne protège le compte au-delà du mot de passe |

**Ce qu'il faut retenir :** deux observations relèvent de la même famille. La
classification ne se lit pas comme une étiquette par faille : une seule famille
peut couvrir des mécanismes très différents. Ici, la confusion entre les données et
les instructions se retrouve dans la recherche comme dans les commentaires — la
seconde s'exécutant dans le navigateur plutôt que dans la base.

**Erreurs fréquentes :** classer les commentaires clients dans une famille à part,
parce que « ça ne touche pas la base de données ». Ce qui compte, c'est le
mécanisme — des données confondues avec des instructions — pas la syntaxe.

---

## Partie 2 — Retracer l'anatomie de l'une de ces failles

**Éléments attendus** (exemple avec la recherche produits — toute autre observation
est acceptable, à condition de remplir les quatre étapes) :

| Étape | Ce qui est attendu |
| --- | --- |
| **Cause racine** | la requête est construite en collant la saisie de l'utilisateur, sans séparer les données des instructions |
| **Exposition** | la recherche est ouverte à tous les visiteurs, sans authentification |
| **Exploitation** | l'attaquant saisit une chaîne qui fait réécrire la requête, et obtient la liste complète |
| **Impact** | des produits non publiés deviennent visibles, et la même porte ouvre sur ce que la base contient |

**Ce qu'il faut retenir :** beaucoup décrivent les conséquences (« le catalogue
fuite ») sans remonter à la cause racine. L'anatomie n'est complète que si les
quatre étapes sont là — et l'étape la plus souvent oubliée est la cause racine.

---

## Partie 3 — Cause racine ou symptôme ?

**Éléments attendus :**

| Correctif | Verdict | Pourquoi |
| --- | --- | --- |
| désactiver le mode debug sur le serveur de production | **cause racine** | la faille disparaît : plus rien n'est exposé par ce canal |
| bloquer l'adresse `/debug/config` dans le pare-feu | **symptôme** | une porte est fermée, mais le mode debug reste actif et peut en ouvrir d'autres |
| afficher une page d'erreur neutre à la place | **symptôme** | on masque l'affichage, pas la configuration |

**Ce qu'il faut retenir :** la question à se poser est « si je corrige cela, la
même configuration peut-elle encore exposer autre chose ? ». Si la réponse est oui,
vous traitez un symptôme.

---

## Partie 4 — Traiter en premier

**Éléments attendus** (classement indicatif — l'important est le critère explicite) :

Pour **DIGICHEESE** (la fromagerie du fil rouge), deux réponses se défendent :

- **la configuration** en premier, si l'on juge sur l'exposition : un identifiant
  de base de données ouvre tout le reste, et l'adresse est accessible sans
  authentification ;
- **la recherche produits** en premier, si l'on juge sur le nombre de personnes
  touchées : c'est la page que tout le monde utilise.

Ce qui est attendu, c'est que le critère soit **dit** : un classement sans critère
affiché ne se discute pas.

---

## Points à retenir

- Classer, c'est donner un **nom commun** à un problème : le code de famille suffit
  à se comprendre entre collègues.
- Une même famille peut couvrir des mécanismes très différents — la confusion entre
  données et instructions se retrouve dans la recherche comme dans les commentaires.
- L'anatomie d'une faille se décrit en quatre étapes, et se coupe à chacune d'elles.
- Un correctif qui traite un symptôme donne l'impression d'avancer : seule la cause
  racine rend l'attaque impossible à rejouer autrement.
- Un classement de priorité n'a de valeur que si son **critère est explicite**.
