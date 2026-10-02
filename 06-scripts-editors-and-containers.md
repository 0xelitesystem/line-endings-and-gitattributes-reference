# 06. Scripts, editors and containers

A shell script checked out with CRLF fails with `/bin/bash^M: bad interpreter` because the kernel reads the carriage return as part of the interpreter's name, and the fix is `*.sh text eol=lf` plus a renormalize.

## What the kernel does with the #! line

In Linux, `load_script` in [fs/binfmt_script.c](https://github.com/torvalds/linux/blob/master/fs/binfmt_script.c) ends the `#!` line at the LF and trims only spaces and tabs. A CRLF line therefore asks for an interpreter called `/bin/bash` followed by a carriage return. No such file exists, and execution fails with "No such file or directory".

## Where the ^M comes from

bash then explains the failure. In [execute_cmd.c](https://git.savannah.gnu.org/cgit/bash.git/plain/execute_cmd.c) (the same check is in [bash 5.1](https://git.savannah.gnu.org/cgit/bash.git/plain/execute_cmd.c?h=bash-5.1)), if the interpreter name ends in a CR, bash replaces it with the two visible characters `^M` before printing:

```
./deploy.sh: /bin/bash^M: bad interpreter: No such file or directory
```

With `#!/usr/bin/env bash` the CR rides along on the argument instead, so `env` goes looking for a program named `bash` plus a carriage return. The cause and the fix are the same.

## The container version of the same bug

1. A Windows machine at `core.autocrlf=true` checks out `entrypoint.sh` with CRLF (chapter 04).
2. Docker `COPY` copies those bytes exactly.
3. The Linux kernel inside the container parses the `#!` line correctly, CR included, and the start fails.

Nobody misbehaved. The repo just never pinned the ending.

## The fix

```
*.sh text eol=lf
```

Commit that line in .gitattributes, then `git add --renormalize .` and commit (chapter 05). Every checkout on every machine now writes LF for shell scripts. Check with `git ls-files --eol '*.sh'`.

## EditorConfig is a different layer

The EditorConfig Specification, version 0.17.2 at [spec.editorconfig.org](https://spec.editorconfig.org/), defines `end_of_line` as `lf`, `cr` or `crlf`, case-insensitive. That tells an editor what to write when you save. It does not change what Git stores or what Git writes on checkout. Use it alongside .gitattributes, not instead of it:

```
[*.sh]
end_of_line = lf
```
