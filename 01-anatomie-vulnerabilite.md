[Sommaire](Readme.md) · [Chapitre suivant →](02-owasp-injections.md)

# Anatomie d'une vulnérabilité — Vie d'une faille

> La fromagerie **DIGICHEESE** (la fromagerie du fil rouge) vient
> de passer en ligne. Son développeur a écrit la recherche produits en une ligne
> de Python. Ça marche, c'est même rapide. Et c'est exactement ça, le problème :
> les failles les plus graves tiennent souvent dans les lignes les plus simples
> à écrire.

**Durée de lecture :** 30 min (noyau exigé, atelier compris) · **Aller plus loin :** +10 min (facultatif, pas exigé pour finir la journée) · **Prérequis :** le chapitre « Pourquoi sécuriser ? »

---

## Objectifs

À l'issue de ce chapitre, vous serez capable de :

- **Retracer** les quatre étapes de la vie d'une faille.
- **Distinguer** une cause racine d'un symptôme.
- **Expliquer** pourquoi corriger le symptôme ne suffit pas.
- **Relier** la gravité d'une faille à son exposition et à son contexte métier.

---

## I. Anatomie d'une faille

En 2011, le service de jeux en ligne de Sony est mis hors service pendant
plusieurs semaines. L'attaque expose 77 millions de comptes : noms, adresses,
courriels, données de paiement. L'origine, selon les rapports de l'époque : une
faille d'injection SQL (Structured Query Language) — une requête construite en
collant ce que l'utilisateur tape, sans jamais le vérifier.

Une seule ligne mal écrite, et tout l'édifice tombe. Il n'y a pas de magie noire
derrière. Une faille a une **vie**, et cette vie suit toujours les mêmes quatre
étapes.

| Étape | La question qu'elle pose | Dans l'histoire de Sony |
| --- | --- | --- |
| **Cause racine** | Pourquoi la faille existe-t-elle ? | une requête écrite par concaténation |
| **Exposition** | Où est-elle atteignable, et par qui ? | une page de recherche ouverte à tous |
| **Exploitation** | Comment l'attaquant s'en sert-il ? | il fait réécrire la requête avec ses propres critères |
| **Impact** | Qu'est-ce que l'organisation y perd ? | des comptes, de la confiance, des semaines d'activité |

![La vie d'une faille : cause racine, exposition, exploitation, impact](assets/vie-dune-faille.svg)

*Chaque étape est une occasion de couper la chaîne : corriger la cause, réduire l'exposition, surveiller, préparer la réponse.*

L'intérêt de ces quatre étapes, c'est qu'elles se coupent à plusieurs endroits.
Une requête paramétrée supprime la cause racine. Un filtre en amont réduit
l'exposition. Une surveillance réduit la durée de l'exploitation. Une procédure
de crise réduit l'impact. **Aucune de ces barrières ne remplace les autres** :
c'est leur succession qui fait la sécurité.

> 🧪 **Atelier rapide (5 min)**
>
> Un formulaire d'inscription n'a pas de captcha : un script peut y créer des
> comptes par milliers. Retrouvez les quatre étapes de cette faille : cause
> racine, exposition, exploitation, impact. Une ligne par étape suffit.

---

## II. Cause racine ou symptôme ?

La fromagerie DIGICHEESE constate que sa recherche produits renvoie tout le
catalogue dès qu'on y tape une apostrophe. L'équipe est pressée. Trois
correctifs sont sur la table :

- bloquer les requêtes qui contiennent une apostrophe ;
- afficher une page d'erreur neutre, sans détail technique ;
- écrire la requête avec des paramètres, au lieu de coller les chaînes.

Le premier traite le **symptôme** : il cache un cas précis, et il en bloquera
d'autres — dont des clients qui cherchent un cœur de Comté ou l'ami du fromager.
Le deuxième traite aussi un symptôme : il empêche l'attaquant de lire l'erreur,
pas d'agir. Le troisième traite la **cause racine** : la requête ne mélange plus
les données et les instructions.

Retenez la question : **si je corrige cela, la faille peut-elle encore être
exploitée autrement ?** Si la réponse est oui, vous traitez un symptôme.

---

## III. Gravité et exposition

Toutes les failles ne se valent pas. Une faille critique sur un poste isolé, sans
réseau, peut être moins grave qu'une faille moyenne sur la page de paiement d'un
site fréquenté.

Deux notions permettent d'y voir clair :

- la **surface d'attaque** : ce qui est joignable de l'extérieur, et par qui. Une
  page ouverte à tous les visiteurs n'a pas la même exposition qu'un outil interne
  réservé à l'équipe ;
- le **contexte métier** : ce qui est derrière. Les données de paiement, les
  dossiers patients ou les comptes administrateurs n'ont pas la même valeur que le
  texte d'une page vitrine.

Pour classer les failles sans repartir de zéro à chaque fois, l'industrie s'appuie
sur des grilles de notation. La plus répandue est le **CVSS** (Common
Vulnerability Scoring System) : un score qui combine ce que l'attaquant doit faire
pour réussir et ce que l'organisation y perd. Ce score est un **point de départ**,
pas une décision : un score élevé sur un système sans valeur reste à traiter après
un score moyen sur la caisse.

Pour se parler, les professionnels regroupent aussi les failles en familles. La
classification la plus répandue est le Top 10 de l'**OWASP** (Open Web
Application Security Project) : les dix familles de failles web les plus
fréquentes, révisées régulièrement. Ce n'est pas une liste à apprendre par cœur :
c'est une **carte**, celle qui permet de dire « c'est une faille de la famille
A03 » plutôt que « c'est une faille, je ne sais pas laquelle ». Vous la
manipulerez dès l'exercice qui suit, puis famille par famille au fil de la
journée.

---

## IV. Aller plus loin

*Contenu facultatif, pas exigé pour finir la journée : ces approfondissements
ne sont jamais exigés pour terminer le cours.*

- **L'histoire du Top 10 OWASP** : publiée depuis le début des années 2000 et
  révisée plusieurs fois, la liste montre ce qui a changé dans les pratiques de
  développement — par exemple la place prise par les dépendances tierces.
- **ANSSI** (Agence nationale de la sécurité des systèmes d'information) : ses
  guides d'hygiène détaillent, sur un système réel, ce que ce chapitre résume en
  quatre étapes.
- **Les rapports publics sur l'incident de Sony** : ils décrivent la chaîne
  complète, de la faille initiale à la remise en service.

---

## Points clés

- Une faille suit quatre étapes : **cause racine, exposition, exploitation,
  impact** — et la chaîne se coupe à chacune d'elles.
- Corriger un symptôme donne l'impression d'avancer : seule la cause racine rend
  l'attaque impossible à rejouer autrement.
- La gravité dépend de **l'exposition et du contexte métier**, pas de la seule
  complexité technique.
- Les familles de vulnérabilités sont une **carte** pour se parler, pas une liste
  à réciter.

---

## 🧠 Vérifiez vos acquis

*Formatif, jamais noté. Répondez sans relire le chapitre, puis cochez. Les réponses sont juste après le séparateur : ce qui compte est de repérer où vous vacillez, pas de faire 5 sur 5.*

**1.** Un développeur remplace la concaténation par une requête écrite avec des paramètres, au lieu de coller les chaînes. Quelle étape de la vie de la faille vient-il de supprimer ?

- A. L'exposition : la page de recherche n'est plus joignable de l'extérieur
- B. La cause racine : la requête ne mélange plus les données et les instructions
- C. L'exploitation : l'attaquant ne sait plus écrire une requête
- D. L'impact : les données de paiement n'ont plus de valeur métier

- [ ] ✓ je savais

**2.** L'équipe de DIGICHEESE retient un seul correctif : afficher une page d'erreur neutre, sans détail technique. Que traite cette mesure ?

- A. La cause racine, puisque l'erreur disparaît
- B. Un symptôme : elle empêche l'attaquant de lire l'erreur, pas d'agir
- C. Le contexte métier, en retirant les données de paiement de la page
- D. La surface d'attaque, en fermant la page à tous les visiteurs

- [ ] ✓ je savais

**3.** Une faille reçoit un score **CVSS** (Common Vulnerability Scoring System) de 9,1 sur un outil interne sans donnée sensible ; une autre, score 5,4, touche la page de paiement du site. Par laquelle commencer, et pourquoi ?

- A. La 9,1 : le score combine ce que l'attaquant doit faire et ce que l'organisation y perd, il fixe donc l'ordre
- B. La 5,4 : le score est un point de départ, et le contexte métier tranche
- C. La 9,1 : un score élevé reste à traiter après un score moyen, c'est la règle
- D. Impossible de trancher : un score CVSS ne dit rien de l'exposition

- [ ] ✓ je savais

**4.** Dans le vocabulaire de ce chapitre, à quoi sert le Top 10 de l'**OWASP** (Open Web Application Security Project) ?

- A. À apprendre par cœur les dix failles à corriger dans l'ordre
- B. À chiffrer la gravité d'une faille pour un système donné
- C. À regrouper les familles de failles web les plus fréquentes, pour pouvoir dire « c'est une faille de la famille A03 »
- D. À interdire les habitudes de programmation les plus risquées

- [ ] ✓ je savais

**5.** *Question ouverte (2-3 lignes).* Le chapitre donne une question pour distinguer cause racine et symptôme : « si je corrige cela, la faille peut-elle encore être exploitée autrement ? ». Appliquez-la au blocage des requêtes contenant une apostrophe : que répond-on, et que faut-il en conclure ?

- [ ] ✓ je savais

---

### Réponses

**1. B.** Écrire la requête avec des paramètres supprime la cause racine : les données et les instructions ne passent plus par le même chemin.
**Pourquoi :** les trois autres mesures agissent ailleurs — un filtre en amont réduit l'exposition, une surveillance réduit la durée de l'exploitation, une procédure de crise réduit l'impact. Aucune ne remplace la correction de la cause.

**2. B.** La page d'erreur neutre cache un symptôme : l'attaquant ne lit plus le détail technique, mais la faille reste exploitable.
**Pourquoi :** la question du chapitre se pose ici — la faille peut encore être exploitée autrement, donc le correctif ne traitait pas la cause.

**3. B.** Un score élevé sur un système sans valeur se traite après un score moyen sur la caisse : le contexte métier et l'exposition décident.
**Pourquoi :** le CVSS est présenté comme un point de départ, pas une décision — la gravité dépend de l'exposition et du contexte métier, pas de la seule complexité technique.

**4. C.** C'est une carte pour se parler : elle nomme la famille plutôt que de laisser la faille anonyme.
**Pourquoi :** le chapitre insiste — ce n'est ni une liste à réciter ni une grille de notation ; la notation, c'est le rôle du CVSS.

**5.** Attendu : oui, la faille reste exploitable — l'apostrophe n'est qu'un cas particulier, et le blocage en punira d'autres (un client qui cherche un cœur de Comté). Conclusion attendue : on traitait un symptôme, il faut reprendre la requête elle-même avec des paramètres.
**Pourquoi :** seule la cause racine rend l'attaque impossible à rejouer autrement ; masquer ou filtrer un cas précis donne seulement l'impression d'avancer.

---

[Sommaire](Readme.md) · [Chapitre suivant →](02-owasp-injections.md)
