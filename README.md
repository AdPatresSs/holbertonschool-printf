# _printf

Implémentation personnalisée de la fonction standard `printf` en C (`gnu89`), respectant les règles du style **Betty**.

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