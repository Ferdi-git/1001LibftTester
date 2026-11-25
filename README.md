# The 1001 Tester (mouli): libft tester for the 42 subject v19.3

An **unofficial** tester for the 42 Libft, subject version **19.3**. Two commands:

| Command | What it does |
|---|---|
| `mouli` | tests the 43 functions (**1001 tests**), the Makefile and `libft.h` |
| `mouli-norm` | norminette (`.c` and `.h`) + every subject rule about the norm and the submission |

The goal: your functions should behave **exactly** like the `man` page (not "better"), and you should know precisely why a test fails.

---

## Install

You need `python3` (3.8 or newer), `cc`, `make`, `ar`, `nm`: all of them are already on 42 computers.

```sh
bash install_mouli.sh
```

`install_mouli.sh` is a single file that holds the whole tester. It copies it to `~/.mouli` and adds the `mouli` and `mouli-norm` aliases to your `~/.zshrc` (and `~/.bashrc` if it exists). Then open a new terminal, or type `source ~/.zshrc`.

- With the archive instead: `tar xzf mouli.tar.gz && bash mouli/install.sh`.
- Update: run the installer of the new version.
- Uninstall: `bash ~/.mouli/install.sh --uninstall` (also removes the aliases and the cache).
- `mouli-norm` needs norminette. If it is missing: `python3 -m pip install --user --upgrade norminette`.

## Usage

```sh
cd ~/path/to/libft
mouli                 # everything: Makefile, libft.h, parts 1, 2 and 3
mouli -f split,itoa   # only some functions (with or without ft_)
mouli -p 2            # only part 2
mouli -v              # every failing test in full, plus the undefined cases
mouli --git           # tests the LAST COMMIT, that is what the moulinette will see
mouli -q              # skips the slow Makefile tests (relink, clean...)
mouli --strict        # also counts differences on undefined behaviour
mouli-norm            # norminette + subject rules
mouli-norm --git      # only the committed files
mouli --plain         # no big ASCII banners (same for mouli-norm)
```

Other options: `--timeout 5` (seconds per test, 3 by default), `--seed 7` (another seed for the random tests), `--no-color`, `--keep` (keeps the test binaries). A log without colours is written to `~/.cache/mouli/last_test.log` (and `last_norm.log`).

`mouli` never changes your folder: it works on a copy (no `.o` files left behind).

## Reading the output

```
   ██╗ ██████╗  ██████╗  ██╗    ╭────────────────────────────────────────────────────╮
  ███║██╔═████╗██╔═████╗███║    │ THE 1001 TESTER  ·  mouli 2.2.0                    │
  ╚██║██║██╔██║██║██╔██║╚██║    │ libft · subject v19.3 · 43 functions · 1001 tests  │
   ██║████╔╝██║████╔╝██║ ██║    │ unofficial: does not replace the moulinette        │
   ██║╚██████╔╝╚██████╔╝ ██║    │ mouli -h for the options · mouli-norm for the norm │
   ╚═╝ ╚═════╝  ╚═════╝  ╚═╝    ╰────────────────────────────────────────────────────╯
```

The output has five parts, always in the same order:

1. **One line per function** while the tests run, with one symbol per test.
2. **FAILURES**: every function with a failing test gets its own block, with at most 3 failing tests in full (`mouli -f <name> -v` shows all of them). Each one says what was called, what was expected, what your function did, and **why** (the rule from the `man` page or the subject).
3. **WARNINGS**: undefined behaviour that differs from libc, extra `malloc`, slowness. Not counted, but each one is explained in full.
4. **NOTES**: cases the subject does not define (`NULL` arguments...): what your function does, so you can defend your choice. Not counted.
5. **SUMMARY**: Makefile, `libft.h` and each part, the list of **what failed** (function, how many tests, which ones), the functions with warnings, and a big **OK** or **KO** with the score.

```
━━ FAILURES  (1 function · why each one fails) ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

  ── ft_strncmp · 7 failing tests ────────────────────────────────────────────────────────────────

    KO      #12 · test 354/1001  byte > 127 against ASCII: compared as unsigned char
      › Return value: wrong sign
        Call           ft_strncmp("\x80", "a", 1)
        Expected       a positive result (libc returns 31)
        Got            -225 (negative)
        Why            Characters are compared as unsigned char: 0x80 (128) is greater than 'a'
                       (97).


━━ SUMMARY ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

  Makefile  ✔ 8/8 checks
  libft.h   ✔ 4/4 checks
  Part 1    ✗ 22/23 functions    564/571 tests
  Part 2    ✔ 11/11 functions    292/292 tests
  Part 3    ✔ 9/9 functions      138/138 tests
  Tests     994/1001 passed   · 4 warnings, 25 notes (not counted)

  What failed
    ✗ ft_strncmp      7 of 32 tests failed   #12 #13 #14 #15 #19 #20 #23

  Warnings in  ft_memmove, ft_toupper, ft_tolower, ft_atoi   (see WARNINGS above)

  ██╗  ██╗ ██████╗ 
  ██║ ██╔╝██╔═══██╗     ✗ 0/100 
  █████╔╝ ██║   ██║
  ██╔═██╗ ██║   ██║    First error at test 354/1001:
  ██║  ██╗╚██████╔╝    ft_strncmp #12  byte > 127 against ASCII: compared as unsigned char
  ╚═╝  ╚═╝ ╚═════╝     Like the moulinette: one error gives 0.
```

Each symbol is one test:

| Symbol | Meaning | Counted |
|---|---|---|
| `•` | OK | |
| `✗` | wrong result (expected / got shown, differing bytes highlighted) | yes |
| `S` | crash (segfault, bus error, abort, double free...) with the exact cause | yes |
| `T` | timeout: probably an infinite loop | yes |
| `L` | memory leak (size, content of the block, call that allocated it) | yes |
| `M` | wrong behaviour when a `malloc` fails (not `NULL`, leak, crash) | yes |
| `O` | your function writes to standard output (a forgotten debug `printf`) | yes |
| `!` | warning: undefined behaviour different from libc, `malloc` larger than needed, slowness | no |
| `i` | note: case the subject does not define (e.g. `NULL`), listed in NOTES | no |

Like the moulinette, the result is **100/100 if everything passes, 0/100 at the first error**, and it tells you which one.

The 1001 tests are split as part 1: 571, part 2: 292, part 3: 138.

### The look

The header shows the big "1001" of **The 1001 Tester** (a big "NORM" for `mouli-norm`), the progress line counts the tests (`Test 245/1001  ft_strlcat  3/28`), and the summary ends with a big **OK** or **KO**. Colours: green for what passes, red for errors, amber for warnings, blue for notes (256 colours, with an automatic fallback to 16). The error explanations never depend on the look.

Prefer a plain display? Use `--plain` (or `export MOULI_PLAIN=1` in your `~/.zshrc`): the header becomes plain text.

## What is tested

**Engine.** Each test runs in its own process: a crash or an infinite loop never takes the tester down.
- Every buffer passed to your function is stuck to a protected memory page: reading or writing **a single byte** too far crashes, and the report says which byte of which parameter (e.g. `dst[8] while dst is only 8 bytes long`). `const` parameters are read-only: writing into them is reported.
- Every `malloc` made during your function is watched: block overflow, use after `free`, double `free`, `free` of a pointer that was not allocated, leaks (with the content of the lost block).
- `malloc` failures: for each function that allocates, the tester makes the 1st `malloc` fail, then the 2nd, and so on. Your function must return `NULL` **and** free what it had already allocated (`ft_split`, `ft_lstmap`...).
- Allocation sizes: a `malloc` larger than needed is reported as a warning (some evaluators check it).
- External functions: the subject says part 1 "must not rely on any external functions", and gives for each function of parts 2 and 3 its "External functs." list. A `malloc` or a `free` during a call to a function that may not use them (even hidden behind another of your `ft_` functions) is reported as a warning; `mouli-norm` tells you whether it is direct (forbidden function) or indirect.

**Part 1.** Direct comparison with the real libc (`strlcpy`, `strlcat`, `strnstr`: the BSD version of the system, of libbsd, or a model faithful to the BSD `man` page if none exists). Among others: values of `c` outside `[0, 255]` (`'a' + 256`, `-1`, `200`...), bytes > 127 compared as `unsigned char`, `memmove` overlaps in both directions, `strlcat` when `size` ≤ `strlen(dst)` or when `dst` has no `'\0'` in its `size` bytes, `strnstr` with a `len` that cuts the word searched for, `atoi` with every byte from 1 to 255 in front of the number, `INT_MIN`, `calloc(0, 0)` (subject rule), `nmemb * size` overflows, and thousands of random cases.

**Part 2.** Expected results computed with libc. `ft_substr` with `start` past the end or `len = SIZE_MAX`, `ft_split` (repeated separators, `c = '\0'`, each byte as a separator, array ended by `NULL`, each word allocated separately), `ft_strtrim`, `ft_itoa(INT_MIN)`, the exact calls received by `f` in `ft_strmapi` and `ft_striteri` (index, character, address inside `s`), the exact content written to the fd by the `ft_put*_fd` functions.

**Part 3.** Lists are **mandatory** in v19.3 (`make` must build them) and `ft_lstsize` returns an `unsigned int`. The tester checks the order of the nodes, that `del` gets the content (and only once), that `ft_lstdelone` does not free the next node, that `ft_lstclear` sets `*lst` to `NULL`, that `ft_lstiter` calls `f` on every content (even `NULL`), that `ft_lstmap` does not touch the original list and does not leak when a `malloc` fails (the content returned by `f` must go through `del`, exactly once, never the one of the original list). Lists also start from a node in the middle, have invalid (never read), shared or `NULL` contents, and go up to 70,000 nodes.

**Undefined behaviour** (`NULL`, out-of-domain values...). Your function and libc each run in their own process and are compared: "crashes", "loops" or "returns X". A difference is shown as `!` but **is not counted** (except with `--strict`), because the `man` page does not define these cases.

**Makefile.** `make` creates `libft.a`, default rule `all`, rules `$(NAME) all clean fclean re`, `cc` with `-Wall -Wextra -Werror` on each `.c`, `ar` (not `libtool`), no `-std=c99` nor `-fsanitize`, sources listed without wildcard, every function (lists included) in `libft.a`, no relink (3 times), only one `.o` rebuilt when a `.c` changes, `clean`, `fclean`, `re`.

**libft.h.** Compiles alone (included twice) with the flags, no `restrict`, `t_list` exactly as in the subject, the 43 prototypes identical to the subject v19.3.

## mouli-norm

Same layout as `mouli`: one line per group of checks, then **PROBLEMS** in full, then **WARNINGS** with their explanations, then the **SUMMARY** with the list of what failed and a big **OK** (`NORM OK`) or **KO** (`TO FIX`).

- **Files**: only `Makefile`, `libft.h`, `README.md` and `ft_*.c` files at the root. Reports extra files (`main.c`, `.o`, committed `libft.a`...), files **not tracked by git** (they will not be submitted), uncommitted changes and commits not pushed.
- **Norminette** on every `.c` and `.h`, with a plain-English explanation of each error. Like the moulinette, `.c` files are also run with `-R CheckForbiddenSourceHeader` and `.h` files with `-R CheckDefine`: the tester tells you if an error only shows with plain norminette (the one an evaluator types), and still counts it.
- **Symbols** (each `.c` compiled alone, then read with `nm`): forbidden external functions (each function may only use what its "External functs." line lists: `malloc`, `free`, `write` depending on the function), global variables (even `static`), non-`static` helper functions, a forgotten `main`, functions defined twice, indirect use (e.g. `ft_putnbr_fd` going through `ft_itoa`, so through `malloc`).
- **Makefile** (read only) and **libft.h**.
- **README.md** (chapter V of the subject): first line in italics `*This project has been created as part of the 42 curriculum by <login>.*`, Description / Instructions / Resources sections, use of AI described, detailed description of the library.

## Limits

- This is an unofficial tester: it does not replace the real moulinette or an evaluation. It is stricter on purpose.
- Tested on Linux (glibc 2.39, gcc and clang). It is written to work on macOS too, but has not been run there yet.
- Your libft is built by your own Makefile, with the machine's `cc`. Run `mouli` on a 42 computer to be in the moulinette's conditions: some bugs only show with one compiler (for example, a division by zero goes unnoticed with gcc but crashes with clang).
- The `NULL` cases of parts 2 and 3 are not defined by the subject: they are shown as information, and it is up to you to defend your choice.

## The subject's AI rules

The Libft subject asks you not to use AI to get the answers. This tester holds **no solution**: the expected results come from the real libc or from values written by hand, and the messages explain what the `man` page or the subject says, not how to write the function.
