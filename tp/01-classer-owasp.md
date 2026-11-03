# Exercice — Classer des scénarios dans le Top 10 de l'OWASP

> 🟢 **Débutant** · **Durée :** 25 min · **Lien :** chapitre [Anatomie d'une vulnérabilité](../01-anatomie-vulnerabilite.md)
> **Objectif :** rattacher des scénarios de la fromagerie aux dix familles de
> l'**OWASP** (Open Web Application Security Project), puis retracer l'anatomie
> de l'une de ces failles.

---

## Contexte

La fromagerie **DIGICHEESE** (le fil rouge de ce cours) vient de mettre sa
boutique en ligne. Six observations ont été remontées, par l'équipe et par des
clients. Aucune n'est encore qualifiée : c'est votre travail.

Pour vous repérer, voici les dix familles du Top 10 de l'OWASP, la classification
de référence des failles web. Vous n'avez pas à les apprendre : vous les utilisez
comme une carte.

| Code | Famille | Ce qu'elle recouvre |
| --- | --- | --- |
| **A01** | Broken Access Control | faire ce qu'on n'a pas le droit de faire |
| **A02** | Cryptographic Failures | des données mal protégées, chiffrées mal ou pas |
| **A03** | Injection | confondre les données et les instructions |
| **A04** | Insecure Design | un choix de conception dangereux, pas un bug |
| **A05** | Security Misconfiguration | réglages par défaut, debug laissé actif, pages inutiles |
| **A06** | Vulnerable and Outdated Components | une dépendance connue pour être fragile |
| **A07** | Identification and Authentication Failures | des comptes et des mots de passe mal protégés |
| **A08** | Software and Data Integrity Failures | des mises à jour ou des données jamais vérifiées |
| **A09** | Security Logging and Monitoring Failures | ne pas voir l'attaque arriver |
| **A10** | **SSRF** (Server-Side Request Forgery) | faire demander au serveur ce qu'il ne devrait pas |

### Les six observations

1. **La recherche produits.** La requête envoyée à la base est construite en
   collant ce que le visiteur tape. En saisissant `' OR '1'='1`, on obtient la
   liste complète du catalogue, y compris les produits non publiés.
2. **Les commentaires clients.** Un commentaire s'affiche tel quel sur la fiche
   produit. Un visiteur y dépose une balise de script qui s'exécute dans le
   navigateur de quiconque lit la fiche.
3. **L'espace administration.** L'adresse `/admin/users` renvoie la liste des
   comptes clients. Un client connecté qui la tape dans son navigateur l'obtient,
   sans être administrateur.
4. **La configuration.** Le serveur tourne en mode debug. L'adresse
   `/debug/config` affiche les identifiants de la base de données.
5. **La génération de factures.** Les factures sont produites par une bibliothèque
   vieille de plusieurs années, dont une faille de contournement est publiquement
   documentée.
6. **Le formulaire de connexion.** Aucune limite de tentatives, aucun blocage après
   plusieurs essais ratés.

---

## Consignes

### Partie 1 — Rattacher chaque observation à une famille

Pour chacune des six observations, donnez le **code** de la famille (A01, A03…) et
son nom. Une observation peut relever de plusieurs familles : dans ce cas, donnez
la principale et citez l'autre en une phrase.

### Partie 2 — Retracer l'anatomie de l'une de ces failles

Choisissez l'observation qui vous semble la plus grave. Retrouvez ses quatre
étapes : **cause racine**, **exposition**, **exploitation**, **impact**. Une phrase
par étape.

### Partie 3 — Cause racine ou symptôme ?

Pour l'observation 4 (le mode debug laissé actif en production), l'équipe propose
trois correctifs :

- désactiver le mode debug sur le serveur de production ;
- bloquer l'adresse `/debug/config` dans le pare-feu ;
- afficher une page d'erreur neutre à la place.

Dites, pour chacun, s'il traite la **cause racine** ou un **symptôme**, et
justifiez en une phrase.

### Partie 4 — Traiter en premier

Parmi les six observations, laquelle traitez-vous en premier ? Justifiez en deux
lignes en vous appuyant sur l'**exposition** et le **contexte métier**.

---

## ✅ Auto-évaluation

Avant de passer au corrigé, vérifiez que :

- [ ] J'ai classé **les six** observations, et pas seulement les évidentes.
- [ ] Mon classement donne le **code** de la famille (A01, A03…), pas seulement son
      nom.
- [ ] Pour l'anatomie, j'ai donné les **quatre étapes**, y compris l'impact.
- [ ] J'ai distingué, pour les trois correctifs, ce qui traite la **cause racine**
      de ce qui traite le **symptôme**.
- [ ] Mon choix de priorité s'appuie sur un **critère explicite** (exposition,
      contexte métier, personnes touchées…).

---

## Pour aller plus loin

*Facultatif.* Reprenez l'observation 2 (les commentaires clients). Que change le
fait que la faille s'exécute dans le navigateur d'un administrateur plutôt que dans
celui d'un client ? Reformulez l'impact en conséquence.
