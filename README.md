git-subline-merge
=================

An interactive git merge driver which can resolve non-overlapping conflicts on individual or adjacent lines.


Introduction
------------
Git's default merge strategy will throw a conflict whenever two branches make changes to adjacent (or the same) lines. This is eminently sensible: when changes are made, neighboring lines are needed to give them context – simply merging changes when their context has also changed won't always give the desired result.

However, sometimes you wish that Git would just try to merge those conflicting lines anyway.

Enter git-subline-merge:

![git-subline-merge-example](https://raw.githubusercontent.com/paulaltin/git-subline-merge/master/example.png)


Installation
------------
* Step 1: place this script somewhere on your path

* Step 2: add these lines to your ~/.gitconfig:

```
    [merge "git-subline-merge"]
        name = An interactive merge driver for resolving sub-line conflicts
        driver = git-subline-merge %O %A %B %L %P
        recursive = binary
```

* Step 3: add this line to your git attributes file (e.g. $HOME/.config/git/attributes or $GIT_DIR/info/attributes):

```
    * merge=git-subline-merge
```


Features
--------
- Interactive resolution of conflicts on adjacent lines
- Interactive resolution of non-overlapping conflicts on individual lines
- Can be used during both merging and rebasing operations
- Word-level diff highlighting shows exactly what has changed on each branch
- Option to resolve manually using the default git editor
- Environment variables to control which hunks are unlikely to be resolvable and should be skipped


How does it work?
-----------------

The basic idea is quite straightforward – we want to try merging the conflicted hunk using the smallest possible context, i.e. the neighboring characters instead of the neighboring lines.

This is done using the simplest, most brute-force approach imaginable: we split the hunk up so that each character is on a separate line, then try to merge them, and then remove all the extra newline characters. If the merge still fails, it can only be because the changes overlap at the character level, in which case there's nothing we can do without human intervention.


Advanced options
----------------
In general, a sub-line merge is less likely to succeed the larger the conflicted hunk is, and the larger the difference in size between the original ('base') version and the two new versions ('current' and 'other').

If you find that git-subline-merge is stopping too often on conflicted hunks that it can't resolve, or skipping too many that it could have resolved, you can tune the `MAX_SIZE` and `MAX_SIZE_DIFF` constants to control which hunks are processed and which are skipped. These constants are found at the top of the script, or they can be set using environment variables `GIT_SUBLINE_MERGE_MAX_SIZE` and `GIT_SUBLINE_MERGE_MAX_SIZE_DIFF`.

The default values are `MAX_SIZE = 16` and `MAX_SIZE_DIFF = 8`, which means that conflicts spanning more than 16 lines, or whose versions differ by more than 8 lines, will be skipped. For example, a conflict which spans `12/14/14` lines in the current, base and other versions will be processed, but a conflict which spans `6/8/15` lines will be skipped.

The optimal values of these constants will depend on what kind of text you are merging, so the best thing to do is to experiment until you find values that work well for your situation.


Tested with
-----------
- git version 2.15
- Python 2.7 and 3
- Mac OS X 10.12 and 10.13
- Linux
- Windows Subsystem for Linux


Known limitations
-----------------
- *Windows*: this script does not yet work properly on Windows (no fundamental problem, just the lack of a machine to debug it with).
- *Line endings*: this script will only work on files with *nix-style LF (\n) line endings.


Credits
-------

Many thanks to Ben Nizette, Rachel Hendery and Mattias Johnsson for help with testing and debugging on Windows/Linux.


Licence
-------
All files released under GPLv3+ unless otherwise specified.
