# 02. core.autocrlf and core.eol

Git's compiled default is `core.autocrlf=false`, but the interactive Git for Windows installer preselects `true` on a fresh install, so two teammates can follow different published rules with nobody at fault.

## Three defaults, three sources

- **Git itself.** [environment.c line 61](https://github.com/git/git/blob/v2.55.0/environment.c#L61-L63) sets `auto_crlf = AUTO_CRLF_FALSE`. With no attribute, Git then stores and checks out bytes as they are.
- **The Git for Windows installer.** Its page "Configuring the line ending conversions" preselects "Checkout Windows-style, commit Unix-style line endings" ([install.iss line 2344](https://github.com/git-for-windows/build-extra/blob/4797a7b2e4a11c3a104e3dc351e124e1e4bcb900/installer/install.iss#L2344-L2349)) and then writes `core.autocrlf=true` with `git config --system` ([line 3236](https://github.com/git-for-windows/build-extra/blob/4797a7b2e4a11c3a104e3dc351e124e1e4bcb900/installer/install.iss#L3236)). A reinstall replays the earlier choice, or one inferred from the existing system config.
- **GitHub Docs.** The page "Configuring Git to handle line endings" tells Windows users to run `git config --global core.autocrlf true` and macOS and Linux users to run `git config --global core.autocrlf input`.

## What each value does

Per [core.adoc](https://github.com/git/git/blob/v2.55.0/Documentation/config/core.adoc#L223), `core.autocrlf=true` "is the same as setting the `text` attribute to "auto" on all files and core.eol to "crlf"". For a file with no `text` or `eol` attribute:

| core.autocrlf | On add | On checkout |
|---|---|---|
| `false` | no conversion | no conversion |
| `true` | CRLF to LF for text files | LF to CRLF |
| `input` | CRLF to LF for text files | no conversion |

`core.eol` accepts `lf`, `crlf` or `native`, and "The default value is `native`" ([core.adoc line 167](https://github.com/git/git/blob/v2.55.0/Documentation/config/core.adoc#L167)). It is "ignored if `core.autocrlf` is set to `true` or `input`", and only affects text files.

## Find out who set your value

```
git config get --all --show-origin --show-scope core.autocrlf
```

This form is documented from Git 2.46.0. Older Git needs the deprecated form:

```
git config --show-origin --show-scope --get-all core.autocrlf
```

Each line names the scope and the file. Per [git-config(1)](https://github.com/git/git/blob/v2.55.0/Documentation/git-config.adoc), the last value found takes precedence.
