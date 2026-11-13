[← Chapitre précédent](01-anatomie-vulnerabilite.md) · [Sommaire](Readme.md) · [Chapitre suivant →](03-owasp-web.md)

# Les injections — quand les données deviennent des instructions

> `' OR '1'='1` — sept caractères, et la base de données renvoie tout ce qu'elle
> contient. Ce n'est pas de la magie, c'est de la grammaire : la requête envoyée
> par l'application a été **réécrite** par celui qui saisit le formulaire.

**Durée de lecture :** 40 min (noyau exigé, atelier compris) · **Aller plus
loin :** +30 min (facultatif, pas exigé pour finir la journée) · **Prérequis :**
le chapitre « Anatomie d'une vulnérabilité »

---

## Objectifs

À l'issue de ce chapitre, vous serez capable de :

- **Expliquer** ce qui distingue une donnée d'une instruction dans une requête.
- **Identifier** une injection dans un extrait de code Python, SQL ou de commande
  système.
- **Relier** chaque injection à sa cause racine : concaténation de chaînes,
  validation absente.
- **Choisir** la barrière qui coupe réellement la chaîne d'attaque, et dire
  pourquoi les autres ne suffisent pas.

---

## I. Le principe — des données qui deviennent des instructions

Une plateforme de billetterie met en ligne la recherche de ses spectacles. Le
développeur a écrit la fonctionnalité en quelques lignes, avec Python et une base
**SQL** (Structured Query Language) :

```python
# Recherche de spectacles — la requête est construite en collant la saisie
requete = f"SELECT titre, date FROM spectacles WHERE titre LIKE '%{recherche}%'"
curseur.execute(requete)
```

Tant que le visiteur tape « jazz », tout va bien. La requête envoyée à la base
ressemble à ceci :

```sql
SELECT titre, date FROM spectacles WHERE titre LIKE '%jazz%'
```

Le mot « jazz » est une **donnée** : un critère de recherche, rien d'autre. Mais
il est arrivé dans la requête par concaténation — le même chemin que les mots
`select`, `from`, `where`, qui sont des **instructions**. La base de données, elle,
ne fait pas la différence entre les deux : elle reçoit une chaîne et l'exécute
comme un programme.

Un visiteur malveillant saisit donc ceci dans le champ de recherche :

```text
' OR '1'='1
```

La requête devient :

```sql
SELECT titre, date FROM spectacles WHERE titre LIKE '%' OR '1'='1%'
```

L'apostrophe fermée la chaîne du motif de recherche trop tôt. Ce qui suit n'est
plus une donnée : c'est une **condition** ajoutée à la requête. Or
`titre like '%'` est vraie pour chaque spectacles dont le titre n'est pas vide —
le filtre a disparu, la liste complète s'affiche, y compris les représentations
non publiées.

| Nature | Rôle | Exemples dans la requête |
| --- | --- | --- |
| **Instructions** | ce que la base exécute | `select`, `from`, `where`, `like` |
| **Données** | ce que la base compare | `jazz`, un prix, une date, un nom |

Le mot « injection » vient de là : faire entrer une **instruction** par un canal
réservé aux **données**.

![Injection SQL : une donnée saisie devient une instruction](assets/injection-sql-principe.svg)

*La requête prévue ne contient qu'une donnée ; après la saisie malveillante, la base reçoit une condition qu'elle exécute.*

> [!DANGER]
> La cause racine n'est pas « l'attaquant a trouvé une astuce ». C'est la
> concaténation : dès qu'une saisie est collée dans un programme, elle peut
> devenir du code.

> 🧪 **Atelier rapide (8 min)**
>
> Dans le lab fourni, le service `vulnerable` expose une recherche produits
> volontairement fragile. Après `cd lab-digicheese` puis `docker compose up --build -d`,
> comparez deux requêtes :
>
> ```bash
> curl "http://localhost:8080/products/search?q=comte"
> curl "http://localhost:8080/products/search?q=' OR '1'='1"
> ```
>
> Notez ce qui change entre les deux réponses : combien de produits, lesquels.
> Reformulez en une phrase ce que la seconde requête a fait **écrire** à la base.

---

## II. Un même mécanisme, trois syntaxes

Le principe ne se limite pas à SQL. Partout où une application assemble du texte
puis le fait interpréter, la même confusion est possible.

### Injection SQL

La plus fréquente, la plus documentée. Elle se glisse dans une recherche, un
formulaire de connexion, un tri de tableau — partout où une saisie rejoint une
requête. Au-delà de la lecture, une injection peut permettre la modification ou
la suppression de données, selon les droits du compte utilisé par l'application.

### Injection NoSQL

Les bases **NoSQL** (Not only SQL) — MongoDB et consœurs — interrogent les données
avec des documents JSON (JavaScript Object Notation) plutôt qu'avec du texte SQL.
Le mécanisme reste le même : si l'application recopie la saisie dans la requête,
l'attaquant n'envoie plus une valeur mais un **opérateur**. L'opérateur `$ne`
(« différent de ») transforme alors une recherche de compte en « n'importe quel
client » : le premier compte de la base convient. Pas de SQL, pas d'apostrophe —
mais la même confusion entre la donnée attendue et l'instruction exécutée.

### Injection de commandes

Quand une application dialogue avec le système d'exploitation, la confusion
devient exécution de programmes.

```python
# Vérification de disponibilité d'un hôte — la saisie rejoint une commande shell
os.system("ping -c 1 " + hote)
```

Un hôte saisi comme `127.0.0.1; whoami` fait exécuter une seconde commande après
le ping. Ici, ce n'est même plus une requête qui est réécrite : c'est la ligne de
commande elle-même.

| Famille | Ce qui est interprété | Ce que l'attaquant injecte |
| --- | --- | --- |
| SQL | la requête envoyée à la base | une condition, une clause supplémentaire |
| NoSQL | le document de recherche | un opérateur (`$ne`, `$gt`, `$where`) |
| Commandes | la ligne passée au système | une seconde commande, un séparateur |

> [!WARNING]
> Les trois cas se ressemblent parce qu'ils partagent une cause racine
> identique : une chaîne de caractères assemble du code et des données, puis
> l'ensemble est interprété. Cherchez la cause, pas la syntaxe.

---

## III. Causes racines et barrières

### Ce qui fabrique une injection

Deux habitudes suffisent, et elles sont prises dans des projets pressés tous les
jours :

1. **La concaténation.** Interpoler une variable dans une chaîne — littéraux
   Python, `format()`, simple addition — puis envoyer le résultat à un
   interpréteur. C'est le point de départ de presque toutes les injections.
2. **L'absence de validation.** Une saisie qui n'est jamais examinée peut
   contenir ce que l'attaquant veut. Mais attention : valider n'est pas
   corriger, on va le voir.

### Les barrières qui ne traitent qu'un symptôme

L'équipe de la billetterie propose trois correctifs après l'incident :

- bloquer les requêtes qui contiennent une apostrophe ;
- ajouter un pare-feu applicatif qui rejette les motifs suspects ;
- afficher une page d'erreur neutre, sans détail technique.

Le premier casse la recherche de « soirée jazz à l'Olympia » et celle de
« Colette » — les clients légitimes sont les premières victimes, et l'attaquant
remplace l'apostrophe par un encodage différent. Le deuxième filtre les formes
connues ; une forme inédite passe. Le troisième masque l'erreur, pas la faille.
Ces trois mesures ont leur utilité en **défense en profondeur**, mais aucune ne
supprime la cause racine.

### Les barrières qui traitent la cause racine

- **La requête paramétrée** (aussi appelée requête préparée) : la requête est
  envoyée une fois pour toutes, avec des emplacements réservés ; les saisies
  arrivent ensuite, **à part**, et sont toujours traitées comme des données. La
  base ne peut plus les lire comme des instructions.

  ```python
  # La requête et les données voyagent séparément
  curseur.execute(
      "SELECT titre, date FROM spectacles WHERE titre LIKE ?",
      (f"%{recherche}%",),
  )
  ```

- **Les API d'échappement ou les bibliothèques qui paramètrent** : un ORM
  (Object-Relational Mapping) ou un constructeur de requêtes qui sépare les
  valeurs du texte, à condition de l'utiliser sans reconstruire la requête
  soi-même.
- **La liste autorisée** (allow-list) : pour un tri, un code pays, un identifiant
  de module, n'accepter que les valeurs connues — il n'y a alors rien à
  échapper.
- **Les commandes sans shell** : passer les arguments sous forme de liste, un par
  un, au lieu d'assembler une ligne de commande.
- **La réduction des droits** : le compte de base utilisé par l'application n'a
  pas besoin de supprimer des tables. Cela ne supprime pas l'injection, mais
  limite ce qu'elle coûte.

> [!TIP]
> La question à se poser devant chaque ligne qui assemble du texte : « ce que
> l'utilisateur tape peut-il être lu comme une instruction ? ». Si oui, la ligne
> est à reprendre — quel que soit le langage.

La mise en œuvre complète de ces correctifs — paramétrage, ORM, tests
d'injection — est traitée dans la partie « sécuriser le code » du parcours. Ici,
l'objectif est de **voir** la faille : la reconnaissance précède la réparation.

### Exercice 1 — Repérer une injection SQL dans un extrait Python/SQL (40 min)

#### Contexte

Cinq petits services viennent d'être livrés par cinq équipes différentes : une
bibliothèque municipale, une école, un studio photo, une plateforme de streaming
et une billetterie. Chacun fonctionne. Aucun n'a été passé en revue sur le plan
sécurité.

Votre mission : dire, extrait par extrait, si une injection est possible, où elle
se niche, et quel correctif la supprime.

Pour situer vos réponses, retenez que la famille **A03** — Injection, du Top 10 de
l'**OWASP** (Open Web Application Security Project), regroupe les failles où des
données sont traitées comme des instructions.

##### Extrait 1 — Bibliothèque municipale, recherche du catalogue

```python
@app.get("/catalogue")
def chercher(mot: str):
    requete = f"SELECT titre, auteur FROM livres WHERE titre LIKE '%{mot}%'"
    return db.execute(requete).fetchall()
```

##### Extrait 2 — École, consultation des notes

```python
@app.get("/notes")
def notes(eleve: str):
    return db.execute(
        "SELECT matiere, valeur FROM notes WHERE eleve = ?",
        (eleve,),
    ).fetchall()
```

##### Extrait 3 — Studio photo, archivage des séances

```python
def archiver(dossier: str):
    # le client choisit le nom du dossier de séance
    os.system("tar czf /backup/" + dossier + ".tgz /data/" + dossier)
```

##### Extrait 4 — Plateforme de streaming, recherche par courriel

```python
@app.post("/abonnes/recherche")
def rechercher(corps: dict):
    # le corps de la requête arrive en JSON, tel quel
    return db.abonnes.find_one({"email": corps["email"]})
```

##### Extrait 5 — Billetterie, version « protégée » de la recherche

```python
@app.get("/spectacles")
def chercher(recherche: str):
    if "'" in recherche or ";" in recherche:
        return {"erreur": "caractère interdit"}
    requete = f"SELECT titre FROM spectacles WHERE titre LIKE '%{recherche}%'"
    return db.execute(requete).fetchall()
```

---

#### Consignes

##### Partie 1 — Repérer

Pour chacun des cinq extraits, répondez **oui** ou **non** à la question « une
injection est-elle possible ? », et citez le motif exact qui la rend possible (la
construction de code concernée, pas le numéro de ligne).

##### Partie 2 — Expliquer le mécanisme

Pour chaque extrait jugé vulnérable, décrivez en deux phrases :

- ce que l'attaquant **envoie** (une chaîne, un opérateur, un séparateur…) ;
- ce que le programme **exécute** à la place de ce qu'il devait faire.

##### Partie 3 — Nommer le correctif

Pour chaque extrait jugé vulnérable, proposez **une** mesure qui supprime la
cause racine — pas qui masque le symptôme. Une phrase par extrait suffit.

##### Partie 4 — Juger le « correctif »

L'extrait 5 a été corrigé « à la hâte » par l'équipe de la billetterie. Ce
correctif traite-t-il la cause racine ou un symptôme ? Justifiez en deux lignes,
en vous appuyant sur la question : « si je corrige cela, la faille peut-elle
encore être exploitée autrement ? ».

---

#### ✅ Auto-évaluation

Avant de passer au corrigé, vérifiez que :

- [ ] J'ai donné un verdict **pour les cinq extraits**, y compris ceux qui me
      semblaient sains.
- [ ] Mon verdict cite le **motif de code** (concaténation, recopie d'un corps de
      requête, construction d'une commande) et pas seulement « c'est dangereux ».
- [ ] Pour chaque extrait vulnérable, j'ai décrit **ce que l'attaquant envoie** et
      **ce qui est exécuté**.
- [ ] Mes correctifs suppriment la **cause racine** : je peux dire pourquoi un
      filtre de caractères ne suffirait pas.
- [ ] J'ai justifié mon jugement sur l'extrait 5 avec la question « la faille
      peut-elle encore être exploitée autrement ? ».

---

#### Prolongement de l'exercice

*Facultatif.* Parmi les cinq services, deux manipulent des données personnelles
(courriels, notes d'élèves). Reformulez l'impact d'une injection pour ces deux-là
en particulier : que perd-on, au-delà du fonctionnement technique ?

<details>
<summary>Voir la correction</summary>

> Comparez d'abord votre travail aux **étapes** ci-dessous : la méthode compte autant que le résultat. La famille **A03** — Injection, du Top 10 de l'**OWASP** (Open Web Application Security Project), est celle où des données sont traitées comme des instructions.

#### Partie 1 — Repérer

**Éléments attendus :**

| Extrait | Verdict | Motif de code |
| --- | --- | --- |
| Bibliothèque municipale | **oui** | la saisie `mot` est interpolée dans la requête par un littéral Python |
| École | **non** | la requête est préparée, la saisie voyage dans un emplacement réservé |
| Studio photo | **oui** | la saisie `dossier` est concaténée dans une ligne de commande shell |
| Plateforme de streaming | **oui** | le corps JSON est recopié tel quel dans la requête de la base |
| Billetterie | **oui** | la saisie reste interpolée dans la requête, malgré le filtre |

**Ce qu'il faut retenir :** quatre extraits sur cinq sont vulnérables, et ils ne
se ressemblent pas. Le motif à chercher n'est pas « SQL » ni « apostrophe » : c'est
**une saisie qui rejoint un texte interprété** — requête de base, ligne de
commande, document de recherche.

**Erreurs fréquentes :** ne juger que les extraits qui parlent de base de données,
et rater le studio photo. Une ligne de commande est un programme au même titre
qu'une requête SQL.

---

#### Partie 2 — Expliquer le mécanisme

**Éléments attendus :**

| Extrait | Ce que l'attaquant envoie | Ce qui est exécuté |
| --- | --- | --- |
| Bibliothèque municipale | une chaîne qui referme le motif puis ajoute une condition | une requête où le filtre sur le titre est remplacé par une condition toujours vraie |
| Studio photo | un nom de dossier contenant un séparateur de commandes | l'archivage, puis une seconde commande lancée par le shell |
| Plateforme de streaming | un objet JSON contenant un opérateur de comparaison | une recherche « n'importe quel abonné » au lieu d'un courriel précis |
| Billetterie | une chaîne qui referme le motif, sans apostrophe ni point-virgule | la même requête réécrite que pour la bibliothèque |

**Ce qu'il faut retenir :** dans les quatre cas, la confusion est identique — la
saisie emprunte le chemin des instructions. Le studio photo montre que ce chemin
n'est pas toujours une base de données ; la plateforme de streaming montre que la
syntaxe JSON n'y change rien.

**Erreurs fréquentes :** décrire les conséquences (« on voit tous les livres »)
sans décrire le mécanisme. Ce qui est attendu, c'est la transformation : ce que
l'attaquant envoie, et ce que le programme exécute à la place.

---

#### Partie 3 — Nommer le correctif

**Éléments attendus :**

| Extrait | Correctif attendu |
| --- | --- |
| Bibliothèque municipale | requête préparée avec un emplacement réservé pour `mot` |
| Studio photo | appeler l'archivage avec une liste d'arguments, sans passer par le shell |
| Plateforme de streaming | valider le champ `email` comme une chaîne attendue, et le passer comme valeur — jamais comme fragment de requête |
| Billetterie | requête préparée, comme pour la bibliothèque |

**Ce qu'il faut retenir :** chaque correctif supprime la **concaténation** ou la
**recopie de la saisie dans un texte interprété**. C'est le même geste dans trois
langages : faire voyager la donnée à part, jamais dans le programme.

**Erreurs fréquentes :** proposer « ajouter un filtre sur les caractères
spéciaux ». Un filtre réduit l'exposition ; il laisse la concaténation en place,
donc la faille exploitable autrement.

---

#### Partie 4 — Juger le « correctif »

**Éléments attendus :** le filtre de l'extrait 5 traite un **symptôme**.

Justification attendue : la concaténation est toujours là. La question « si je
corrige cela, la faille peut-elle encore être exploitée autrement ? » reçoit la
réponse oui — un encodage différent, un caractère non filtré ou un autre canal de
saisie peut rejoindre la même requête. Le filtre ferme une porte, pas le passage.

**Ce qu'il faut retenir :** un correctif qui examine les caractères examine la
forme ; un correctif qui sépare les données des instructions examine la structure.
Seul le second rend l'attaque impossible à rejouer.

---

#### Points à retenir

- Le motif à repérer est **une saisie qui rejoint un texte interprété** : requête,
  document de recherche, ligne de commande.
- Un verdict se justifie par un **motif de code**, pas par une impression de
  danger.
- Le même mécanisme traverse SQL, NoSQL et commandes système — seule la syntaxe
  change.
- Le correctif qui supprime la cause est celui qui **sépare** la donnée de
  l'instruction ; filtrer les caractères traite un symptôme.
- Un jugement de correction n'a de valeur que s'il répond à la question « la
  faille peut-elle encore être exploitée autrement ? ».

</details>

---

## IV. Aller plus loin

*Contenu facultatif, pas exigé pour finir la journée : ces approfondissements
ne sont jamais exigés pour terminer le cours.*

- **L'exemple NoSQL complet** : la connexion qui recopie la saisie, et le corps
  de requête qui devient un opérateur.

  ```python
  # Connexion d'un client — la saisie est recopiée telle quelle dans la requête
  requete = {"email": corps_json["email"]}
  client = db.clients.find_one(requete)
  ```

  ```json
  {"email": {"$ne": ""}}
  ```

- **Les outils d'automatisation** : `sqlmap` et ses équivalents testent
  automatiquement des centaines de variantes d'injection. Ils sont utiles pour
  vérifier **son propre** lab ou une application dont on est responsable ; les
  lancer contre un système tiers est illégal et dangereux. Le lab de ce cours est
  le seul terrain de jeu prévu.
- **Le pare-feu applicatif** — **WAF** (Web Application Firewall) : il filtre le
  trafic entrant selon des motifs connus. C'est une couche utile, mais elle joue
  sur l'exposition, pas sur la cause racine — une injection qu'il ne reconnaît
  pas traverse.
- **Les injections de second ordre** : la saisie est stockée sans dommage, puis
  concaténée plus tard dans une autre requête — un export, une tâche de nuit, un
  rapport. La validation à l'entrée ne protège pas ce second usage.

---

## Points clés

- Une injection naît d'une **confusion** : une donnée est traitée comme une
  instruction, parce que les deux empruntent le même chemin.
- Le mécanisme est le même en SQL, en NoSQL et en commandes système — seule la
  syntaxe change.
- Les causes racines tiennent en deux habitudes : **concaténation** et **absence
  de validation**.
- Bloquer des motifs, filtrer ou masquer les erreurs traite des symptômes ; la
  **requête paramétrée** supprime la confusion, donc la faille.
- Réduire les droits du compte de base ne corrige pas l'injection, mais réduit
  ce qu'elle coûte.

---

## 🧠 Vérifiez vos acquis

*Formatif, jamais noté. Répondez sans relire le chapitre, puis dépliez « Vérifier ses réponses » pour vous corriger : ce qui compte est de repérer où vous vacillez, pas de faire 5 sur 5.*

**1.** Dans `SELECT ... WHERE titre LIKE '%{recherche}%'`, le mot saisi par le visiteur arrive par le même chemin que `select`, `from`, `where`. Quelle confusion cela installe-t-il ?

- A. Une donnée arrive par un canal réservé aux instructions, et la base l'exécute comme un programme
- B. Une instruction arrive par un canal réservé aux données, et la base l'affiche
- C. Une donnée arrive trop tôt dans le traitement, avant que la base n'ait pu vérifier le format attendu
- D. Une instruction est exécutée deux fois, la requête étant construite puis relue par la base

**2.** Que partagent l'injection SQL, l'injection NoSQL et l'injection de commandes (`127.0.0.1; whoami`) ?

- A. La même syntaxe : une apostrophe qui referme le texte attendu et ajoute une condition toujours vraie
- B. Le même interpréteur : dans les trois cas, c'est le moteur de la base qui lit la saisie
- C. La même cause racine : une chaîne assemble du code et des données, puis l'ensemble est interprété
- D. Le même remède : une liste autorisée de caractères, appliquée avant toute construction de chaîne

**3.** Pourquoi la requête paramétrée supprime-t-elle la faille, là où le blocage de l'apostrophe échoue ?

- A. Parce qu'elle s'exécute plus vite que la concaténation, ce qui laisse à l'attaquant moins de temps pour intervenir
- B. Parce que les saisies arrivent à part et restent traitées comme des données : la base ne peut plus les lire comme des instructions
- C. Parce qu'elle examine la saisie, repère les motifs suspects et les rejette avant qu'ils n'atteignent la base
- D. Parce qu'elle remplace la page d'erreur technique par un message neutre, qui ne renseigne plus l'attaquant

**4.** On réduit les droits du compte de base utilisé par l'application. Que gagne-t-on ?

- A. L'injection disparaît : elle ne peut plus être écrite
- B. L'injection reste, mais ce qu'elle coûte est limité
- C. L'exposition disparaît, puisque le compte est restreint
- D. La validation de la saisie devient inutile

**5.** *Question ouverte (2-3 lignes).* Le pare-feu applicatif qui rejette les motifs suspects, la page d'erreur neutre et le blocage de l'apostrophe sont trois correctifs proposés par la billetterie. En une phrase chacun, dites ce que chacun traite, puis expliquez pourquoi le chapitre les garde quand même.

---

<details>
<summary>Vérifier ses réponses</summary>

### Réponses

**1. A.** Le mot saisi est une donnée, mais il emprunte le chemin des instructions ; la base reçoit une chaîne et l'exécute comme un programme, sans faire la différence.
**Pourquoi :** c'est la définition même de l'injection donnée par le chapitre — faire entrer une instruction par un canal réservé aux données.

**2. C.** Seule la syntaxe change : une condition ajoutée en SQL, un opérateur comme `$ne` en NoSQL, une seconde commande après un séparateur en shell.
**Pourquoi :** le chapitre conclut « cherchez la cause, pas la syntaxe » — les trois cas viennent d'une chaîne qui assemble code et données.

**3. B.** La requête est envoyée une fois pour toutes avec des emplacements réservés ; les saisies voyagent séparément et ne peuvent plus devenir des instructions.
**Pourquoi :** bloquer l'apostrophe filtre une forme connue (et casse « soirée jazz à l'Olympia »), un encodage différent passe ; la requête paramétrée supprime la confusion, donc la faille.

**4. B.** Le compte de base n'a pas besoin de supprimer des tables : la mesure ne supprime pas l'injection, elle en borne les conséquences.
**Pourquoi :** c'est une barrière utile en défense en profondeur, pas une correction de la cause racine.

**5.** Attendu : le blocage de l'apostrophe et le pare-feu applicatif ne filtrent que les formes connues (et pénalisent les clients légitimes), la page d'erreur neutre empêche seulement de lire le détail technique — aucun des trois ne supprime la concaténation. Ils sont gardés parce qu'ils ont leur utilité en défense en profondeur.
**Pourquoi :** le chapitre distingue nettement ce qui traite un symptôme de ce qui traite la cause ; plusieurs barrières utiles peuvent coexister sans jamais remplacer la requête paramétrée.

</details>

---

[← Chapitre précédent](01-anatomie-vulnerabilite.md) · [Sommaire](Readme.md) · [Chapitre suivant →](03-owasp-web.md)
