[← Chapitre précédent](10-attaques-appliquees.md) · [Sommaire](Readme.md) · [Chapitre suivant →](12-prevention-et-detection.md)

# Attaques d'authentification — Le premier rempart, et ses failles

> En 2012, la plateforme professionnelle LinkedIn est victime d'un vol de
> données : des dizaines de millions de mots de passe sont emportés. Des années
> plus tard, des milliers de comptes tombent sur d'autres services — sans que
> ces services aient été attaqués. La cause : les mots de passe volés ont été
> **réutilisés** ailleurs. L'authentification est le premier rempart d'une
> application ; ses faiblesses sont souvent humaines, et presque toujours
> exploitables à grande échelle.

**Durée de lecture :** 30 min (noyau exigé, atelier compris) · **Aller plus
loin :** +15 min (facultatif, pas exigé pour finir la journée) · **Prérequis :**
les chapitres « Modèles d'attaquant » et « Attaques appliquées »

---

## Objectifs

À l'issue de ce chapitre, vous serez capable de :

- **Distinguer** force brute, credential stuffing et hameçonnage par leur
  mécanisme et par leur trace.
- **Décrire** le détournement de session et les conditions qui le rendent
  possible.
- **Repérer**, sur un service donné, les mesures qui limitent les tentatives de
  connexion.
- **Relier** une mesure de protection (limite de débit, MFA, expiration de
  session) à l'attaque qu'elle neutralise.

---

## I. Credential stuffing, brute force

Deux attaques visent le même formulaire de connexion, mais elles ne partent pas
du même endroit.

La **force brute** (*brute force*) consiste à essayer des mots de passe un par
un, jusqu'à tomber juste. Elle part de zéro : elle ne sait rien de la victime.
Face à un mot de passe long et unique, elle est hors d'atteinte — le nombre de
combinaisons à essayer devient astronomique. Face à un mot de passe court ou
courant, elle finit par trouver.

Le **credential stuffing** (« bourrage d'identifiants ») part, lui, de quelque
chose : des couples identifiant / mot de passe déjà volés ailleurs. Comme les
personnes réutilisent leurs mots de passe, il suffit de rejouer la liste sur
des centaines de services pour en trouver où elle fonctionne. C'est une
attaque de **masse**, automatisée, qui ne vous vise pas spécialement : votre
service est simplement dans la liste.

| | Force brute | Credential stuffing |
| --- | --- | --- |
| **Point de départ** | aucune information | des identifiants déjà volés |
| **Ce qui est essayé** | des mots de passe devinés | les mots de passe réels des personnes |
| **Volume** | beaucoup d'essais par compte | peu d'essais, beaucoup de comptes |
| **Ce qui la stoppe** | un mot de passe long et unique | une limite de débit, la MFA (authentification multi-facteurs), un mot de passe unique par service |

Le mot « stuffing » vient de l'image : on **remplit** le formulaire avec une
liste volée. La trace caractéristique est donc l'inverse de celle de la force
brute : peu d'échecs par compte, mais des échecs sur des milliers de comptes,
depuis des adresses variées.

> 🧪 **Atelier rapide (5 min)**
>
> Sur le service `vulnerable` du lab local, envoyez plusieurs tentatives de
> connexion à la suite — la requête utilise le verbe POST (envoi de données) —
> et observez la réponse :
>
> ```bash
> curl -X POST -F "username=alice" -F "password=incorrect" http://localhost:8080/login
> ```
>
> Répétez la requête six fois. Le service finit-il par ralentir, bloquer ou
> demander une vérification supplémentaire ? Notez ce que vous observez : c'est
> la notion de **limite de débit** (*rate limit*), et son absence est une
> invitation à l'automatisation.

Ce que révèle ce petit test : la question n'est pas « le mot de passe est-il
solide ? » mais « que se passe-t-il quand quelqu'un essaie dix mille fois ? ».
Une application qui ne limite pas les tentatives rend inutile la complexité des
mots de passe — l'attaquant a le temps de lui.

> [!WARNING]
> Un message d'erreur trop précis est aussi une fuite : « mot de passe
> incorrect » et « compte inconnu » permettent de deviner quels identifiants
> existent. Un message unique — « identifiants incorrects » — ne renseigne pas.

---

## II. Phishing, session hijacking

Deux autres attaques ne cherchent pas à deviner le mot de passe : elles
cherchent à l'obtenir, ou à s'en passer.

**L'hameçonnage** (*phishing*) consiste à présenter une page de connexion
identique à la vraie, sur une adresse presque correcte, et à laisser la
personne y saisir elle-même ses identifiants. Aucune faille technique n'est
exploitée : la personne donne ce qu'on lui demande, parce que la page est
convaincante. La variante ciblée — un message personnalisé, qui cite un
collègue, un projet réel — s'appelle du *spear phishing* : elle vise une
personne précise, et son taux de réussite est bien supérieur.

Le **détournement de session** (*session hijacking*) part du même constat : une
fois connecté, on n'a plus besoin du mot de passe. La preuve de connexion est un
**cookie de session** (ou un jeton, *token*) émis par le serveur ; quiconque
possède ce jeton est traité comme la personne. Il peut être volé par un script
malveillant injecté dans une page, capturé sur un réseau sans chiffrement, ou
simplement lu sur un poste laissé ouvert.

La chronologie tient en un fil : la personne se connecte et le serveur émet
un cookie de session ; le cookie est intercepté ou copié ; l'attaquant le
rejoue et le serveur le prend pour la personne ; la session dure tant que le
cookie n'est ni expiré ni révoqué.

> ⚠️ **À ARBITRER** — suppression envisagée : le développement sur le
> **détournement de session** (§ II), déjà illustré par le scénario XSS → vol
> de session du chapitre 10. Le tableau de protection se réduirait alors à la
> ligne « Hameçonnage / MFA ».

Ce qui protège, à chaque étape :

| Étape | Mesure |
| --- | --- |
| Hameçonnage | une seconde preuve (**MFA**, *multi-factor authentication*), qui ne se vole pas par un simple formulaire |
| Vol du cookie | marquages `HttpOnly` et `Secure`, transport en HTTPS exclusivement |
| Rejeu | une durée de session courte, une révocation possible, une liaison à l'appareil |
| Persistance | une déconnexion active et une expiration côté serveur, pas seulement côté navigateur |

> [!TIP]
> La MFA ne rend pas l'hameçonnage impossible : elle rend le vol d'un mot de
> passe **insuffisant**. C'est déjà beaucoup — la majorité des attaques de masse
> s'arrêtent là, parce qu'elles reposent sur la réutilisation et l'automatisation.

---

## IV. Aller plus loin

*Contenu facultatif, pas exigé pour finir la journée : ces approfondissements
ne sont jamais exigés pour terminer le cours.*

- **Les recommandations du NIST** (*National Institute of Standards and
  Technology*) sur la gestion des identifiants : les rotations obligatoires de
  mots de passe y sont abandonnées au profit de la détection des mots de passe
  compromis, jugées plus efficaces. Un document qui a changé les pratiques
  d'entreprise.
- **Les gestionnaires de mots de passe** : ils rendent le mot de passe unique
  par service praticable — la mesure la plus simple contre le credential
  stuffing, à condition d'être adoptée.
- **Les jetons à durée de vie courte** : les architectures modernes émettent des
  jetons valables quelques minutes, renouvelés en arrière-plan. Le vol d'un
  jeton devient alors une fenêtre étroite plutôt qu'une porte ouverte.

---

## Points clés

- La force brute cherche un mot de passe ; le **credential stuffing** rejoue des
  mots de passe déjà volés — leurs traces ne se ressemblent pas.
- Une application sans **limite de débit** rend inutile la complexité des mots
  de passe : c'est le nombre d'essais qui compte.
- L'hameçonnage n'exploite pas une faille technique : il exploite la
  **confiance**, d'où l'intérêt d'une seconde preuve (MFA).
- Une session est une **preuve** : la protéger (cookie `HttpOnly`, HTTPS,
  expiration, révocation) compte autant que protéger le mot de passe.
- Les mesures les plus rentables sont celles qui cassent l'automatisation :
  limite de débit, MFA, mot de passe unique par service.

---

[← Chapitre précédent](10-attaques-appliquees.md) · [Sommaire](Readme.md) · [Chapitre suivant →](12-prevention-et-detection.md)
