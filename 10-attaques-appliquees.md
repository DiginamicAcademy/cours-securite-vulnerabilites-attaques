# Attaques appliquées — Trois scénarios, un même raisonnement

> Trois histoires, tirées de retours d'expérience publiés : une recherche
> produit qui vide une base de données, un commentaire qui vole une session
> d'administration, une API dont personne ne vérifie les droits. Les mécanismes
> ont été vus dans la journée « Vulnérabilités applicatives » — ici, on les
> regarde **en action**, avec la chronologie des faits, pour comprendre comment
> une ligne mal écrite devient un incident.

**Durée de lecture :** 80 min · **Prérequis :** les chapitres sur les injections, les failles web et la kill chain

---

## Objectifs

À l'issue de ce chapitre, vous serez capable de :

- **Retracer** le déroulé complet d'une attaque par injection, de la
  reconnaissance à l'extraction des données.
- **Décrire** la chaîne qui transforme un commentaire anodin en vol de session.
- **Enchaîner** plusieurs failles de faible gravité pour obtenir un accès
  complet à des données.
- **Relier** chaque étape d'un scénario d'attaque à la mesure qui l'aurait
  coupée.

---

## I. Injection SQL → vol de données

Une marketplace de matériel professionnel reçoit une plainte : un client dit
avoir vu, dans les résultats de recherche, des produits qui ne sont pas encore
publiés. L'équipe regarde les journaux, puis la base. Ce qu'elle y trouve est
plus grave que prévu.

**La chronologie des faits.**

1. **Reconnaissance.** Le champ de recherche se comporte de manière curieuse :
   une apostrophe saisie provoque une erreur technique. Pour un attaquant, une
   erreur SQL affichée est une carte : elle confirme que la saisie est
   directement mêlée à la requête.
2. **Compréhension.** En variant la saisie, il devine la structure de la
   requête — une sélection sur la table des produits, filtrée par nom.
3. **Exploitation.** Une saisie qui réécrit le filtre renvoie la totalité du
   catalogue, y compris les produits non publiés. Ce n'est encore qu'une gêne
   commerciale.
4. **Extraction.** La même porte, utilisée avec une jointure vers une autre
   table, fait sortir les adresses de courriel et les mots de passe des comptes
   clients. C'est là que l'incident commence pour de vrai.

Le mécanisme est celui vu en journée 2 : la requête est construite en collant
la saisie de l'utilisateur, si bien que cette saisie finit par être **exécutée**
au lieu d'être **comparée**. Ce qui change avec le scénario complet, c'est la
lecture des étapes : la faille technique (étape 3) n'est pas l'incident.
L'incident, c'est l'étape 4 — et c'est elle que la défense devait rendre
impossible.

Où la chaîne se coupait :

| Étape | Ce qui l'aurait coupée |
| --- | --- |
| Reconnaissance | ne pas afficher les erreurs techniques en production |
| Compréhension | une requête **paramétrée** : la saisie ne peut pas devenir une instruction |
| Exploitation | un compte de base de données aux droits minimaux, sans accès aux tables clients |
| Extraction | une alerte sur les lectures volumétriques d'une table sensible |

> [!DANGER]
> Le réflexe « on filtre les apostrophes » ne traite pas la cause racine : il
> bloque un cas, pas le mécanisme. La question reste celle de la journée
> précédente — si je corrige cela, la faille peut-elle encore être exploitée
> autrement ?

---

## II. XSS → vol de session

Un forum d'entraide entre développeurs laisse ses membres commenter les
publications. Un commentaire y dépose un message anodin, contenant une balise
de script. Quelques jours plus tard, deux modérateurs signalent des actions
qu'ils n'ont pas faites — des publications supprimées, des comptes bannis.

**La chronologie des faits.**

1. **Dépôt.** Le commentaire est enregistré tel quel, sans être nettoyé ni
   échappé. Il ne déclenche rien à l'enregistrement : c'est une attaque
   **stockée**, qui attend.
2. **Exécution.** Chaque lecteur de la page exécute le script dans son propre
   navigateur — avec les droits de ce lecteur. Le script est donc anodin pour un
   visiteur, et dangereux pour un modérateur.
3. **Collecte.** Le script envoie le **cookie de session** du lecteur vers une
   adresse contrôlée par l'attaquant. Ce cookie est la preuve de connexion : le
   posséder, c'est être connecté à la place de la personne.
4. **Usage.** L'attaquant rejoue ce cookie depuis son navigateur et accède au
   compte de modération, sans jamais connaître le mot de passe.

Le point décisif est l'étape 2 : la faille ne s'exécute pas « sur le serveur »,
elle s'exécute **dans le navigateur de la victime**, avec ses droits. C'est
pourquoi la gravité d'un même script varie selon qui lit la page — un vol de
session d'administrateur n'a rien à voir avec une simple déformation d'affichage.

Où la chaîne se coupait :

| Étape | Ce qui l'aurait coupée |
| --- | --- |
| Dépôt | échapper le HTML à l'affichage, ou n'accepter qu'une mise en forme limitée |
| Exécution | une politique de sécurité de contenu (*CSP*) qui refuse les scripts inline |
| Collecte | marquer le cookie de session `HttpOnly` : le script ne peut pas le lire |
| Usage | une expiration de session courte et une vérification de l'origine des requêtes |

> [!TIP]
> `HttpOnly` ne protège pas contre le script : il protège **ce que le script
> peut en faire**. Une mesure qui réduit l'impact vaut souvent mieux qu'une
> mesure qu'on espère totale — surtout quand le contenu vient des utilisateurs.

---

## III. Chaîne API + BDD

Un éditeur de logiciels de gestion expose une API (*application programming
interface*) interne à son application mobile. Aucune faille spectaculaire n'y
est trouvée. Et pourtant, la base de données clients sort en quelques heures.

**La chronologie des faits.**

1. **Reconnaissance.** L'API publie sa propre description — c'est pratique pour
   les développeurs, et c'est une carte pour qui veut la parcourir. L'attaquant
   y relève les routes disponibles.
2. **Contournement.** Une route d'administration répond sans vérifier le rôle
   de l'appelant : elle renvoie la liste des comptes. La faille est un contrôle
   d'accès absent, pas un bug de programmation.
3. **Appui.** Une route de diagnostic, laissée active en production, affiche la
   configuration du service — y compris les identifiants de connexion à la base
   de données.
4. **Accès direct.** Muni de ces identifiants, l'attaquant ne passe plus par
   l'API : il se connecte à la base et copie ce qu'il veut.

Cette chaîne est emblématique : **aucune des deux failles n'est critique prise
isolément**. Une liste de comptes, ce n'est pas un vol de données ; une fuite de
configuration, ce n'est pas un accès à la base. Enchaînées, elles produisent
l'accès complet. C'est exactement ce que la kill chain décrit : plusieurs
échecs doivent s'enchaîner — donc plusieurs occasions de couper.

> 🧪 **Atelier rapide (10 min)**
>
> Sur le service `vulnerable` du lab local (`docker compose up --build -d` dans
> le dossier `lab`), enchaînez deux appels pour obtenir les identifiants de base
> de données :
>
> ```bash
> curl http://localhost:8080/admin/users
> curl http://localhost:8080/debug/config
> ```
>
> Notez, pour chaque appel, la faille exploitée (contrôle d'accès absent, fuite
> de configuration) et la mesure qui l'aurait coupée. Ces notes serviront aux
> exercices de la suite.

> [!WARNING]
> Ces manipulations sont réservées au lab local, sur votre machine. Les mêmes
> gestes lancés contre un système qui ne vous appartient pas sont interdits par
> la loi — et ne vous apprennent rien de plus : c'est l'**analyse** du
> mécanisme qui est instructive, pas la cible.

---

## IV. Aller plus loin

*Ces approfondissements sont facultatifs : ils ne sont jamais exigés pour terminer
la journée.*

- **L'économie des failles** : sur les marchés clandestins, les vulnérabilités
  se vendent, les accès déjà obtenus se revendent, et les rançongiciels se
  louent en service. Comprendre cette économie explique pourquoi une petite
  faille sans valeur apparente est exploitée en masse : elle alimente un
  commerce.
- **La revente de données** : les données volées se déprécient vite, ce qui
  pousse à l'exfiltration rapide et volumétrique. C'est un argument concret en
  faveur de la détection des sorties de données anormales.
- **Les retours d'expérience publiés** : les rapports d'incident détaillés
  décrivent ces chaînes avec leurs délais. En comparer plusieurs fait apparaître
  les étapes qui reviennent toujours.

---

## Points clés

- Un scénario d'attaque se lit comme une **chronologie** : reconnaissance,
  compréhension, exploitation, puis usage de ce qui a été obtenu.
- La faille technique n'est pas l'incident : l'incident commence quand elle est
  **utilisée** pour atteindre les données ou les comptes.
- Un script exécuté dans le navigateur agit **avec les droits de la personne
  qui lit la page** — d'où l'écart de gravité selon les comptes touchés.
- Des failles de gravité moyenne, enchaînées, produisent un accès complet :
  c'est la leçon de la kill chain appliquée au web.
- Chaque étape a sa mesure de coupe — et les mesures qui réduisent l'**impact**
  (droits minimaux, `HttpOnly`, journalisation) comptent autant que celles qui
  empêchent.
