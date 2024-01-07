# fzf - Search syntax
Save From : [GitHub - junegunn/fzf: :cherry_blossom: A command-line fuzzy finder](https://github.com/junegunn/fzf#search-syntax) 

## Content
### Search syntax

Unless otherwise specified, fzf starts in "extended-search mode" where you can type in multiple search terms delimited by spaces. e.g. `^music .mp3$ sbtrkt !fire`

| Token | Match type | Description |
| --- | --- | --- |
| `sbtrkt` | fuzzy-match | Items that match `sbtrkt` |
| `'wild` | exact-match (quoted) | Items that include `wild` |
| `^music` | prefix-exact-match | Items that start with `music` |
| `.mp3$` | suffix-exact-match | Items that end with `.mp3` |
| `!fire` | inverse-exact-match | Items that do not include `fire` |
| `!^music` | inverse-prefix-exact-match | Items that do not start with `music` |
| `!.mp3$` | inverse-suffix-exact-match | Items that do not end with `.mp3` |

If you don't prefer fuzzy matching and do not wish to "quote" every word, start fzf with `-e` or `--exact` option. Note that when `--exact` is set, `'`-prefix "unquotes" the term.

A single bar character term acts as an OR operator. For example, the following query matches entries that start with `core` and end with either `go`, `rb`, or `py`.

```
^core go$ | rb$ | py$
```
## Note