# Corrigé — Modéliser la kill chain d'une fuite de données

> Modèle de référence pour [le défi « Modéliser la kill chain d'une fuite de
> données »](../13-defi-kill-chain-data.md). Un défi n'a pas de solution unique :
> ce document propose **un** modèle défendable et une grille de vérification,
> pas une correction pas-à-pas. Comparez votre raisonnement au sien, pas vos
> réponses mot à mot.

---

## Grille de vérification

Avant de lire le modèle, vérifiez votre travail sur ces points — ils valent plus
que la conformité au modèle :

| Ce qui est attendu | Ce qui trahit un modèle fragile |
| --- | --- |
| Une chaîne **continue** : chaque étape produit ce dont la suivante a besoin | une liste d'événements sans lien entre eux |
| Des composants **nommés** (exports, lac, planification, tableau de bord, réseau) | des étapes génériques qui pourraient décrire n'importe quelle entreprise |
| Une exfiltration **décrite** (par quoi, vers où, sous quelle apparence) | « les données ont fuité », sans mécanisme |
| Des ruptures **à chaque étape**, pas seulement à l'entrée | une seule barrière, placée devant |
| Des signaux formulés en **règle** | « surveiller les journaux » |

---

## Modèle de référence

| Étape | Ce que fait l'attaquant | Composant | Ce qu'il obtient |
| --- | --- | --- | --- |
| Reconnaissance | relève les accès exposés et les technologies employées, à partir de ce qui est publiquement visible | réseau, tableau de bord | la carte des points d'entrée |
| Accès initial | utilise un accès distant mal fermé ou un compte de service dont les identifiants ont fuité | réseau, exports | un point d'appui dans le périmètre |
| Escalade de privilèges | élève ses droits pour atteindre les traitements, en exploitant un compte de service trop privilégié | chaîne de planification | la capacité à lancer ou modifier des traitements |
| Mouvement latéral | passe du poste ou du service compromis vers le stockage de données | lac de données | un accès aux données brutes |
| Persistance | crée un compte ou une tâche planifiée qui rejouera l'accès | chaîne de planification | un retour sans nouvelle intrusion |
| Actions sur les objectifs | sélectionne les données de fidélité, parfois en copiant d'abord dans un espace intermédiaire | lac de données | les données visées, rassemblées |
| Exfiltration | sort les données par un canal qui ressemble à un flux normal — un export planifié, une sauvegarde, un transfert sortant | exports, réseau | la fuite effective |

**Ce qu'il faut retenir :** la plateforme data multiplie les **comptes de
service** et les **transferts automatisés** — deux qualités qui sont aussi les
deux meilleures alliées d'un attaquant. Un transfert de données y est un geste
normal, ce qui rend l'exfiltration difficile à distinguer d'un traitement
légitime. C'est là que la modélisation prend tout son sens : elle oblige à
demander, pour chaque étape, « qu'est-ce qui, ici, ressemble à une activité
normale ? ».

---

## Points de rupture

| Étape | Mesure | Prévention ou détection |
| --- | --- | --- |
| Reconnaissance | ne laisser aucun accès ni aucune information technique exposés inutilement | prévention |
| Accès initial | fermer les accès distants inutiles, authentification forte sur les comptes de service, rotation des identifiants | prévention |
| Escalade | droits minimaux sur les comptes de traitement : un job n'a pas besoin d'administrer la plateforme | prévention |
| Mouvement latéral | cloisonner le lac de données du reste du réseau, accès nominatifs plutôt que partagés | prévention |
| Persistance | revue périodique des comptes et des tâches planifiées, alerte sur toute création | détection |
| Actions sur les objectifs | journaliser les lectures de tables sensibles, plafonner les volumes consultés | détection |
| Exfiltration | plafonner et justifier les volumes sortants, alerter sur un transfert hors habitude | détection |

**Ce qu'il faut retenir :** les mesures de prévention tiennent dans la
conception de la plateforme — droits, cloisonnement, identifiants. Les mesures
de détection, elles, portent sur les **gestes répétés** : lecture de table,
transfert, création de tâche. Une plateforme data se protège donc autant par son
architecture que par sa surveillance.

---

## Signaux de détection

Modèle de règles, à partir des journaux disponibles :

- **journaux de planification** — « quand une tâche est créée ou modifiée hors
  d'une fenêtre de déploiement connue, alerter » ; « quand un traitement lit une
  table qu'il ne lit jamais, alerter » ;
- **journaux du lac de données** — « quand un compte lit un volume hors de son
  habitude, alerter » ; « quand un compte accède à une table de données
  personnelles pour la première fois, alerter » ;
- **journaux du tableau de bord** — « quand un compte exporte plus que son
  périmètre habituel, alerter » ;
- **journaux réseau** — « quand un transfert sortant dépasse le volume habituel
  d'une sauvegarde, alerter ».

**Sur la détection précoce :** avec les journaux existants, l'attaque devient
probablement visible au stade des **actions sur les objectifs** — les lectures
de la table de fidélité y sont traçables. Ce qui manque le plus souvent, c'est
la **comparaison à une habitude** : un journal qui enregistre les accès sans
dire lesquels sont inhabituels oblige l'analyste à tout relire. La mesure la
plus rentable est donc souvent un travail de fond — établir ce qui est normal,
pour pouvoir signaler ce qui ne l'est pas.

**Ce qu'il faut retenir :** la question « à quelle étape aurions-nous vu ? » est
plus utile que « avions-nous des journaux ? ». Elle transforme l'audit de
journalisation en évaluation de détection.

---

## Points à retenir

- Une fuite de données sur une plateforme data suit la kill chain classique, mais
  ses étapes se **dissimulent dans des gestes normaux** : transferts, exports,
  traitements planifiés.
- Les comptes de service et les automatisations sont à la fois la valeur de la
  plateforme et sa surface d'attaque : ils se protègent par des droits minimaux
  et une revue régulière.
- Chaque étape a sa rupture : la prévention tient dans la conception, la
  détection dans la surveillance des gestes répétés.
- Un signal de détection se formule en **règle** — « quand X arrive, alerter » —
  et suppose de savoir ce qui est normal.
- La question qui structure l'exercice : **à quelle étape aurions-nous vu
  l'attaque, et que manquait-il pour la voir plus tôt ?**
