# mouli :1001 Test (testeur libft, sujet v19.3)

Testeur **non officiel** pour la Libft de 42, version du sujet **19.3**. Deux commandes :

| Commande | Ce qu'elle fait |
|---|---|
| `mouli` | teste les 43 fonctions (**1001 tests**), le Makefile et `libft.h` |
| `mouli-norm` | norminette (`.c` et `.h`) + toutes les règles du sujet qui touchent à la norme et au rendu |

Le but : que tes fonctions se comportent **exactement** comme celles du `man` (pas « mieux »), et que tu saches précisément pourquoi un test échoue.

## Le thème : les 1001 nuits

`mouli`

`mouli-norm`

Tu préfères un affichage simple ? `mouli --sobre` et `mouli-norm --sobre` (ou `export MOULI_SOBRE=1` dans ton `~/.zshrc`).

---

## Installation

Il faut `python3` (3.8 ou plus), `cc`, `make`, `ar`, `nm` : tout est déjà sur les postes 42.

```sh
bash install_mouli.sh
```

`install_mouli.sh` est un fichier unique qui contient tout le testeur. Il le copie dans `~/.mouli` et ajoute les alias `mouli` et `mouli-norm` à ton `~/.zshrc` (et `~/.bashrc` s'il existe). Ouvre ensuite un nouveau terminal, ou tape `source ~/.zshrc`.

- Avec l'archive à la place : `tar xzf mouli.tar.gz && bash mouli/install.sh`.
- Mise à jour : relance l'installateur de la nouvelle version.
- Désinstallation : `bash ~/.mouli/install.sh --uninstall` (supprime aussi les alias et le cache).
- `mouli-norm` a besoin de la norminette. Si elle manque : `python3 -m pip install --user --upgrade norminette`.

## Utilisation

```sh
cd ~/chemin/vers/libft
mouli                 # tout : Makefile, libft.h, parties 1, 2 et 3
mouli -f split,itoa   # seulement certaines fonctions (avec ou sans ft_)
mouli -p 2            # seulement la partie 2
mouli -v              # détail de chaque test, y compris les cas non définis
mouli --git           # teste le DERNIER COMMIT, c'est-à-dire ce que verra la moulinette
mouli -q              # saute les tests longs du Makefile (relink, clean...)
mouli --strict        # compte aussi les écarts sur les comportements indéfinis
mouli-norm            # norminette + règles du sujet
mouli-norm --git      # seulement les fichiers commités
mouli --sobre         # sans le thème (pareil pour mouli-norm)
```

Autres options : `--timeout 5` (secondes par test, 3 par défaut), `--seed 7` (autre graine pour les tests aléatoires), `--no-color`, `--keep` (garde les binaires de test). Un log sans couleurs est écrit dans `~/.cache/mouli/dernier_test.log` (et `derniere_norme.log`).

`mouli` ne modifie jamais ton dossier : il travaille sur une copie (pas de `.o` laissés derrière).

## Lire le résultat

```
  ✔ ft_strlcat      OK    28/28  ••••••••••••••••••••••••••••
  ✗ ft_substr       KO    30/35  •••S••••••SS•••••SS•••••••••••••••i
    [CRASH] #4 len plus grand que ce qui reste : on s'arrête à la fin de s
        Plantage     Segmentation fault (SIGSEGV)
        Cause        accès hors limites : s[5] alors que s ne fait que 5 octets (1 octet après la fin)
        Appel        ft_substr("hola", 2, 30)
        Rappel       len est une longueur MAXIMALE : il ne reste que 2 caractères après start...
```

Chaque symbole est un test :

| Symbole | Signification | Compté |
|---|---|---|
| `•` | OK | |
| `✗` | mauvais résultat (attendu / obtenu affichés, octets différents surlignés) | oui |
| `S` | plantage (segfault, bus error, abort, double free...) avec la cause exacte | oui |
| `T` | timeout : boucle infinie probable | oui |
| `L` | fuite mémoire (taille, contenu du bloc, appel qui l'a allouée) | oui |
| `M` | mauvais comportement quand un `malloc` échoue (pas `NULL`, fuite, crash) | oui |
| `O` | ta fonction écrit sur la sortie standard (un `printf` de debug oublié) | oui |
| `!` | avertissement : comportement indéfini différent de la libc, `malloc` plus grand que nécessaire, lenteur | non |
| `i` | information : cas que le sujet ne définit pas (ex : `NULL`), visible avec `-v` | non |

Le résumé final imite la moulinette : **100/100 si tout passe, 0/100 à la première erreur** (et il te dit laquelle, avec le numéro de la nuit fatale).

Répartition des 1001 tests : partie 1 : 571, partie 2 : 292, partie 3 : 138.

## Ce qui est testé

**Moteur.** Chaque test tourne dans son propre processus : un crash ou une boucle infinie ne fait jamais tomber le testeur.
- Chaque buffer passé à ta fonction est collé à une page mémoire protégée : lire ou écrire **un seul octet** de trop fait planter, et le rapport dit quel octet de quel paramètre (ex : `dst[8] alors que dst ne fait que 8 octets`). Les paramètres `const` sont en lecture seule : y écrire est signalé.
- Chaque `malloc` fait pendant ta fonction est surveillé : dépassement de bloc, utilisation après `free`, double `free`, `free` d'un pointeur qui n'a pas été alloué, fuites (avec le contenu du bloc perdu).
- Échecs de `malloc` : pour chaque fonction qui alloue, le testeur fait échouer le 1er `malloc`, puis le 2e, etc. Ta fonction doit renvoyer `NULL` **et** libérer ce qu'elle avait déjà alloué (`ft_split`, `ft_lstmap`...).
- Taille des allocations : un `malloc` plus grand que nécessaire est signalé en avertissement (certains correcteurs le vérifient).
- Fonctions externes : le sujet dit que la partie 1 « must not rely on any external functions », et donne pour chaque fonction des parties 2 et 3 la liste « External functs. ». Un `malloc` ou un `free` pendant l'appel d'une fonction qui n'y a pas droit (même caché derrière une autre de tes `ft_`) est signalé en avertissement ; `mouli-norm` dit s'il est direct (fonction interdite) ou indirect.

**Partie 1.** Comparaison directe avec la vraie libc (`strlcpy`, `strlcat`, `strnstr` : la version BSD du système, de libbsd, ou un modèle fidèle au `man` BSD si aucune n'existe). Entre autres : valeurs de `c` hors de `[0, 255]` (`'a' + 256`, `-1`, `200`...), octets > 127 comparés en `unsigned char`, chevauchements de `memmove` dans les deux sens, `strlcat` quand `size` ≤ `strlen(dst)` ou quand `dst` n'a pas de `'\0'` dans ses `size` octets, `strnstr` avec `len` qui coupe le mot cherché, `atoi` avec tous les octets de 1 à 255 devant le nombre, `INT_MIN`, `calloc(0, 0)` (règle du sujet), dépassements de `nmemb * size`, et des milliers de cas aléatoires.

**Partie 2.** Résultats attendus calculés avec la libc. `ft_substr` avec `start` après la fin ou `len = SIZE_MAX`, `ft_split` (séparateurs multiples, `c = '\0'`, chaque octet comme séparateur, tableau terminé par `NULL`, chaque mot alloué séparément), `ft_strtrim`, `ft_itoa(INT_MIN)`, les appels exacts reçus par `f` dans `ft_strmapi` et `ft_striteri` (indice, caractère, adresse dans `s`), le contenu exact écrit dans le fd par les `ft_put*_fd`.

**Partie 3.** Les listes sont **obligatoires** en v19.3 (`make` doit les compiler) et `ft_lstsize` renvoie un `unsigned int`. Le testeur vérifie l'ordre des nœuds, que `del` reçoit bien le contenu (et une seule fois), que `ft_lstdelone` ne libère pas le suivant, que `ft_lstclear` met `*lst` à `NULL`, que `ft_lstiter` appelle `f` sur chaque contenu (même `NULL`), que `ft_lstmap` ne touche pas la liste d'origine et ne fuit pas quand un `malloc` échoue (le contenu renvoyé par `f` doit passer par `del`, une seule fois, jamais celui de la liste d'origine). Les listes partent aussi d'un nœud au milieu, ont des contenus invalides (jamais lus), partagés ou `NULL`, et vont jusqu'à 70 000 nœuds.

**Comportements indéfinis** (`NULL`, valeurs hors domaine...). Ta fonction et la libc tournent chacune dans leur processus et on compare : « plante », « boucle » ou « renvoie X ». Une différence s'affiche en `!` mais **n'est pas comptée** (sauf avec `--strict`), parce que le `man` ne définit pas ces cas.

**Makefile.** `make` crée `libft.a`, règle par défaut `all`, règles `$(NAME) all clean fclean re`, `cc` avec `-Wall -Wextra -Werror` sur chaque `.c`, `ar` (pas `libtool`), pas de `-std=c99` ni de `-fsanitize`, sources listées sans wildcard, toutes les fonctions (listes comprises) dans `libft.a`, pas de relink (3 fois), recompilation d'un seul `.o` quand un `.c` change, `clean`, `fclean`, `re`.

**libft.h.** Compile seul (inclus deux fois) avec les flags, pas de `restrict`, `t_list` exactement comme le sujet, les 43 prototypes identiques au sujet v19.3.

## mouli-norm

- **Fichiers rendus** : seulement `Makefile`, `libft.h`, `README.md` et des `ft_*.c` à la racine. Signale les fichiers en trop (`main.c`, `.o`, `libft.a` commités...), les fichiers **pas suivis par git** (qui ne seront pas rendus), les changements non commités et les commits pas poussés.
- **Norminette** sur tous les `.c` et `.h`, avec une explication en français de chaque erreur. Comme la moulinette, les `.c` sont aussi passés avec `-R CheckForbiddenSourceHeader` et les `.h` avec `-R CheckDefine` : le testeur indique si une erreur n'apparaît qu'avec la norminette normale (celle que tape un correcteur), et la compte quand même.
- **Symboles** (chaque `.c` compilé seul, puis analysé avec `nm`) : fonctions externes interdites (chaque fonction n'a droit qu'à ce que liste « External Function » : `malloc`, `free`, `write` selon les cas), variables globales (même `static`), fonctions d'aide non `static`, `main` oublié, fonctions définies deux fois, usage indirect (ex : `ft_putnbr_fd` qui passe par `ft_itoa`, donc par `malloc`).
- **Makefile** (lecture) et **libft.h**.
- **README.md** (chapitre V du sujet) : première ligne en italique `*This project has been created as part of the 42 curriculum by <login>.*`, sections Description / Instructions / Resources, usage de l'IA décrit, description détaillée de la bibliothèque.

## Limites

- C'est un testeur non officiel : il ne remplace pas la vraie moulinette ni une évaluation. Il est volontairement plus exigeant qu'elle.
- Testé sous Linux (glibc 2.39, gcc et clang). Il est écrit pour fonctionner aussi sous macOS, mais n'y a pas encore été lancé.
- Ta libft est compilée par ton propre Makefile, avec le `cc` de la machine. Lance `mouli` sur un poste 42 pour être dans les conditions de la moulinette : certains bugs ne se voient qu'avec un compilateur (par exemple, une division par zéro passe inaperçue avec gcc mais plante avec clang).
- Les cas `NULL` de la partie 2 et 3 ne sont pas définis par le sujet : ils s'affichent en information, à toi de savoir défendre ton choix.

## Règles du sujet sur l'IA

Le sujet Libft demande de ne pas utiliser l'IA pour obtenir les réponses. Ce testeur ne contient **aucune solution** : les résultats attendus viennent de la vraie libc ou de valeurs écrites à la main, et les messages expliquent ce que dit le `man` ou le sujet, pas comment coder la fonction.
