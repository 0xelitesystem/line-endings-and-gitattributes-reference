# Line endings and .gitattributes reference

A plain-language reference for the three line ending problems every mixed Windows and Linux team hits: Git warning that LF will be replaced by CRLF, a script failing with `/bin/bash^M: bad interpreter`, and a diff where one edit marks every line changed. It explains which of `.gitattributes`, `core.autocrlf` and `core.eol` decides the ending each file actually gets, read from Git's own source and documentation at the tag v2.55.0. Every source link is pinned, so the line numbers do not drift.

## The core idea

Git decides a file's line endings twice, once when you add it and once when you check it out: the `text` attribute in .gitattributes decides whether a file is normalized to LF, the `eol` attribute pins the ending it gets on checkout, and only when no `eol` attribute is set does `core.autocrlf`, then `core.eol`, then the platform default decide it. Git's compiled default is `core.autocrlf=false`, while the interactive Git for Windows installer preselects `true` on a fresh install, so two machines can disagree with nobody at fault. A committed `.gitattributes` with `eol` lines removes the disagreement. Files committed before that file existed keep their old bytes until you renormalize them.

## What is inside

- [01-what-the-warning-means.md](01-what-the-warning-means.md) both wordings of the LF/CRLF warning, what Git simulated, and why it is only a warning.
- [02-core-autocrlf-and-core-eol.md](02-core-autocrlf-and-core-eol.md) the three competing defaults and how to find which scope set yours.
- [03-gitattributes-on-add.md](03-gitattributes-on-add.md) the checkin table: what `text`, `text=auto`, `-text` and `binary` do on `git add`.
- [04-gitattributes-on-checkout.md](04-gitattributes-on-checkout.md) the checkout table, the worked handoff, and a starter .gitattributes.
- [05-renormalizing-a-repo.md](05-renormalizing-a-repo.md) `git add --renormalize`, `git ls-files --eol`, and the every-line-changed diff.
- [06-scripts-editors-and-containers.md](06-scripts-editors-and-containers.md) `bad interpreter`, the container version of it, and where EditorConfig fits.

## How to use it

- **You saw the warning on `git add`.** Read 01 for what it means, then 02 to find which config set `core.autocrlf` on your machine.
- **A script works on your laptop and fails in CI or Docker.** Read 06, add `*.sh text eol=lf`, then follow the recipe in 05.
- **You are adding a .gitattributes to an existing repo.** Copy the starter file from 04, then renormalize with 05 in its own commit.
- **A pull request shows every line changed.** Run `git ls-files --eol` on the file (05) and check whether an editor flipped the endings.
- **Two teammates get different endings from the same commit.** Run the `git config` command from 02 on both machines and compare against the table in 04.

## The stance

Line ending bugs are rarely one person's mistake. Git, the Git for Windows installer and GitHub Docs each publish a reasonable default, and they differ. The fix is to stop relying on any machine's settings and pin the ending per file type in the repository. Where GitHub Docs and Git's own documentation word a rule differently, both are shown and the Git source decides. Anything that could not be confirmed against a pinned source was left out: there are no claims here about MinGit, portable or IDE-bundled Git, package managers, or exit codes.

This reference is independent. It is not affiliated with, endorsed by, or connected to the Git project, Git for Windows, GitHub, the EditorConfig project, the Linux kernel or GNU Bash. Product names belong to their owners.

## Sources

No code is reproduced here. Behaviour is described from the sources below by file, function and line, and the only quotations are short documentation sentences and user-visible messages, each attributed where it appears.

- Git v2.55.0: [convert.c](https://github.com/git/git/blob/v2.55.0/convert.c), [environment.c](https://github.com/git/git/blob/v2.55.0/environment.c), [config.mak.uname](https://github.com/git/git/blob/v2.55.0/config.mak.uname), [gitattributes.adoc](https://github.com/git/git/blob/v2.55.0/Documentation/gitattributes.adoc), [core.adoc](https://github.com/git/git/blob/v2.55.0/Documentation/config/core.adoc), [git-add.adoc](https://github.com/git/git/blob/v2.55.0/Documentation/git-add.adoc), [git-ls-files.adoc](https://github.com/git/git/blob/v2.55.0/Documentation/git-ls-files.adoc)
- Git v2.36.0 [convert.c](https://github.com/git/git/blob/v2.36.0/convert.c), for the older warning wording
- Git for Windows installer, [install.iss at commit 4797a7b](https://github.com/git-for-windows/build-extra/blob/4797a7b2e4a11c3a104e3dc351e124e1e4bcb900/installer/install.iss)
- GitHub Docs, [Configuring Git to handle line endings](https://docs.github.com/en/get-started/git-basics/configuring-git-to-handle-line-endings)
- [EditorConfig Specification](https://spec.editorconfig.org/), version 0.17.2
- Linux [fs/binfmt_script.c](https://github.com/torvalds/linux/blob/master/fs/binfmt_script.c) and bash [execute_cmd.c](https://git.savannah.gnu.org/cgit/bash.git/plain/execute_cmd.c)

Version numbers named in the chapters (2.8.0, 2.16.0, 2.37.0, 2.46.0) were confirmed by comparing the tagged documentation or source either side of each release.

## Related

- https://github.com/0xelitesystem/git-basics-reference
- https://github.com/0xelitesystem/character-encoding-utf8-basics-reference

## Privacy

This repository is plain Markdown. There is no code to run, no build step, no tracking, no analytics, and no network calls. Reading it sends nothing anywhere.

## License

MIT. Copyright RBL Industries LLC 2026.
