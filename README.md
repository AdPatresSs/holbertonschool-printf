# _printf

« Allez, c'est facile, printf c'est juste de l'affichage, on règle ça en dix minutes ! » — Phrase prononcée juste avant de se battre avec va_arg et ses 3 acolytes, malloc et son meilleur pote free, sans oublier cette bonne vieille Betty sans qui nos nuits seraient douces, longues et ininterrompues.

Bon finalement, au démarrage, le flou était total : difficile de savoir par quel bout attaquer un tel morceau. Alors on a fait un pas en arrière, posé les bases et pris le temps de comprendre exactement ce qu'on nous demandait de reproduire avant de toucher au reste.

En gros, `printf` (pour *print formatted*), c'est la fonction de `<stdio.h>` qui sert à prendre des données en mémoire et à les afficher proprement sur la sortie standard (`stdout`, souvent le terminal). Pour se faire une image simple, ça fonctionne comme une phrase à trous : on fournit une chaîne modèle contenant des emplacements marqués par `%` (comme `%s` pour du texte, `%d` pour un entier ou `%c` pour un caractère), suivis des valeurs correspondantes dans le même ordre. `printf` parcourt la chaîne, remplace chaque `%` par la valeur convertie en texte, interprète les séquences spéciales comme `\n` (retour à la ligne) et renvoie le nombre total de caractères écrits (ou une valeur négative en cas d'erreur). En revanche, elle ne lit pas l'entrée clavier, n'écrit pas directement dans un fichier ciblé par défaut (`stdout` uniquement) et ne rattrape pas nos erreurs si l'ordre ou le type des arguments ne correspond pas au format.

L'objectif de ce projet n'est pas de copier ligne par ligne la libc, mais de comprendre le fonctionnement interne : lecture de la chaîne de format, manipulation des arguments variadiques, gestion de la sortie et sécurité mémoire. Lorsque la tâche ne précise pas le comportement attendu sur un cas limite, la règle est simple : **faire comme `printf` standard**.

## Contraintes et éléments autorisés

Seules les fonctions et macros suivantes sont autorisées dans le projet :

- `write` (`man 2 write`).
- `malloc` (`man 3 malloc`).
- `free` (`man 3 free`).
- `va_start` (`man 3 va_start`).
- `va_end` (`man 3 va_end`).
- `va_copy` (`man 3 va_copy`).
- `va_arg` (`man 3 va_arg`).

### Implications techniques de ces contraintes

La restriction à ces seuls appels système et macros implique plusieurs choix d'architecture :

- **Réécriture complète des utilitaires :** aucune fonction de la bibliothèque standard (`strlen`, `putchar`, etc.) n'est utilisée. Le calcul de longueur de chaîne et les conversions numériques en différentes bases sont écrits à la main.
- **Écriture directe et suivi strict du compteur :** chaque appel à `write(1, ...)` renvoie le nombre d'octets réellement écrits ; chaque fonction de sortie remonte ce compte afin que `_printf` retourne le total exact de caractères imprimés (ou `-1` en cas d'erreur).
- **Gestion rigoureuse des arguments variadiques (`va_*`) :** prise en compte des promotions implicites en C (ex. `char` promu en `int` dans `va_arg`) et appel systématique à `va_end`, y compris sur les chemins d'erreur.
- **Sécurité mémoire et cas limites :** vérification systématique des pointeurs `NULL` (ex. `format == NULL` ou `%s` pointant vers `NULL` affichant `(null)`), gestion des bornes comme `INT_MIN`, alignement sur `printf` en l'absence de consigne spécifique, et libération systématique de toute allocation dynamique (`malloc`/`free`) sans fuite.

### Règles de style, de langage et de dépôt

- **Éditeurs autorisés :** `vi`, `vim` et `emacs`.
- **Norme C :** compilation stricte sous Ubuntu 20.04 LTS avec `-Wall -Werror -Wextra -pedantic -std=gnu89` (déclarations en début de bloc).
- **Style de code :** respect strict de **Betty** (`betty-style.pl` et `betty-doc.pl`), avec une nouvelle ligne à la fin de chaque fichier.
- **Modularité :** maximum **5 fonctions par fichier `.c`**.
- **Variables globales :** interdites.
- **Header :** prototypes regroupés dans `main.h` avec garde d'inclusion (`#ifndef MAIN_H` / `#define MAIN_H` / `#endif`).
- **Dépôt GitHub :** un seul dépôt par groupe, sans `main.c` à la racine lors de la soumission (les fichiers de test restent dans un dossier dédié non poussé à la racine), et attention à ne pas cloner ou forker un dépôt de même nom avant la seconde deadline sous peine d'un score de `0%`.

## Ce qu'on pose d'abord (Tâche 0)

Pas question de tout vouloir avaler d'un coup. Pour démarrer proprement, on se concentre sur les fondations avant d'étendre progressivement aux autres spécificateurs et fonctionnalités du projet :

- Afficher les caractères normaux de `format` tels quels sur `stdout`.
- `%c` : afficher un caractère (`char`).
- `%s` : afficher une chaîne (`char *`), ou `(null)` si le pointeur est `NULL`.
- `%%` : afficher un seul `%`.
- Renvoyer le nombre total de caractères écrits sur `stdout` (sans compter l'éventuel `\0` final de chaîne), ou `-1` en cas d'erreur (ex. `format == NULL`).

*(À ce stade, pas besoin de reproduire la gestion interne du buffer de `printf`, ni de gérer les flags, la largeur de champ, la précision ou les modificateurs de taille : cela viendra au fil des tâches.)*

## Logique, flowcharts et compilation

Avant de foncer tête baissée dans le code ou de chercher pourquoi une fonction fait n'importe quoi, je m'appuie sur `man 3 printf` et des flowcharts (norme **ISO 5807**, sur `draw.io`) pour poser la logique à plat et découper les sous-processus :

1. Vérifier que `format` n'est pas `NULL` (sinon renvoyer `-1`).
2. Parcourir `format` caractère par caractère tant qu'on n'a pas atteint `'\0'`.
3. Si le caractère courant n'est pas `'%'`, l'écrire sur `stdout` et incrémenter le compteur.
4. Si on tombe sur `'%'`, avancer au caractère spécificateur suivant, appeler la conversion adaptée (`%c`, `%s`, `%%`), ajouter le nombre de caractères écrits au compteur, puis avancer l'index pour ne pas relire le spécificateur comme un caractère ordinaire.
5. Fermer `va_list` avec `va_end` et renvoyer le total.

Pour tester et comparer `_printf` avec `printf` standard sans `main.c` à la racine du dépôt :

```bash
gcc -Wall -Werror -Wextra -pedantic -std=gnu89 -Wno-format *.c