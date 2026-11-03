[Sommaire](Readme.md) · [Chapitre suivant →](02-owasp-injections.md)

# Anatomie d'une vulnérabilité — Vie d'une faille

> La fromagerie **DIGICHEESE** (la fromagerie du fil rouge) vient
> de passer en ligne. Son développeur a écrit la recherche produits en une ligne
> de Python. Ça marche, c'est même rapide. Et c'est exactement ça, le problème :
> les failles les plus graves tiennent souvent dans les lignes les plus simples
> à écrire.

**Durée de lecture :** 45 min · **Prérequis :** le chapitre « Pourquoi sécuriser ? »

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

*Ces approfondissements sont facultatifs : ils ne sont jamais exigés pour terminer
la journée.*

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

[Sommaire](Readme.md) · [Chapitre suivant →](02-owasp-injections.md)
