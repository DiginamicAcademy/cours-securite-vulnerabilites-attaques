# Corrigé — Repérer une injection SQL dans un extrait Python/SQL

> Corrigé de [l'exercice « Repérer une injection SQL dans un extrait
> Python/SQL »](../02-trouver-injection.md). Comparez d'abord votre travail aux
> **étapes** ci-dessous : la méthode compte autant que le résultat. La famille
> **A03** — Injection, du Top 10 de l'**OWASP** (Open Web Application Security
> Project), est celle où des données sont traitées comme des instructions.

---

## Partie 1 — Repérer

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

## Partie 2 — Expliquer le mécanisme

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

## Partie 3 — Nommer le correctif

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

## Partie 4 — Juger le « correctif »

**Éléments attendus :** le filtre de l'extrait 5 traite un **symptôme**.

Justification attendue : la concaténation est toujours là. La question « si je
corrige cela, la faille peut-elle encore être exploitée autrement ? » reçoit la
réponse oui — un encodage différent, un caractère non filtré ou un autre canal de
saisie peut rejoindre la même requête. Le filtre ferme une porte, pas le passage.

**Ce qu'il faut retenir :** un correctif qui examine les caractères examine la
forme ; un correctif qui sépare les données des instructions examine la structure.
Seul le second rend l'attaque impossible à rejouer.

---

## Points à retenir

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
