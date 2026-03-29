# picoCTF 2026 - bytemancy 1

| Champ | Détail |
|---|---|
| **Catégorie** | General Skills |
| **Difficulté** | Easy |
| **Auteur** | LT 'SYREAL' JONES |

---

## Description

> Can you conjure the right bytes? The program's source code can be downloaded here.

Le serveur attend un input très précis. Le code source est fourni - il suffit de le lire pour comprendre exactement ce qu'il faut envoyer.

---

## Analyse du code source

```python
while(True):
    try:
        print('⊹──────[ BYTEMANCY-1 ]──────⊹')
        print('☍⟐☉⟊☽☈⟁⧋⟡☍⟐☉⟊☽☈⟁⧋⟡☍⟐☉⟊☽☈⟁⧋⟡☍⟐')
        print()
        print('Send me ASCII DECIMAL 101 1751 times, side-by-side, no space.')
        print()
        print('☍⟐☉⟊☽☈⟁⧋⟡☍⟐☉⟊☽☈⟁⧋⟡☍⟐☉⟊☽☈⟁⧋⟡☍⟐')
        print('⊹─────────────⟡─────────────⊹')
        user_input = input('==> ')
        if user_input == "\x65"*1751:
            print(open("./flag.txt", "r").read())
            break
        else:
            print("That wasn't it. I got: " + str(user_input))
    except Exception as e:
        print(e)
        break
```

### Ce que le code attend

La condition de succès est :

```python
user_input == "\x65"*1751
```

- `\x65` est le code hexadécimal ASCII du caractère **`e`** (valeur décimale : **101**)
- `*1751` signifie que ce caractère doit être répété **1751 fois**
- Donc l'input attendu est exactement la chaîne : `eeeeeeee...e` (1751 fois)

Le serveur l'explique lui-même dans son message :
> *"Send me ASCII DECIMAL 101 1751 times, side-by-side, no space."*

| Info | Valeur |
|---|---|
| ASCII décimal demandé | `101` |
| Caractère correspondant | `e` |
| Nombre de répétitions | `1751` |
| Input attendu | `eee...e` × 1751 |

---

## Résolution

On génère la chaîne directement en Python et on la pipe vers le serveur via `netcat` :

```bash
python3 -c 'print("e"*1751)' | nc foggy-cliff.picoctf.net 50133
```

**Décomposition de la commande :**
- `python3 -c 'print("e"*1751)'` - génère la chaîne de 1751 `e` suivie d'un `\n` (newline), ce qui simule un appui sur Entrée
- `|` - pipe : redirige la sortie vers l'entrée standard du netcat
- `nc foggy-cliff.picoctf.net 50133` - se connecte au serveur du challenge

**Résultat :**
```
⊹──────[ BYTEMANCY-1 ]──────⊹
Send me ASCII DECIMAL 101 1751 times, side-by-side, no space.
==> picoCTF{h0w_m4ny_e's???_be9356c0}
```

---

## Flag

```
picoCTF{h0w_m4ny_e's???_be9356c0}
```

---

## Rappel : ASCII décimal → caractère

| Décimal | Hex | Caractère |
|---|---|---|
| 65 | `\x41` | `A` |
| 97 | `\x61` | `a` |
| **101** | **`\x65`** | **`e`** |
| 48 | `\x30` | `0` |

Pour convertir rapidement en Python :
```python
>>> chr(101)
'e'
>>> ord('e')
101
```

---

## Leçons

- **Lire le code source fourni** est toujours la première étape - ici la solution est littéralement dans le code (`"\x65"*1751`).
- `\x65` est la notation hexadécimale d'un caractère ASCII. Connaître la table ASCII (ou savoir utiliser `chr()` / `ord()` en Python) est un réflexe utile en CTF.
- **Piper Python vers netcat** (`python3 -c '...' | nc`) est une technique simple et puissante pour automatiser des inputs réseau sans écrire un script complet.

---

*Writeup par [TidiX](https://github.com/Tidjene) - TidiX.Cyber*
