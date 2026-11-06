[← Chapitre précédent](05-autres-familles.md) · [Sommaire](Readme.md) · [Chapitre suivant →](07-modeles-attaquant.md)

# Lire du code vulnérable — la méthode des trois passes

> Un relecteur expérimenté passe vingt minutes sur un diff de quarante lignes et
> trouve une injection SQL. Comment ? Il ne lit pas le code comme un développeur —
> il le lit comme un attaquant : non pas « que fait ce programme ? », mais « que
> peut-on lui faire faire ? ».

**Durée de lecture :** 30 min (noyau exigé, atelier compris) · **Aller plus
loin :** +15 min (facultatif, pas exigé pour finir la journée) · **Prérequis :**
les chapitres « Les injections » et « Les failles web »

---

## Objectifs

À l'issue de ce chapitre, vous serez capable de :

- **Appliquer** la méthode des trois passes — entrées, sorties, appels système —
  à un extrait de code ou à un diff.
- **Repérer** les marqueurs de code qui méritent un examen attentif.
- **Parcourir** une revue de code sécurité avec une checklist de dix points.
- **Situer** l'apport des outils d'analyse automatique, et leurs limites.

---

## I. La méthode en trois passes

Lire un code inconnu en cherchant des failles, c'est chercher une aiguille dans
une botte de foin. La méthode consiste à ne pas lire tout : à suivre **trois
pistes**, dans l'ordre, et à ne s'arrêter que là où elles mènent.

### Première passe — les entrées

D'où vient ce que le programme traite ? Toute donnée qui arrive de l'extérieur —
un champ de formulaire, un paramètre d'adresse, un fichier reçu, une réponse
d'API (Application Programming Interface), une variable d'environnement — est un
point d'entrée. On les liste, sans juger encore.

### Deuxième passe — les sorties

Où va cette donnée ? Une requête envoyée à une base, une page HTML renvoyée au
navigateur, une ligne de commande passée au système, un fichier écrit sur le
disque : chaque sortie est un endroit où la donnée **agit**. C'est là que la
confusion entre données et instructions devient possible.

### Troisième passe — les appels système

Que fait le programme qui touche à la sécurité en dehors de son propre code ?
Ouvrir un fichier, lancer un processus, charger un module, vérifier un mot de
passe, décider si un utilisateur a le droit de faire quelque chose. Ces appels
sont rares et concentrés : les examiner un par un coûte peu.

> [!TIP]
> Le gain de la méthode vient de ce qu'elle **découpe** la lecture. Un diff de
> quarante lignes contient rarement plus de deux ou trois points d'entrée ; les
> suivre mène directement aux lignes qui comptent.

> 🧪 **Atelier rapide (10 min)**
>
> Le fichier `lab/vulnerable/app.py` du lab contient cinq failles volontaires,
> signalées par des commentaires dans le code :
>
```python
# FAILLE 1 — Injection SQL (A03:2021)
# Requête construite par concaténation de chaînes.
```
>
> Ouvrez le fichier et appliquez les trois passes : listez d'abord les entrées
> (les paramètres des routes), puis les sorties (les requêtes, les pages
> renvoyées), puis les appels système (le hachage des mots de passe, les
> vérifications de rôle). Combien de failles avez-vous trouvées **avant** de lire
> les commentaires ? Notez le temps passé.

---

## II. Les marqueurs à chercher

Les trois passes disent **où** regarder ; les marqueurs disent **quoi** chercher.
Ce sont des constructions de code qui, par nature, méritent un examen — pas des
fautes en soi.

```python
# marqueurs d'attention — à examiner, pas à supprimer
requete = f"SELECT nom FROM clients WHERE id = {identifiant}"  # interpolation
requete = "SELECT nom FROM clients WHERE id = {}".format(identifiant)  # format()
resultat = eval(expression)                    # exécution de code
contenu = open(chemin).read()                  # lecture de fichier
os.system("ping " + hote)                      # commande shell
pickle.loads(donnees)                          # reconstruction d'objet
db.execute("... " + valeur)                    # concaténation de requête
```

À ces marqueurs de construction s'ajoutent des marqueurs d'**absence** — plus
difficiles à voir, tout aussi importants :

- une route qui modifie des données sans vérifier de rôle ;
- une page qui insère une variable dans du HTML sans échappement ;
- un mot de passe réduit à un condensat rapide, ou stocké en clair ;
- un secret — clé, jeton, mot de passe de service — écrit dans le code ;
- une dépendance figée à une vieille version, jamais réexaminée.

> [!WARNING]
> Un marqueur n'est pas une preuve. `open()` est parfaitement légitime ; c'est
> `open()` sur un chemin choisi par l'extérieur qui pose question. Le marqueur
> dit où poser la question, pas quelle est la réponse.

---

## III. La checklist — dix points pour une revue

Pour ne rien oublier d'une revue à l'autre, dix points suffisent à parcourir :

1. Les **entrées** sont-elles toutes identifiées — y compris les fichiers et les
   réponses tierces ?
2. Chaque entrée est-elle **validée** — forme, longueur, type, valeurs admises ?
3. Les requêtes aux données sont-elles **paramétrées**, sans concaténation ?
4. Les sorties vers le navigateur sont-elles **échappées** ?
5. Les commandes système reçoivent-elles des **arguments séparés**, jamais une
   ligne assemblée ?
6. Chaque action sensible vérifie-t-elle le **rôle côté serveur** ?
7. Les actions qui modifient quelque chose prouvent-elles leur **origine** ?
8. Les mots de passe sont-ils traités par un **algorithme adapté**, avec un sel ?
9. Les **secrets** sont-ils hors du code, injectés à l'exécution ?
10. La **configuration** de production est-elle dépourvue de debug, de pages
    inutiles et de réglages par défaut ?

Cette checklist ne remplace pas l'analyse : elle la cadre. Elle se coche en
quelques minutes sur un diff, et laisse une trace de ce qui a été vérifié — utile
quand la revue est contestée, ou reprise par quelqu'un d'autre.

---

## IV. Aller plus loin

*Contenu facultatif, pas exigé pour finir la journée : ces approfondissements
ne sont jamais exigés pour terminer le cours.*

- **Les outils d'analyse automatique** — **SAST** (Static Application Security
  Testing) : `bandit` pour Python, `semgrep` pour de nombreux langages, ou les
  analyseurs intégrés aux forges. Ils passent en revue des milliers de lignes en
  quelques secondes et signalent les marqueurs connus.
- **Les limites de ces outils** : ils voient les motifs, pas les intentions. Une
  absence — un contrôle de rôle manquant, une confiance mal placée — leur échappe
  le plus souvent. Un outil qui ne signale rien n'est pas une revue qui a réussi.
- **La revue de code en équipe** : faire relire par quelqu'un qui ne connaît pas
  le code, et donner à la revue un objectif explicite — « cherchez les entrées »
  plutôt que « dites-moi si c'est bien ».

---

## Points clés

- La méthode des trois passes découpe la lecture : **entrées**, **sorties**,
  **appels système** — on suit la donnée, on ne lit pas tout.
- Les marqueurs — interpolation, concaténation, `eval`, chargement d'objet —
  disent **où** poser la question ; ils ne sont pas des preuves.
- Les marqueurs d'**absence** — contrôle de rôle, échappement, secret, mise à
  jour — se cherchent aussi, et se voient moins facilement.
- La checklist de dix points cadre la revue et en garde la trace.
- Les outils automatiques accélèrent la recherche des motifs connus ; ils ne
  remplacent pas le raisonnement sur les absences.

---

## 🧠 Vérifiez vos acquis

*Formatif, jamais noté. Répondez sans relire le chapitre, puis cochez. Les réponses sont juste après le séparateur : ce qui compte est de repérer où vous vacillez, pas de faire 5 sur 5.*

**1.** Dans la méthode des trois passes, que cherche la deuxième passe — les sorties ?

- A. D'où vient ce que le programme traite
- B. Où la donnée va, c'est-à-dire où elle **agit** : chaque sortie est un endroit où la confusion entre données et instructions devient possible
- C. Ce que le programme fait qui touche à la sécurité en dehors de son propre code
- D. Les lignes les plus longues du diff, là où les fautes se cachent

- [ ] ✓ je savais

**2.** Pourquoi la troisième passe — les appels système — coûte-t-elle peu à mener ?

- A. Parce qu'un outil la réalise automatiquement
- B. Parce que ces appels sont rares et concentrés : on peut les examiner un par un
- C. Parce qu'ils concernent uniquement les commandes passées au système
- D. Parce qu'ils sont tous signalés par un commentaire dans le code

- [ ] ✓ je savais

**3.** Vous rencontrez `contenu = open(chemin).read()` dans un diff. Que vous dit ce marqueur ?

- A. Que la ligne est une faute : il faut la remplacer
- B. Que `open()` ne doit jamais être employé avec des données externes
- C. Où poser la question : la lecture devient problématique si le chemin est choisi par l'extérieur
- D. Que la revue de cette ligne est faite, puisqu'elle est repérée

- [ ] ✓ je savais

**4.** À quoi sert la checklist de dix points, telle que le chapitre la positionne ?

- A. À remplacer l'analyse, puisqu'elle se coche en quelques minutes
- B. À lister les dix familles du Top 10 de l'OWASP
- C. À imposer l'ordre des trois passes
- D. À cadrer la revue et à laisser une trace de ce qui a été vérifié

- [ ] ✓ je savais

**5.** *Question ouverte (2-3 lignes).* Citez deux marqueurs d'**absence** du chapitre et expliquez pourquoi ils se voient moins facilement qu'une interpolation ou qu'un `eval`.

- [ ] ✓ je savais

---

### Réponses

**1. B.** Une requête à une base, une page renvoyée au navigateur, une ligne de commande, un fichier écrit : c'est là que la donnée agit.
**Pourquoi :** la première passe liste les entrées sans juger, la troisième examine les appels sensibles ; c'est aux sorties que la donnée peut devenir une instruction.

**2. B.** Ouvrir un fichier, lancer un processus, charger un module, vérifier un mot de passe, décider d'un droit : peu d'endroits, chacun décisif.
**Pourquoi :** le gain de la méthode vient de ce qu'elle découpe la lecture — un diff de quarante lignes contient rarement plus de deux ou trois points d'entrée.

**3. C.** Un marqueur n'est pas une preuve : `open()` est parfaitement légitime, c'est `open()` sur un chemin choisi par l'extérieur qui pose question.
**Pourquoi :** les marqueurs disent où chercher, pas quoi conclure — c'est ce qui les distingue d'une liste de fautes à supprimer.

**4. D.** Elle ne remplace pas l'analyse, elle la cadre — et garde la trace de ce qui a été vérifié.
**Pourquoi :** cette trace devient utile quand la revue est contestée, ou reprise par quelqu'un d'autre.

**5.** Attendu, deux parmi : une route qui modifie des données sans vérifier de rôle ; une page qui insère une variable dans du HTML sans échappement ; un mot de passe réduit à un condensat rapide ou stocké en clair ; un secret écrit dans le code ; une dépendance figée à une vieille version. Explication attendue : il n'y a rien à lire — le problème est un contrôle qui manque, pas une construction recognizable.
**Pourquoi :** c'est pour cela que la checklist consacre des points à ces absences (rôle côté serveur, échappement des sorties, algorithme de mot de passe, secrets hors du code) : elles ne sautent pas aux yeux en suivant le code.

---

[← Chapitre précédent](05-autres-familles.md) · [Sommaire](Readme.md) · [Chapitre suivant →](07-modeles-attaquant.md)
