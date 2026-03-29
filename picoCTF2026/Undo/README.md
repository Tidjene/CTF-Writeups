# picoCTF 2026 - Undo

| Champ | Détail |
|---|---|
| **Catégorie** | General Skills |
| **Difficulté** | Easy |
| **Auteur** | YAHAYA MEDDY |

---

## Description

> Can you reverse a series of Linux text transformations to recover the original flag?

Le challenge consiste à se connecter à un serveur via `netcat`, qui applique une série de transformations sur le flag. À chaque étape, on reçoit le flag transformé accompagné d'un indice, et l'objectif est de saisir la bonne commande Linux pour annuler la transformation.

---

## Connexion

```bash
nc foggy-cliff.picoctf.net 51785
```

---

## Résolution étape par étape

### Étape 1 - Décodage Base64

**Flag reçu :**
```
KTZvNHFycnE4LWZhMDFnQHplMHNmYTRlRy1nazNnLXRhMWZlcmlyRShTR1BicHZj
```

**Indice :** `Base64 encoded the string.`

La chaîne est encodée en Base64. Pour inverser, il faut décoder. Le serveur n'accepte pas `echo "..." | base64 -d` (commande bloquée), ni `base64 -d <string>` directement. La bonne syntaxe attendue est simplement :

```bash
base64 -d
```

> Le serveur lit lui-même la valeur courante, il suffit de lui indiquer l'outil.

---

### Étape 2 - Inversion de texte (rev)

**Flag reçu :**
```
)6o4qrrq8-fa01g@ze0sfa4eG-gk3g-ta1ferirE(SGPbpvc
```

**Indice :** `Reversed the text.`

Le texte a été inversé caractère par caractère. La commande `rev` annule cette opération :

```bash
rev
```

---

### Étape 3 - Remplacement des tirets par des underscores

**Flag reçu :**
```
cvpbPGS(Eriref1at-g3kg-Ge4afs0ez@g10af-8qrrq4o6)
```

**Indice :** `Replaced underscores with dashes.`

Les underscores `_` ont été remplacés par des tirets `-`. Pour inverser, on fait le remplacement dans l'autre sens :

```bash
tr '-' '_'
```

> Attention à ne pas confondre le sens du `tr` : l'indice dit que les `_` ont été transformés en `-`, donc pour revenir en arrière on transforme les `-` en `_`.

---

### Étape 4 - Remplacement des accolades par des parenthèses

**Flag reçu :**
```
cvpbPGS(Eriref1at_g3kg_Ge4afs0ez@g10af_8qrrq4o6)
```

**Indice :** `Replaced curly braces with parentheses.`

Les accolades `{}` ont été remplacées par des parenthèses `()`. On inverse :

```bash
tr '()' '{}'
```

> Même logique : l'indice indique `tr '{}' '()'` comme transformation initiale, donc on fait l'inverse.

---

### Étape 5 - ROT13

**Flag reçu :**
```
cvpbPGS{Eriref1at_g3kg_Ge4afs0ez@g10af_8qrrq4o6}
```

**Indice :** `Applied ROT13 to letters.`

ROT13 est une substitution de 13 positions dans l'alphabet. Sa particularité : il est **sa propre inverse** (appliquer ROT13 deux fois donne le texte original). On utilise `tr` :

```bash
tr 'a-zA-Z' 'n-za-mN-ZA-M'
```

---

## Flag

```
picoCTF{Revers1ng_t3xt_Tr4nsf0rm@t10ns_8deed4b6}
```

---

## Récapitulatif des transformations (dans l'ordre inverse de résolution)

| Étape | Transformation appliquée | Commande pour annuler |
|---|---|---|
| 1 | Base64 encode | `base64 -d` |
| 2 | Inversion du texte (`rev`) | `rev` |
| 3 | `_` → `-` | `tr '-' '_'` |
| 4 | `{}` → `()` | `tr '()' '{}'` |
| 5 | ROT13 | `tr 'a-zA-Z' 'n-za-mN-ZA-M'` |

---

## Ce que j'ai appris

- Les commandes classiques de manipulation de texte sous Linux (`base64`, `rev`, `tr`) sont des outils fondamentaux en CTF.
- Lire attentivement les indices est crucial : l'indice décrit la transformation **appliquée**, pas celle à taper - il faut bien réfléchir au sens inverse.
- Certains serveurs de challenge filtrent des commandes comme `echo ... | base64 -d` pour forcer à comprendre l'interface réelle plutôt que de "tricher" en réécrivant la valeur.

---

*Writeup par [TidiX](https://github.com/Tidjene) - TidiX.Cyber*
