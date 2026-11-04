[← Chapitre précédent](12-prevention-et-detection.md) · [Sommaire](Readme.md) · [Chapitre suivant →](A-Glossaire-et-ressources.md)

<!-- MODE TD : 🟢 découverte → ✅ correction → 📘 cours → 🔵 application — emplacement : § II « Déroulé d'un pentest » (faire évaluer la qualité d'un rapport — le cas de l'introduction, ou un rapport réel — avant de donner les trois critères : reproductible, priorisé, corrigeable) -->

# Tests d'intrusion — Une preuve, pas un exercice de hacking

> Une entreprise commande un test d'intrusion « pour cocher une case » avant un
> audit. Le prestataire envoie un rapport, que personne ne lit en entier ; les
> recommandations sont classées sans être traitées. Un an plus tard, la faille
> la plus simple de ce rapport est exploitée, et l'entreprise découvre en même
> temps qu'elle la connaissait déjà. Un test d'intrusion n'est pas un exercice
> de hacking : c'est une **preuve** de l'état de sécurité, avec un rapport — et
> sa valeur dépend entièrement de ce qu'on en fait.

**Durée de lecture :** 25 min (noyau exigé) · **Aller plus loin :** +15 min
(facultatif, pas exigé pour finir la journée) · **Prérequis :** les chapitres
« La kill chain » et « Prévention et détection »

---

## Objectifs

À l'issue de ce chapitre, vous serez capable de :

- **Décrire** le cadre d'un test d'intrusion : périmètre, règles d'engagement,
  autorisations.
- **Retracer** le déroulé type d'une mission, de la reconnaissance au rapport.
- **Distinguer** ce qu'un test d'intrusion couvre de ce qu'il ne couvre pas.
- **Évaluer** la qualité d'un rapport de test d'intrusion pour en faire un plan
  d'action.

---

## I. Périmètre, règles, engagement

Un **test d'intrusion** (*pentest*) est une simulation d'attaque, réalisée par
des professionnels mandatés, dans un cadre écrit. Ce cadre n'est pas une
formalité administrative : il est ce qui distingue un test d'un délit.

Trois éléments le composent.

Le **périmètre** (*scope*) nomme précisément ce qui peut être testé : quelles
applications, quels serveurs, quels comptes, à quelles heures. Tout ce qui n'est
pas dans le périmètre est hors limites — un serveur voisin qui tombe pendant un
test n'est pas « un dommage collatéral acceptable », c'est un incident.

Les **règles d'engagement** (*rules of engagement*) décrivent la manière : ce
qui est autorisé (un test de charge sur une fenêtre donnée), ce qui est interdit
(la divulgation de données réelles, l'ingénierie sociale sans accord), les
contacts d'urgence, et la procédure d'arrêt immédiat si un système critique est
menacé.

Les **autorisations** sont écrites, signées par la personne habilitée à les
donner. « Le directeur technique m'a dit de faire le test » ne vaut rien devant
un tribunal : c'est le document signé qui vaut.

> [!DANGER]
> Tester un système sans mandat écrit est un délit, y compris « pour aider »,
> y compris « c'était pour montrer que c'est possible ». La bonne démarche, si
> vous découvrez une faille sur un système qui ne vous appartient pas, est la
> **divulgation responsable** : signaler la faille à son propriétaire, sans
> l'exploiter.

---

## II. Déroulé d'un pentest

Une mission suit un déroulé stable, qui ressemble à s'y méprendre à la kill
chain — c'est volontaire : le testeur joue le rôle de l'attaquant pour révéler
les étapes qui ne tiennent pas.

1. **Cadrage** — réunion de lancement, périmètre, règles d'engagement,
   calendrier, contacts.
2. **Reconnaissance** — le testeur rassemble ce qui est publiquement accessible :
   adresses, technologies employées, comptes visibles. Cette phase est souvent
   la plus riche en découvertes, parce qu'elle ne touche à rien.
3. **Exploitation** — le testeur tente d'atteindre les objectifs convenus :
   accéder à une fonction réservée, extraire une donnée de démonstration,
   atteindre un système interne. Les données réelles ne sont **pas** copiées :
   on prouve l'accès, on ne le consomme pas.
4. **Analyse et rapport** — chaque découverte est documentée : le scénario, la
   preuve, l'impact métier, et la correction recommandée. Le rapport est le
   livrable — pas l'écran du testeur.
5. **Restitution** — une présentation à l'équipe, qui permet de discuter les
   corrections, leur ordre, et les points qui demandent un arbitrage.

La qualité d'un rapport se juge à trois critères : chaque finding est-il
**reproductible** (une autre équipe peut-elle le vérifier ?), est-il
**priorisé** (avec un critère explicite), et propose-t-il une **correction
actionnable** (pas « sécurisez cette fonction », mais ce qu'il faut changer) ?

> [!TIP]
> Un bon test ne se mesure pas au nombre de failles trouvées. Une mission qui
> remonte peu de découvertes, mais toutes vérifiées, priorisées et corrigeables,
> vaut mieux qu'un rapport de cinquante pages que personne ne sait traiter.

---

## III. Limites

Un test d'intrusion ne prouve pas que votre système est sûr. Il prouve que,
**dans le périmètre et le temps testés**, le testeur n'a pas trouvé mieux que ce
qui est documenté. Les limites sont structurelles :

- **la durée** : une mission dure quelques jours ; un attaquant motivé en
  dépense des mois. Ce qui n'a pas été trouvé pendant le test n'est pas
  introuvable ;
- **le périmètre** : ce qui n'est pas testé n'est pas couvert. Une application
  ajoutée après la mission n'a pas été évaluée ;
- **le moment** : le test décrit un état à une date. Une mise à jour, une
  nouvelle fonction ou un changement de configuration modifient cet état ;
- **les angles morts** : un test technique ne couvre pas l'ingénierie sociale
  (sauf accord explicite), ni la sécurité physique, ni les faiblesses de
  conception profondes qui demandent une analyse de architecture ;
- **le point de vue** : le testeur cherche à atteindre des objectifs concrets.
  Il ne remplace pas une analyse de risque, qui pose la question de ce qui
  compte pour votre activité.

Ces limites ne dévalorisent pas le test : elles disent **à quoi il sert**. Un
test d'intrusion est un instantané vérifiable, qui alimente un plan de
correction — pas un certificat de sécurité.

---

## IV. Aller plus loin

*Contenu facultatif, pas exigé pour finir la journée : ces approfondissements
ne sont jamais exigés pour terminer le cours.*

- **Choisir un prestataire de test d'intrusion** : un prestataire qui accepte
  de travailler sans périmètre écrit vous met en danger autant que lui. La
  première question à lui poser n'est pas son outil favori, c'est ce qu'il
  demande comme cadre.

- **Le bug bounty** : un programme ouvert qui rémunère les chercheurs qui
  signalent des failles, en continu plutôt que par mission. Il complète un test
  d'intrusion — il ne le remplace pas : le périmètre y est plus large, la
  méthode moins cadrée.
- **La divulgation responsable** (*coordinated vulnerability disclosure*) : le
  cadre dans lequel un chercheur signale une faille à un propriétaire — délai de
  correction négocié, publication coordonnée. C'est le prolongement naturel des
  règles d'engagement, quand la découverte est faite hors mission.
- **Les référentiels de conduite de mission** : les guides d'hygiène et les
  cadres professionnels détaillent la conduite d'un test — niveaux de
  confidentialité, gestion des preuves, restitution. Consulter celui de votre
  organisation avant de commander une mission.

---

## Points clés

- Un test d'intrusion est une **simulation encadrée** : périmètre écrit, règles
  d'engagement, autorisations signées — sans quoi il devient un délit.
- Le déroulé type suit la kill chain — cadrage, reconnaissance, exploitation,
  rapport, restitution — et le **rapport** est le livrable, pas l'exploit.
- Un bon finding est **reproductible, priorisé et corrigeable**.
- Un test prouve un état **à une date, dans un périmètre** : il ne certifie rien
  au-delà.
- La valeur d'un test se mesure au plan de correction qui en sort — pas au
  nombre de failles remontées.

---

[← Chapitre précédent](12-prevention-et-detection.md) · [Sommaire](Readme.md) · [Chapitre suivant →](A-Glossaire-et-ressources.md)
