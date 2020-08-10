# Completion is not merely syntactic

For example, in `echo $(<TAB> ...)`, syntax would say that either an
assignment (`x=blah`), a word (`./foo` or `${x}.exe`), or a redirect (`>
tmpfile`) should come next. So we'd want a sort of "possibility tree" as
the response:

1. You want an assignment. Completions are existing or common variable
names, followed by =. (E.g., `L<TAB>` would suggest LANG= and LOGNAME=
(both of which ought to be set in an interactive shell) and
LD_LIBRARY_PATH= (which may not be set, but is a very important
environment variable!)).

2. You want a command. Completions are possible commands relative to
current PATH, as well as functions.

3. You want a redirect. Completions are one of >, >|, <, >>, << or an
open fd number followed by &< or &>.

Note that all three have semantic constraints. For (2), any non-symbol,
non-whitespace character _could_ be a command name, but we want to use
contextual information---the PATH, executables on the filesystem, and
functions---to choose good completions.

For ergonomics, I imagine we'd want to really only select (2), since
it's by far the most common case. 

# Completion is about tokens

The grammar can drive the process, but we're really looking for the
next token. If the next token determines one or two more (e.g., `;`
after `then`), that's fine. But not the norm, I don't think.

# Mid-token completion

One thing in particular I'd like to get right (that existing
completion engines don't) is having the completion point in the middle
of a string. Consider the following session:

```
$ ls
bar.c    foo.c     quux.c
$ gcc f<TAB>.c
$ gcc f[oo.c].c # brackets show what appears when I hit tab
```

Whoops! Ideally, completion should use not just the context before the
marker, but also after. Consider this variant:

```
$ ls
bar.c    foo.c     foo.h    quux.c
$ gcc f<TAB>.c
$ gcc f[oo.].c # ACK!
```

Ideally, our completion engine should have enough lexical knowhow to
only produce [oo], giving us a complete token `foo.c`.
