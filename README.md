# 🔐 RSA Single Prime Challenge

## 📋 Informations

| Catégorie | Difficulté | Flag Format | Vulnérabilité |
| :--- | :--- | :--- | :--- |
| **Cryptographie** | 🟡 Medium | `IGTF{...}` | Single Prime & Even Exponent |

## ☎️ L'énnocé
Mon professeur de crypto a dit qu'il fallait deux nombres premiers pour RSA. J'ai décidé d'économiser de l'énergie et d'en utiliser qu'un seul, mais très grand, donc très costaud ! J'ai aussi doublé la sécurité de l'exposant classique en passant à 32.

Pouvez-vous retrouver mon secret ?

Fichiers fournis : `challenge_data.txt` (contenant le N, e, et ct que tu as généré ci-dessus).

## 📝 Le Challenge

L'auteur de ce challenge a voulu "optimiser" le chiffrement RSA en commettant deux erreurs d'implémentation critiques :
1.  Il n'utilise qu'un **seul nombre premier** $N$ au lieu de deux ($N = p$ au lieu de $N = p \times q$).
2.  Il utilise un **exposant pair** $e=32$ (soit $2^5$).

**Objectif :** Retrouver le message original (le flag) sans connaître la clé privée (qui n'existe mathématiquement pas dans cette configuration).

### 📂 Fichier fournis
* `challenge_data.txt` : Les valeurs brutes de $N$, $e$ et $ct$.

---

## 🛠️ Analyse et Solution Théorique

Pour comprendre comment casser ce chiffrement, il faut analyser pourquoi les règles standards de RSA ne s'appliquent pas.

### 1. Le "Terrain de Jeu" : Pourquoi $N$ Premier change tout ?

> **L'Analogie de la Pièce Cachée** 🏠
> * **RSA Normal ($N = p \times q$) :** La "pièce" est un labyrinthe complexe construit en mélangeant deux plans d'architecte ($p$ et $q$). Si vous n'avez pas les plans (la factorisation), vous êtes perdu.
> * **Ce Challenge ($N = p$) :** L'auteur a oublié de mélanger. La "pièce" est un grand espace ouvert (un corps fini $\mathbb{Z}_p$).

**Conséquence mathématique :**
Dans cet espace ouvert, les règles sont simples. Calculer des racines carrées (ce qui est très difficile dans RSA normal sans la clé) devient possible et efficace grâce à des algorithmes comme **Tonelli-Shanks**.

### 2. Le Problème de la "Porte à Tambour" ($e=32$)

Pourquoi ne peut-on pas simplement calculer l'inverse de la clé ($d$) comme d'habitude ?

> **L'Analogie de la Porte** 🚪
> * **Chiffrer ($e$) :** Vous poussez une porte à tambour de $X$ tours.
> * **Déchiffrer ($d$) :** Vous pousses encore pour revenir exactement au point de départ.
> * **Le Bug :** Ici, la taille du tour est paire (car $N-1$ est pair) et votre poussée ($e=32$) est paire. C'est comme si la porte avait fait des tours complets sur elle-même. En la regardant fermée, impossible de savoir si elle a fait 2, 4 ou 32 tours. L'information s'est superposée.

**Conséquence :** Il n'y a pas de "bouton retour" unique. La fonction n'est pas bijective.

### 3. La Résolution : Remonter l'Arbre

Puisque nous ne pouvons pas inverser le calcul d'un coup, nous devons le faire pas à pas.
L'équation du chiffrement est :
$$C \equiv M^{32} \pmod N$$

Comme $32 = 2^5$, cela revient à dire que le message a été mis au carré **5 fois de suite**.
Pour retrouver le message, nous devons extraire la racine carrée 5 fois de suite.

**Le piège des racines :**
En maths modulaires, une racine carrée possède souvent **deux solutions** ($x$ et $-x$). Cela crée un arbre de décision :

* **Départ :** 1 message chiffré.
* **Étape 1 ($\sqrt{}$) :** 2 possibilités.
* **Étape 2 ($\sqrt{}$) :** 4 possibilités.
* ...
* **Étape 5 ($\sqrt{}$) :** Jusqu'à 32 messages potentiels.

L'un de ces 32 messages est le flag `IGTF{...}`. Les autres ne sont que du bruit mathématique.

---

## 💻 Utilisation du Script

Le script `solve.py` automatise cette attaque en utilisant l'algorithme de Tonelli-Shanks.

### Fonctionnement du script
1.  Il charge le chiffré $ct$.
2.  Il boucle 5 fois pour extraire les racines carrées successives.
3.  Il gère l'arbre de décision (les multiples candidats générés à chaque étape).
4.  Il convertit les résultats en texte et cherche le format de flag `IGTF{`.

### Pré-requis
Aucune installation complexe n'est requise, le script utilise Python natif (3.x).

### Commande
Ouvrez un terminal dans le dossier et lancez :

```bash
python solve.py
