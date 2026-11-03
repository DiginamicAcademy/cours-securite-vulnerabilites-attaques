[← Chapitre précédent](03-owasp-web.md) · [Sommaire](Readme.md) · [Chapitre suivant →](05-autres-familles.md)

# Configuration, dépendances et failles connues

> Une bibliothèque Python téléchargée dix millions de fois par mois est
> compromise. Votre application l'utilise — vous ne l'avez jamais écrite, vous ne
> l'avez peut-être jamais lue. Est-ce votre problème ? Oui : c'est exactement ce
> que décrit une **CVE** (Common Vulnerabilities and Exposures).

**Durée de lecture :** 75 min · **Prérequis :** le chapitre « Anatomie d'une
vulnérabilité »

---

## Objectifs

À l'issue de ce chapitre, vous serez capable de :

- **Repérer** une configuration de développement laissée en production, et dire ce
  qu'elle expose.
- **Décrire** le cycle de vie d'une faille connue : identifiant CVE, avis de
  sécurité, score de gravité, correctif.
- **Lire** un advisory de sécurité et en extraire les informations qui décident
  d'une action.
- **Relier** la chaîne de dépendances d'une application à sa surface d'attaque.

---

## I. Security misconfiguration — les réglages qu'on oublie

Un hôpital met en ligne son portail de rendez-vous. L'équipe garde le mode debug
du framework activé « pour voir les erreurs ». Quelques semaines plus tard, une
adresse inattendue répond : `/debug/config` affiche la configuration complète du
service — y compris les identifiants de la base de données.

Personne n'a forcé quoi que ce soit. La configuration **est** la faille : c'est la
famille **A05** — Security Misconfiguration, du Top 10 de l'**OWASP** (Open Web
Application Security Project).

### Les formes les plus fréquentes

- **Le mode debug en production** : pages d'erreur détaillées, traces de pile,
  adresses de diagnostic — autant de renseignements offerts.
- **Les réglages par défaut** : comptes d'administration livrés avec le produit,
  clés de démonstration, ports ouverts, listes de mots de passe connus.
- **Les pages inutiles** : une documentation technique, un ancien service, un
  tableau de bord oublié — chaque page oubliée est une porte de plus.
- **Les en-têtes de sécurité absents** : le serveur ne dit pas au navigateur ce
  qu'il a le droit d'exécuter ou de conserver.

> 🧪 **Atelier rapide (8 min)**
>
> Le service `vulnerable` du lab expose volontairement sa configuration. Après
> `cd lab` puis `docker compose up --build -d`, interrogez-le :
>
> ```bash
> curl http://localhost:8080/debug/config
> ```
>
> Listez les informations qui fuient : identifiants, chemins, versions. Pour
> chacune, notez en une ligne ce qu'un attaquant en ferait.

### Les défaillances cryptographiques

La famille **A02** — Cryptographic Failures, est la voisine directe : les données
sont là, mais mal protégées. On la rencontre sous des formes très concrètes :

- des mots de passe stockés en clair, ou avec un condensat faible comme
  `MD5` (Message-Digest Algorithm) sans sel ;
- des données personnelles transmises ou stockées sans chiffrement ;
- un chiffrement maison, ou un algorithme obsolète conservé « par compatibilité » ;
- des secrets — clés, jetons, mots de passe de service — écrits dans le code ou
  dans un fichier de configuration versionné.

Le point commun avec la configuration : la cause racine est un **choix par défaut
jamais revu**, pas une erreur de programmation isolée.

> [!WARNING]
> Un identifiant de base de données affiché par une page de debug n'est pas un
> détail technique : c'est l'accès à tout ce que la base contient. La gravité se
> mesure à ce qui est derrière, pas à l'élégance de la faille.

---

## II. Vulnerable components — le code qu'on n'a pas écrit

Le code d'une application moderne est majoritairement fait de **dépendances** :
bibliothèques, frameworks, images de conteneurs, briques d'infrastructure. Une
faille dans l'une d'elles est une faille de l'application — c'est la famille
**A06** — Vulnerable and Outdated Components.

### Ce qu'est une CVE

Quand une faille est découverte dans un produit utilisé ailleurs, elle reçoit un
identifiant normalisé : **CVE** (Common Vulnerabilities and Exposures), au format
`CVE-année-numéro`. Cet identifiant est le point de rendez-vous de toute la
communauté : éditeur, chercheurs, équipes de sécurité parlent de la même faille.

Autour de l'identifiant, un **advisory** — un avis de sécurité — rassemble ce
qu'il faut savoir :

| Élément | Ce qu'il apporte |
| --- | --- |
| le produit et les versions touchées | savoir si vous êtes concerné |
| le mécanisme de la faille | comprendre ce qui peut arriver |
| le score **CVSS** (Common Vulnerability Scoring System) | une gravité comparable entre les failles |
| le correctif, ou le contournement | ce qu'il faut faire, et dans quel délai |
| les références | l'annonce de l'éditeur, la base **NVD** (National Vulnerability Database), l'avis **GHSA** (GitHub Security Advisory) |

### Le cycle de vie d'une faille connue

Une faille connue vit en quatre temps :

1. **Découverte** — un chercheur, un client, ou l'éditeur lui-même trouve le
   problème. Pendant cette phase, l'information est tenue secrète pour laisser le
   temps de préparer un correctif.
2. **Publication** — l'identifiant CVE et l'advisory sortent. Tout le monde apprend
   l'existence de la faille — y compris ceux qui vont l'exploiter.
3. **Correctif** — l'éditeur publie une version corrigée. Entre la publication et
   le déploiement de cette version chez vous, la faille est ouverte.
4. **Exploitation** — des preuves de concept circulent, puis des outils
   d'attaque automatisés. Le délai entre la publication et l'exploitation massive
   se compte en heures, parfois en minutes.

C'est la **course correctif / exploitation** : le défenseur doit produire,
tester et déployer plus vite que l'attaquant ne s'adapte. Un correctif installé
mais jamais redéployé ne corrige rien.

> [!TIP]
> La question n'est pas « avons-nous des dépendances vulnérables ? » — la réponse
> est oui, tôt ou tard. C'est « savons-nous lesquelles tournent en production, et
> en combien de temps les mettons-nous à jour ? ».

### D'où viennent les alertes

- les avis des éditeurs et des projets open source ;
- les bases publiques comme **NVD** (National Vulnerability Database) ;
- les avis de sécurité comme ceux du **CERT** (Computer Emergency Response Team)
  de l'Union européenne ;
- les outils d'analyse des dépendances — **SCA** (Software Composition Analysis),
  qui comparent la liste de vos dépendances aux failles connues : `pip-audit`,
  `trivy`, ou les alertes intégrées aux forges.

---

## III. Lire un advisory réel

Savoir qu'une CVE existe ne suffit pas : encore faut-il décider **ce que cela
change pour votre système**. Un advisory se lit comme une fiche — produit,
versions, gravité, correctif — puis se traduit en avis d'exposition : êtes-vous
concerné, où, et dans quel délai agir.

L'exercice dédié [Lire un advisory et rédiger un avis
d'exposition](tp/05-lire-advisory.md) vous fait travailler sur un avis réel du
CERT de l'Union européenne, et sur le cas d'un système d'information data qui
doit décider quoi traiter en premier.

---

## IV. Aller plus loin

*Ces approfondissements sont facultatifs : ils ne sont jamais exigés pour terminer
la journée.*

- **Le registre des dépendances** (**SBOM**, Software Bill of Materials) : la
  liste formalisée de ce qui compose une application, dans un format échangeable.
  Quand une faille sort, c'est ce registre qui répond en quelques minutes à la
  question « sommes-nous concernés ? ».
- **Le durcissement de configuration** : les référentiels de configuration sûre —
  une configuration de référence, versionnée, appliquée à chaque déploiement, au
  lieu d'un serveur réglé à la main.
- **La réponse à incident côté dépendances** : qui décide, qui déploie, qui
  vérifie — la procédure se prépare avant la faille, pas pendant.

---

## Points clés

- Une configuration de développement laissée en production expose autant qu'une
  faille de code — et se corrige en quelques réglages.
- Les défaillances cryptographiques et les mauvaises configurations partagent une
  cause : un **choix par défaut jamais revu**.
- Une **CVE** identifie une faille connue ; l'advisory qui l'accompagne donne le
  produit, les versions, la gravité et le correctif.
- Le cycle découverte → publication → correctif → exploitation se joue en heures :
  la course se gagne à la **veille** et à la vitesse de déploiement.
- La question utile n'est pas « avons-nous des failles ? » mais « connaissons-nous
  nos dépendances, et en combien de temps les corrigeons-nous ? ».

---

[← Chapitre précédent](03-owasp-web.md) · [Sommaire](Readme.md) · [Chapitre suivant →](05-autres-familles.md)
