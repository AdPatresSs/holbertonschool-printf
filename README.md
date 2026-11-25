# _printf

« Allez, c'est facile, printf c'est juste de l'affichage, on règle ça en dix minutes ! » — Phrase prononcée juste avant de se battre avec va_arg et le style Betty.

Bon finalement, au démarrage, le flou était total : difficile de savoir par quel bout attaquer un tel morceau. Alors on a fait un pas en arrière, posé les bases et pris le temps de comprendre exactement ce qu'on nous demandait de reproduire avant de toucher au reste.

En gros, `printf` (pour *print formatted*), c'est la fonction de `<stdio.h>` qui sert à prendre des infos stockées en mémoire et à les afficher proprement sur la sortie standard (`stdout`, souvent le terminal). Pour se faire une image simple, ça marche un peu comme une phrase à trous : on donne d'abord une chaîne modèle avec des emplacements marqués par `%` (comme `%s` pour du texte, `%d` pour un entier ou `%c` pour un caractère), puis on passe les vraies valeurs juste derrière dans le même ordre. `printf` lit la chaîne, remplace les `%` par les valeurs converties en texte, interprète les séquences spéciales comme `\n` (retour à la ligne), et renvoie à la fin le nombre total de caractères écrits (ou une valeur négative en cas d'erreur).

Dans son champ d'action complet, `printf` peut afficher du texte brut, traduire différents types (caractères, chaînes, entiers en base 10/8/16, flottants, pointeurs, ou un simple `%` avec `%%`) et mettre en forme l'affichage (alignement, espaces, zéros, signes, précision). En revanche, il ne lit pas ce qu'on tape au clavier, il n'écrit pas directement dans un fichier ciblé par défaut (`stdout` uniquement) et il ne corrige pas à notre place si on se trompe dans l'ordre ou le type des arguments passés.


## Fonctionnalités supportées (Tâche 0)

- Affichage des caractères normaux de la chaîne `format`.
- `%c` : affiche un caractère (`char`).
- `%s` : affiche une chaîne de caractères (`char *`), ou `(null)` si le pointeur est `NULL`.
- `%%` : affiche un caractère `%`.
- Retourne le nombre total de caractères écrits sur la sortie standard (`stdout`), ou `-1` en cas d'erreur (`format == NULL`).

## Compilation

Les fichiers sources (sans `main.c` à la racine) sont compilés avec : "gcc -Wall -Werror -Wextra -pedantic -std=gnu89 -Wno-format *.c"

## Contraintes et éléments autorisés

Seules les fonctions et macros suivantes sont autorisées dans le projet :
- `write` (`man 2 write`)
- `malloc` (`man 3 malloc`)
- `free` (`man 3 free`)
- `va_start` (`man 3 va_start`)
- `va_end` (`man 3 va_end`)
- `va_copy` (`man 3 va_copy`)
- `va_arg` (`man 3 va_arg`)

### Implications techniques de ces contraintes

La restriction aux seuls appels système/macros de base (`write`, `malloc`, `free` et `va_*`) implique plusieurs choix d'architecture :

- **Réécriture complète des utilitaires :** aucune fonction de la bibliothèque standard (`strlen`, `putchar`, `itoa`, etc.) n'est utilisée. Le calcul de la longueur des chaînes et les conversions numériques en différentes bases sont implémentés manuellement.
- **Écriture directe et suivi strict du compteur :** chaque appel à `write(1, ...)` renvoie le nombre d'octets réellement écrits ; chaque fonction d'affichage remonte ce total afin que `_printf` retourne exactement le nombre de caractères imprimés (ou `-1` en cas d'erreur).
- **Gestion rigoureuse des arguments variadiques (`va_*`) :** les promotions implicites du C (`char` promu en `int` dans `va_arg`) et la libération systématique via `va_end` (y compris sur les chemins d'erreur) sont respectées.
- **Sécurité mémoire et cas limites :** vérification systématique des pointeurs `NULL` (ex. `format == NULL` ou chaîne `%s` égale à `NULL` affichant `(null)`), gestion des valeurs extrêmes (`INT_MIN`) et libération systématique de toute allocation dynamique (`malloc`/`free`) sans fuite mémoire.

### Règles de style, de langage et de dépôt

- **Norme C :** compilation stricte sous Ubuntu 20.04 LTS avec `-Wall -Werror -Wextra -pedantic -std=gnu89` (déclaration des variables en début de bloc).
- **Style de code :** respect strict de **Betty** (`betty-style.pl` et `betty-doc.pl`), avec une nouvelle ligne à la fin de chaque fichier.
- **Modularité :** maximum **5 fonctions par fichier `.c`**.
- **Variables globales :** interdites.
- **Header :** prototypes regroupés dans `main.h` avec garde d'inclusion (`#ifndef MAIN_H` / `#define MAIN_H` / `#endif`).
- **Dépôt GitHub :** un seul dépôt par groupe, sans fichier `main.c` à la racine lors de la soumission (les fichiers de test restent dans un dossier dédié non poussé à la racine).