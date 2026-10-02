# 03. .gitattributes on add: the text attribute

On `git add`, the `text` attribute in .gitattributes decides whether a file is normalized to LF in the repository, and only when it is unspecified does `core.autocrlf` get a say.

## The checkin table

Read from `convert_attrs` and `crlf_to_git` in [convert.c at v2.55.0](https://github.com/git/git/blob/v2.55.0/convert.c#L1321):

| Attribute on the path | What `git add` stores |
|---|---|
| `-text` or `binary` | the bytes as they are, never converted |
| `text` (with or without `eol`) | every CRLF pair becomes LF, even if the file was committed earlier with CRLF; a lone CR is kept |
| `text=auto` | CRLF becomes LF only if Git detects the content as text and the copy already in the index has no CRLF |
| none, `core.autocrlf=false` | the bytes as they are |
| none, `core.autocrlf=true` or `input` | treated as `text=auto` |

Setting `eol` alone also sets `text`: gitattributes(5) says "specifying `eol` automatically sets `text` if `text` was left unspecified" ([line 174](https://github.com/git/git/blob/v2.55.0/Documentation/gitattributes.adoc#L174)).

## The text=auto exception that surprises people

Under `text=auto`, gitattributes(5) says a file is converted only if "it is text and the file was not already in Git with CRLF endings" ([line 157](https://github.com/git/git/blob/v2.55.0/Documentation/gitattributes.adoc#L157)). In convert.c this is the check commented "the new safer autocrlf handling" ([line 536](https://github.com/git/git/blob/v2.55.0/convert.c#L536)). So adding `* text=auto` to an old repo does not clean up files that were already committed with CRLF. Chapter 05 is the fix.

## binary is a macro

gitattributes(5) defines it as `[attr]binary -diff -merge -text` ([line 1318](https://github.com/git/git/blob/v2.55.0/Documentation/gitattributes.adoc#L1318)). GitHub Docs describes it as "an alias for -text -diff"; the Git definition also turns off merging.

## What "overrides" means here

GitHub Docs says that a committed .gitattributes "overrides the core.autocrlf setting for all repository contributors". That is GitHub's summary. Read against convert.c, `text` overrides autocrlf's decision about whether to normalize. It does not by itself fix the ending a file gets on checkout. Only `eol` does that, which is chapter 04.
