# Corrigé — Écrire une analyse menace / prévention

> Corrigé de [l'exercice « Écrire une analyse menace / prévention »](../12-analyse-menace-prevention.md).
> Comparez d'abord votre travail aux **étapes** ci-dessous : la méthode compte
> autant que le résultat. Les scénarios proposés sont un **modèle** — d'autres
> choix se défendent, à condition d'être justifiés.

---

## Partie 1 — Cartographier les menaces

**Éléments attendus :** une cartographie variée, qui ne tourne pas tous autour
du même point d'entrée. Les verbes HTTP (protocole de transfert hypertexte) sont rappelés — GET (lecture) et POST (envoi de données). Modèle de référence :

| Scénario | Étape de la kill chain | Point d'entrée |
| --- | --- | --- |
| Un script rejoue des couples identifiant / mot de passe volés ailleurs jusqu'à trouver un compte valide | accès initial | `POST /login` |
| Une période de rapport mal filtrée fait réécrire la requête de données et renvoie plus que prévu | accès initial | `GET /reports` |
| Un utilisateur connecté tape l'adresse de gestion des comptes et obtient la liste des comptes | escalade de privilèges | `GET /admin/users` |
| Une page de diagnostic, laissée active, affiche les identifiants de connexion à la base de données | accès initial (appui) | `GET /debug/config` |
| Un compte légitime exporte l'historique complet des ventes en une seule opération | exfiltration | `GET /reports/export` |
| Un script déposé dans un contenu affiché vole le cookie de session d'un responsable | accès initial | `GET /reports` |

**Ce qu'il faut retenir :** la variété est l'objectif de cet exercice. Une
cartographie où les six scénarios passent par le formulaire de connexion rate
les risques les plus coûteux — ceux qui viennent des comptes déjà légitimes et
des fonctions déjà autorisées.

**Erreurs fréquentes :** décrire des conséquences (« la plateforme est
piratée ») au lieu de scénarios. Un scénario se décrit par **qui fait quoi, par
quel bouton, et ce qu'il obtient** — sinon il n'est pas actionnable.

---

## Partie 2 — Prévenir et détecter

**Éléments attendus :**

| Scénario | Prévention | Détection |
| --- | --- | --- |
| Credential stuffing | limite de débit sur `POST /login`, MFA (authentification multi-facteurs), message d'erreur unique | série d'échecs de connexion sur un même compte, ou depuis une même adresse sur plusieurs comptes |
| Injection dans le rapport | requête paramétrée, échappement des sorties | erreur technique sur la génération de rapport, requête avec un motif inhabituel |
| Contournement de `/admin/users` | contrôle de rôle vérifié **côté serveur**, sur chaque route | accès à une route d'administration par un compte non administrateur |
| Fuite de `/debug/config` | désactiver le mode diagnostic en production, ne jamais y mettre d'identifiants | toute consultation de la route de diagnostic en production |
| Export volumétrique | plafond de volume par export, justification métier, chiffrement du fichier exporté | export dont le volume sort de l'habitude du compte |
| Vol de session | cookie `HttpOnly` et `Secure`, HTTPS, expiration courte | même session utilisée depuis deux endroits différents |

**Ce qu'il faut retenir :** la colonne « détection » est la plus souvent oubliée,
et c'est elle qui transforme une analyse en outil de travail. Une mesure de
prévention qui échoue n'est pas une défaite si le scénario devient visible —
c'est même le cas nominal d'une défense en profondeur.

**Erreurs fréquentes :** écrire « journaliser les requêtes » comme détection.
C'est une collecte, pas une détection : l'information doit être **comparée à une
habitude** ou à une règle pour devenir un signal.

---

## Partie 3 — Répondre et prioriser

**Éléments attendus** (le choix varie — c'est le critère qui est attendu) :

Deux priorités se défendent pour **Altis Data** :

- la **fuite de `/debug/config`**, si l'on juge sur l'exposition : des
  identifiants de base de données ouvrent l'accès à tout le reste, sans laisser
  de trace dans l'application ;
- l'**export volumétrique**, si l'on juge sur les données touchées : c'est le
  scénario qui met le plus de données personnelles entre de mauvaises mains, et
  le RGPD en fait un risque à traiter en priorité.

Ce qui est attendu, c'est que le critère soit **dit** : un classement sans
critère affiché ne se discute pas.

**Premiers gestes de réponse**, dans l'ordre :

1. **Conserver** — capturer les journaux de l'application, de la base et du
   réseau sur la période concernée, avant toute modification.
2. **Isoler** — révoquer les identifiants ou les sessions concernés, couper
   l'accès en cause, sans arrêter la plateforme pour les autres utilisateurs.
3. **Qualifier** — établir ce qui a été lu ou copié, depuis quand, et par quel
   compte.
4. **Informer** — la direction, l'équipe technique, et si des données
   personnelles sont concernées, l'autorité de protection des données et les
   personnes, dans les délais prévus par le RGPD.
5. **Corriger et vérifier** — traiter la cause racine (pas seulement la route
   en cause), puis confirmer que le scénario ne peut plus être rejoué.

**Ce qu'il faut retenir :** la réponse commence par **conserver**, pas par
nettoyer. L'instinct consiste à rétablir le service au plus vite ; c'est
souvent ce qui détruit les seules traces permettant de savoir ce qui a été
exfiltré — et donc de savoir quoi annoncer.

---

## Points à retenir

- Une analyse menace / prévention se lit comme un **tableau de travail** :
  scénario, étape de la chaîne, prévention, détection.
- La **variété des scénarios** compte plus que leur nombre : les comptes
  légitimes et les fonctions déjà autorisées sont des voies d'accès à part
  entière.
- La détection transforme une analyse en outil : une prévention qui échoue et
  qui se voit reste une défense.
- Une priorité n'a de valeur que si son **critère est explicite**.
- La réponse à incident suit un ordre — conserver, isoler, qualifier, informer,
  corriger — et le premier geste est la conservation des traces.
