`markin` - interpolate shell script output into Markdown
========================================================

A tool for inserting the output of shell commands into your Markdown files.
Inspired by [Cog][], but has no dependencies besides `awk`, which you already
have on your system. Works with your `$SHELL`.

A practical use case would be to insert a Git-derived changelog, or the output
of your program's `--help` option into a Markdown README.


Quickstart
----------

_Download [`markin`](markin), put it in your `$PATH`, and make it executable.
See the "[Installation](#installation)" section for a step-by-step._

Given:

```markdown
<!-- README.md -->
My command's help output:

<!-- {{{markin
# single quotes are not supported--yet (see issue #1)
mycommand --help | sed "s/^/    /"
}}} -->
<!-- {{{end}}} -->
```

then:

```bash
markin README.md > README.md_ 

# see if you like what it did
diff README.md*

# if you're happy with that, overwrite the original
mv -i README.md{_,}
```

_An in-place option (`-i` like sed) will very likely be added later. To be
honest though, I probably wouldn't do these steps by hand, because that's what
`Makefile`s and Git [pre-commit hooks][hooks] are for._

This produces:

```markdown
<!-- README.md -->
My command's help output:

<!-- {{{markin
# single quotes are not supported--yet (see issue #1)
mycommand --help | sed "s/^/    /"
}}} -->
    mycommand - do the things
  
    usage:
      mycommand [-h|--help]
      mycommand [-i|--input-file] [-o|--output-file] [-v|--verbose]
  
    where:
      ⋮
<!-- {{{end}}} -->
```

Which renders as:

> My command's help output:
> 
>     mycommand - do the things
>
>     usage:
>       mycommand [-h|--help]
>       mycommand [-i|--input-file] [-o|--output-file] [-v|--verbose]
>
>     where:
>       ⋮

Note that—just like Cog, and _unlike_ conventional template languages—the bits
used to programmatically generate the document output are preserved in-place.
There is no separate "template file" to have to deal with.


Installation
------------

Assuming a reasonably default macOS or Linux box and the Bash shell:

```bash
MARKIN=https://raw.githubusercontent.com/ernstki/markin/master/markin
mkdir -p ~/bin
curl "$MARKIN" > ~/bin/markin
chmod a+x ~/bin/markin
```

It's quite rare for it _not_ to be these days, but if `$HOME/bin` is not in
your search path, you can update that in your `~/.bash_profile` or `~/.profile`
(whichever one you have):

```bash
export PATH=$HOME/bin:$PATH
```

Log (completely) out and back in in order for this change to take effect. Then
double-check to make sure your shell can find it:

```bash
which markin
```


Other tips
----------

If you have [moreutils][mu] installed, you can update the file in-place with
`sponge`:

```bash
markin README.md | sponge README.md
```

…at least until #3 is implemented.

Test run to see changes side-by-side with the original:

```bash
diff README.md <(markin README.md)

# or
sdiff --suppress-common-lines README.md <(markin README.md)
```

…or, if you have [`delta`][delta]:


```bash
diff -u README.md <(markin README.md) | delta --side-by-side
```

That's a lot to type. Try this:

```bash
# in your ~/.bashrc, or wherever you put that stuff
markin-diff() {
    for file in "$@"; do
        diff -u "$file" <(markin "$file") | delta --side-by-side
    done
}
```

Notes
-----

Markin itself is supposed to work with whatever `awk` you have; if it doesn't
that's [a bug](../../issues).

However, macOS and probably BSD have no problem with multiple arguments in the
shebang line, so if for some odd reason you wanted to use your
`/opt/local/bin/gawk` from MacPorts, you can do so by updating the first line
of the `markin` script to say:

    #!/usr/bin/env gawk -f

Linux doesn't [doesn't support this][wps], but unless you're on an embedded
system, your `/usr/bin/awk` is probably `gawk` anyway. See [here][shb] for more
details and history of the Unix `#!` line.

I have a preference for using tools that are _already_ on the system, rather
than installing some extra package like Jinja, Mustache, or Template Toolkit.
Often I'll reach for [M4][] in cases like this. But the idea of having
a `README.md.m4` _in addition to_ the output `README.md` seemed unpalatable.
There is already so much crap in the top-level of Git repositories these days.
Everybody has `/usr/bin/awk`, basically built-in, even Windows if you use WSL
or Cygwin.

Although 100% inspired by [Cog][]'s markers (triple square brackets), I chose

     <!-- {{{markin
     shell-command --option1 --option2
     }}} -->
     Where the output of `shell-command` appears
     <!-- {{{end}}} -->

for the simple reasons that the braces don't have to be escaped in regexes, and
the triple curly braces coincide with Vim's default [`foldmarker`][fm].


Bugs / mis-features
-------------------

Single quotes within shell commands are not currently supported (see #1). This
means something as simple as `echo "Mornin', y'all!"` will crash the script.
This is pretty unacceptable. The solution will be to read the entire `markin`
block into a temp file, invoke it with `$SHELL` and `getline` the results back
in; but, for now, this is how it is.

Inline usage is not currently supported (see #2); what this means is you can't
do something like this yet:

```markdown
# My Program - v. <!--{{{markin: myprogram --version}}}--><!--{{{end}}}-->

```

…but that _is_ the dream, even if the syntax looks gnarly.

Multi-line strings, line continuation markers (_e.g._, in Bash, `\`), and
shebangs (alternate interpreters) could be supported later. Pretty easily,
I think!

Its similarity to Cog's `cog`/`end` markers, and being able to `:set
foldmethod=marker` in Vim and fold the Markin blocks is nifty and all, but
I still think you have to type way more _symbols_ than should be necessary to
get the job done.


Credits
-------

* [Cog][] by Ned Batchelder
  * A really neat tool! However, for small automation tasks like this, I work
    mostly in shell script, so

    ```python
    import cog
    import subprocess
    cog.outl(subprocess.check_output('myprogram --version'))
    ```

    seemed like a lot to remember for what is essentially a one-liner shell
    script.


License
-------

[MIT](LICENSE).

[cog]: https://nedbatchelder.com/code/cog
[fm]: http://vimdoc.sourceforge.net/htmldoc/options.html#'foldmarker'
[m4]: https://mbreen.com/m4.html#toc5
[shb]: https://www.in-ulm.de/~mascheck/various/shebang/
[wps]: https://en.wikipedia.org/wiki/Shebang_(Unix)#Character_interpretation
[hooks]: https://git-scm.com/book/en/v2/Customizing-Git-Git-Hooks
[mu]: https://joeyh.name/code/moreutils
[delta]: https://github.com/dandavison/delta
