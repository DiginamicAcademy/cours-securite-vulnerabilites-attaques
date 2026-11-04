# Exercice — Repérer une injection SQL dans un extrait Python/SQL

> **Nature :** exercice · **Durée :** 40 min · **Notion :** les injections — quand les données deviennent des instructions (famille A03)
> 🟡 **Intermédiaire** · **Durée :** 40 min · **Lien :** chapitre [Les injections —
> quand les données deviennent des instructions](../02-owasp-injections.md)
> **Objectif :** repérer, dans cinq extraits de code, ceux où une saisie peut
> devenir une instruction — puis nommer le correctif qui supprime la cause.

---

## Contexte

Cinq petits services viennent d'être livrés par cinq équipes différentes : une
bibliothèque municipale, une école, un studio photo, une plateforme de streaming
et une billetterie. Chacun fonctionne. Aucun n'a été passé en revue sur le plan
sécurité.

Votre mission : dire, extrait par extrait, si une injection est possible, où elle
se niche, et quel correctif la supprime.

Pour situer vos réponses, retenez que la famille **A03** — Injection, du Top 10 de
l'**OWASP** (Open Web Application Security Project), regroupe les failles où des
données sont traitées comme des instructions.

### Extrait 1 — Bibliothèque municipale, recherche du catalogue

```python
@app.get("/catalogue")
def chercher(mot: str):
    requete = f"SELECT titre, auteur FROM livres WHERE titre LIKE '%{mot}%'"
    return db.execute(requete).fetchall()
```

### Extrait 2 — École, consultation des notes

```python
@app.get("/notes")
def notes(eleve: str):
    return db.execute(
        "SELECT matiere, valeur FROM notes WHERE eleve = ?",
        (eleve,),
    ).fetchall()
```

### Extrait 3 — Studio photo, archivage des séances

```python
def archiver(dossier: str):
    # le client choisit le nom du dossier de séance
    os.system("tar czf /backup/" + dossier + ".tgz /data/" + dossier)
```

### Extrait 4 — Plateforme de streaming, recherche par courriel

```python
@app.post("/abonnes/recherche")
def rechercher(corps: dict):
    # le corps de la requête arrive en JSON, tel quel
    return db.abonnes.find_one({"email": corps["email"]})
```

### Extrait 5 — Billetterie, version « protégée » de la recherche

```python
@app.get("/spectacles")
def chercher(recherche: str):
    if "'" in recherche or ";" in recherche:
        return {"erreur": "caractère interdit"}
    requete = f"SELECT titre FROM spectacles WHERE titre LIKE '%{recherche}%'"
    return db.execute(requete).fetchall()
```

---

## Consignes

### Partie 1 — Repérer

Pour chacun des cinq extraits, répondez **oui** ou **non** à la question « une
injection est-elle possible ? », et citez le motif exact qui la rend possible (la
construction de code concernée, pas le numéro de ligne).

### Partie 2 — Expliquer le mécanisme

Pour chaque extrait jugé vulnérable, décrivez en deux phrases :

- ce que l'attaquant **envoie** (une chaîne, un opérateur, un séparateur…) ;
- ce que le programme **exécute** à la place de ce qu'il devait faire.

### Partie 3 — Nommer le correctif

Pour chaque extrait jugé vulnérable, proposez **une** mesure qui supprime la
cause racine — pas qui masque le symptôme. Une phrase par extrait suffit.

### Partie 4 — Juger le « correctif »

L'extrait 5 a été corrigé « à la hâte » par l'équipe de la billetterie. Ce
correctif traite-t-il la cause racine ou un symptôme ? Justifiez en deux lignes,
en vous appuyant sur la question : « si je corrige cela, la faille peut-elle
encore être exploitée autrement ? ».

---

## ✅ Auto-évaluation

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

## Pour aller plus loin

*Facultatif.* Parmi les cinq services, deux manipulent des données personnelles
(courriels, notes d'élèves). Reformulez l'impact d'une injection pour ces deux-là
en particulier : que perd-on, au-delà du fonctionnement technique ?
