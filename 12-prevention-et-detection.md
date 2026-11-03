# Prévention et détection — Prévenir, détecter, répondre

> Une compromission passe inaperçue pendant des mois — les rapports publics sur
> les fuites de données s'accordent sur des durées moyennes de l'ordre de deux
> cents jours entre l'intrusion et sa découverte. La question n'est donc pas
> « comment ne jamais être attaqué », mais « comment s'en rendre compte vite ».
> Prévenir, détecter, répondre : les trois temps de la défense, et aucun ne
> remplace les deux autres.

**Durée de lecture :** 55 min · **Prérequis :** les chapitres « La kill chain » et « Attaques appliquées »

---

## Objectifs

À l'issue de ce chapitre, vous serez capable de :

- **Associer** à une étape de la kill chain les mesures de prévention et de
  détection qui la couvrent.
- **Choisir** les événements à journaliser pour rendre une attaque visible.
- **Distinguer** une alerte exploitable d'un bruit de journal.
- **Rédiger** les premiers gestes d'une réponse à incident.

---

## I. Prévenir

Prévenir, c'est rendre l'attaque difficile ou inintéressante. Les mesures ont
déjà été croisées au fil de la journée « Vulnérabilités applicatives » — ce qui
change ici, c'est le point de vue : on ne les classe plus par famille de
faille, mais par **étape de la chaîne d'attaque** qu'elles coupent.

| Étape visée | Mesures de prévention | Effet recherché |
| --- | --- | --- |
| Reconnaissance | réduire la surface exposée, retirer les pages et services inutiles | donner moins d'informations |
| Accès initial | mises à jour, requêtes paramétrées, échappement des sorties, authentification forte | fermer les portes d'entrée connues |
| Escalade | moindre privilège, comptes de service séparés | limiter ce qu'un accès permet |
| Mouvement latéral | segmentation, cloisonnement des environnements | empêcher de passer d'un système à l'autre |
| Persistance | contrôle des tâches planifiées et des comptes créés | rendre le retour visible |
| Exfiltration | chiffrement, limitation des volumes sortants, minimisation des données | réduire ce qui peut sortir utilement |

Deux remarques sur ce tableau.

La **minimisation des données** est la mesure la plus sous-estimée : des données
qu'on ne détient pas ne peuvent pas être volées. Une application qui conserve
des adresses de courriel dix ans après la fin de la relation client expose dix
ans de risque inutile — et le **RGPD** (Règlement général sur la protection des
données) en fait d'ailleurs une obligation.

Et la prévention a une **limite structurelle** : elle porte sur ce qu'on a
anticipé. Une faille inconnue, un usage détourné d'une fonction légitime, un
accès interne mal intentionné passent au travers. C'est pour cela que la
détection n'est pas un complément de confort.

> [!TIP]
> Une mesure de prévention se choisit par ce qu'elle coupe dans la chaîne, pas
> par sa sophistication. Un compte de base de données aux droits minimaux coupe
> l'extraction ; c'est souvent plus efficace qu'un outil supplémentaire.

---

## II. Détecter

Détecter, c'est rendre l'attaque **visible**. Trois ingrédients sont nécessaires :
des journaux qui contiennent l'information, une règle qui la repère, et quelqu'un
qui traite l'alerte.

### Ce qui mérite d'être journalisé

Un journal (*log*) utile répond à la question « qui a fait quoi, quand, et
depuis où ? ». Pour une application web, le minimum comprend :

- les **connexions** — réussies et échouées, avec l'identifiant et l'adresse
  d'origine ;
- les **changements de droits** — élévation de privilège, création de compte,
  modification de rôle ;
- les **accès aux données sensibles** — exports, lectures volumétriques ;
- les **erreurs techniques** — en particulier celles qui révèlent la structure
  interne ;
- les **changements de configuration** — activation d'un mode debug, ouverture
  d'une route.

> 🧪 **Atelier rapide (5 min)**
>
> Sur le service `vulnerable` du lab local, déclenchez une injection depuis la
> recherche, puis consultez les journaux du conteneur :
>
> ```bash
> curl "http://localhost:8080/products/search?q=' OR '1'='1"
> docker logs lab-vulnerable
> ```
>
> Observez ce qui apparaît : la requête est-elle journalisée ? La saisie
> l'est-elle ? Une erreur est-elle visible ? Notez ce qui manque pour qu'un
> analyste puisse comprendre l'événement **après coup**.

Ce que montre ce test est fréquent : la requête est souvent journalisée, la
**suspicion** rarement. Un journal qui enregistre tout sans rien signaler force
l'analyste à relire des milliers de lignes — c'est le bruit, pas la détection.

### Ce qui fait une alerte exploitable

Une alerte utile contient assez de contexte pour être **traitée sans
investigation préalable** : quoi, quand, qui, depuis où, et ce qui a été touché.
Une alerte qui dit « erreur 500 » n'est pas une alerte : c'est un symptôme.

Les signaux les plus rentables sont souvent les plus simples :

- une série d'échecs de connexion sur un même compte ;
- une connexion depuis un pays ou une plage horaire inhabituelle ;
- un volume de lecture de données hors norme ;
- l'apparition d'un compte ou d'une tâche planifiée inconnue.

> [!WARNING]
> La détection sans traitement est une fausse sécurité. Si personne ne lit les
> alertes, ou si aucune procédure ne dit quoi en faire, la journalisation ne
> réduit pas la durée de compromission — elle la documente.

---

## III. Répondre

Répondre, c'est limiter les dégâts puis rétablir un fonctionnement sûr. Les
plans d'incident détaillés sont approfondis en fin de parcours ; les premiers
gestes, eux, se connaissent dès maintenant :

1. **Conserver** — ne pas effacer les traces avant de les avoir capturées : les
   journaux, les captures, l'état des systèmes touchés.
2. **Isoler** — couper l'accès de ce qui est compromis, sans pour autant
   détruire la preuve ni arrêter un service critique sans arbitrage.
3. **Qualifier** — ce qui a été atteint, ce qui a pu être lu ou copié, depuis
   quand.
4. **Informer** — la direction, les personnes concernées, et le cas échéant
   l'autorité de protection des données : le RGPD impose une notification en cas
   de violation de données personnelles, dans un délai court.
5. **Corriger et vérifier** — traiter la cause racine, pas seulement le
   symptôme, puis confirmer que l'attaque ne peut plus être rejouée.

L'ordre compte : isoler avant de comprendre fait parfois plus de dégâts que
l'attaque elle-même, et effacer « pour nettoyer » détruit ce qui permettrait de
savoir ce qui s'est passé.

---

## IV. Aller plus loin

*Ces approfondissements sont facultatifs : ils ne sont jamais exigés pour terminer
la journée.*

- **EDR et XDR** : l'EDR (*Endpoint Detection and Response*) surveille les postes et
  serveurs (processus, fichiers, connexions) ; le XDR (*eXtended Detection and
  Response*) corrèle ces signaux avec ceux du réseau, des identités et du cloud.
  Ce sont les outils d'un centre de sécurité, le SOC (*Security Operations
  Center*) — utiles quand le volume de journaux dépasse ce qu'une équipe peut
  lire.
- **La détection comportementale** : plutôt que de chercher une signature
  connue, elle repère ce qui s'écarte d'une habitude — un compte qui lit des
  données qu'il n'a jamais lues. Plus difficile à calibrer, plus efficace contre
  les attaques inconnues.
- **Les exercices de crise** : un plan d'incident non testé se déroule rarement
  comme prévu. Un exercice court, une fois par an, révèle les points de blocage
  avant l'incident réel.

---

## Points clés

- La défense a trois temps — **prévenir, détecter, répondre** — et aucun ne
  remplace les deux autres.
- La prévention coupe des étapes connues de la chaîne ; la **détection** couvre
  ce que la prévention n'a pas anticipé.
- Un journal utile répond à « qui a fait quoi, quand, depuis où » ; une alerte
  utile contient assez de contexte pour être traitée telle quelle.
- La **minimisation des données** réduit le risque à la source : ce qu'on ne
  détient pas ne peut pas fuiter.
- Les premiers gestes de réponse suivent un ordre : conserver, isoler, qualifier,
  informer, corriger — et vérifier que la cause racine a bien été traitée.
