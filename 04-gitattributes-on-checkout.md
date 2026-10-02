# 04. .gitattributes on checkout: the eol attribute

On checkout, the `eol` attribute pins the ending a text file gets, and when no `eol` is set Git asks `core.autocrlf`, then `core.eol`, then the platform it was built for.

## The checkout table

From `text_eol_is_crlf` ([convert.c line 161](https://github.com/git/git/blob/v2.55.0/convert.c#L161)) and `crlf_to_worktree` ([line 592](https://github.com/git/git/blob/v2.55.0/convert.c#L592)). Git only ever adds a CR before a bare LF. It never removes one on checkout.

| Attribute | Settings | Working tree gets |
|---|---|---|
| `eol=crlf` | any | CRLF |
| `eol=lf` | any | the index bytes, unchanged |
| `text` or `text=auto`, no `eol` | `core.autocrlf=true` | CRLF |
| same | `core.autocrlf=input` | the index bytes |
| same | autocrlf false, `core.eol=crlf` | CRLF |
| same | autocrlf false, `core.eol=lf` | the index bytes |
| same | autocrlf false, `core.eol` native or unset | CRLF on Windows builds, the index bytes elsewhere |
| none | follows `core.autocrlf` alone | see chapter 02 |
| `-text` or `binary` | any | the index bytes |

Under `text=auto`, content Git detects as binary, or that already contains a CR, is left alone. "Windows builds" is literal: [config.mak.uname line 747](https://github.com/git/git/blob/v2.55.0/config.mak.uname#L747) sets `NATIVE_CRLF` in the MINGW block, and [convert.h](https://github.com/git/git/blob/v2.55.0/convert.h) maps native to CRLF only when that is set.

## eol=lf does not convert anything

gitattributes(5) says `eol=lf` "uses the same line endings in the working directory as in the index". GitHub Docs says "Git will always convert line endings to LF on checkout". The Git wording is the precise one: a blob committed with CRLF stays CRLF on disk until it is renormalized (chapter 05).

## The handoff, worked

A repo commits only `* text=auto`. A Windows laptop at `core.autocrlf=true` checks out CRLF. A Linux CI runner at the default checks out LF. A Linux box whose owner copied `autocrlf=true` from a tutorial checks out CRLF. Every party is following its rule.

## Pin it for everyone

```
* text=auto
*.sh  text eol=lf
*.bat text eol=crlf
*.cmd text eol=crlf
*.png binary
```

The `eol` lines are what make the ending the same on every machine.
