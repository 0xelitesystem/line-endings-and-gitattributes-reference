# 01. What the warning means

The warning means Git simulated adding your file and then checking it out again, and the line endings now on disk would not survive that round trip; nothing has been changed yet.

## The two wordings people search for

Git 2.37.0 and later print one of these:

```
warning: in the working copy of 'build.sh', LF will be replaced by CRLF the next time Git touches it
warning: in the working copy of 'build.sh', CRLF will be replaced by LF the next time Git touches it
```

Git 2.36 and earlier printed:

```
warning: LF will be replaced by CRLF in build.sh.
The file will have its original line endings in your working directory
```

Sources: [convert.c at v2.55.0](https://github.com/git/git/blob/v2.55.0/convert.c#L204-L222) and [at v2.36.0](https://github.com/git/git/blob/v2.36.0/convert.c). The `warning: ` prefix comes from [usage.c](https://github.com/git/git/blob/v2.55.0/usage.c).

## What Git actually checked

When you run `git add`, the function `crlf_to_git` in convert.c counts the line endings in your file, simulates the add (CRLF to LF, if this file is normalized), then simulates the next checkout (LF to CRLF, if this file gets CRLF on checkout). If the simulated result differs from the file on disk, it warns. The blob it stores follows your rules. The file in your folder is not rewritten now; it changes the next time a checkout, switch or reset writes that path.

## Which direction you got

- **LF will be replaced by CRLF.** The file has LF endings and its checkout rule is CRLF: usually `core.autocrlf=true`, or `eol=crlf` in .gitattributes.
- **CRLF will be replaced by LF.** The file has CRLF endings, it is normalized to LF on add, and its checkout rule is LF: usually `core.autocrlf=input`, or `text eol=lf`.

Chapter 03 has the full table of which setting decides.

## Why it is only a warning

The switch is `core.safecrlf`. Its compiled default is warn: [environment.c line 63](https://github.com/git/git/blob/v2.55.0/environment.c#L61-L63) sets `global_conv_flags_eol = CONV_EOL_RNDTRP_WARN`, a default the [core.safecrlf entry](https://github.com/git/git/blob/v2.55.0/Documentation/config/core.adoc#L178) never states. Set it to `true` and the same case is fatal (`fatal: LF would be replaced by CRLF in build.sh`). Set it to `false` and it is silent. Per gitattributes(5), `git add` and `git diff` run this check and `git apply` does not.

Silencing the warning changes nothing about the endings. Fix the rule instead.
