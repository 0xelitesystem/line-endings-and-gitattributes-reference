# 05. Renormalizing a repo

Adding or editing .gitattributes does not rewrite blobs that are already committed; you have to re-add them with `git add --renormalize .` and commit the result.

## Why a new .gitattributes seems to do nothing

The rules apply when a file passes through `git add` or a checkout. Files already in the repository keep the bytes they were committed with. Under `text=auto` there is a second reason: a path whose index copy already has CRLF is skipped on every add (chapter 03). `git add --renormalize`, available from Git 2.16.0, passes the flag `CONV_EOL_RENORMALIZE`, which bypasses that skip ([convert.c line 536](https://github.com/git/git/blob/v2.55.0/convert.c#L536)).

## The recipe

This is the sequence gitattributes(5) gives ([line 265](https://github.com/git/git/blob/v2.55.0/Documentation/gitattributes.adoc#L265)), run from a clean working tree:

```
# write or edit .gitattributes first
git add --renormalize .
git status        # shows the files that will be normalized
git commit -m "Normalize line endings"
```

If a file that must keep its bytes shows up in `git status`, unset its `text` attribute (for example `manual.pdf -text`) and add again before committing.

One limit from [git-add(1)](https://github.com/git/git/blob/v2.55.0/Documentation/git-add.adoc#L189): "Lone CR characters are untouched", so CRLF cleans to LF but CRCRLF only cleans to CRLF.

## Diagnose before and after

```
git ls-files --eol
```

Available from Git 2.8.0 ([git-ls-files(1)](https://github.com/git/git/blob/v2.55.0/Documentation/git-ls-files.adoc#L198)). Each line shows `i/` (the index copy), `w/` (the working tree copy) and `attr/` (the attribute in force). The values are `lf`, `crlf`, `mixed`, `none` or `-text`. A file showing `i/crlf` under `attr/text=auto` is exactly the case renormalize fixes.

## Why one edit shows every line changed

With no attribute and `core.autocrlf=false`, Git compares raw bytes. An editor that flipped a file from LF to CRLF has changed every line, and the diff says so. `git diff --ignore-cr-at-eol` (Git 2.16.0 and later) hides that difference for review. It fixes nothing; the commit would still carry the new endings.

## Expect one big commit

Renormalizing touches every affected file once. Do it in its own commit, so `git blame` users can skip it and reviewers are not asked to read it.
