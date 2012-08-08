# Syncing remotes

## `git push<CR>` and `git pull<CR>`

Be careful when using “`git push`” and “`git pull`” without other arguments, **their comportment depend on the configuration**.

- “`git pull`” may use an other branch than master.
- “`git push`” may push _every_ branches you have.

To be safe, always specify the remote(s) and the branch(es) : “`git pull origin topic/blabla`” and “`git push origin master`”

Here is the `git config`'s man page section about `push.default`.

> Defines the action git push should take if no refspec is given on the command line, no refspec is configured in the remote, and no refspec is implied by any of the options given on the command line. Possible values are:
> 
> o  nothing - do not push anything.
> 
> o  matching - push all branches having the same name in both ends. This is for those who prepare all the branches into a publishable shape and then push them out with a single command. It is not appropriate for pushing into a repository shared by multiple users, since locally stalled branches will attempt a non-fast forward push if other users updated the branch.  
>   *This is currently the default, but Git 2.0 will change the default to simple*.
> 
> o  upstream - push the current branch to its upstream branch. With this, git push will update the same remote ref as the one which is merged by git pull, making push and pull symmetrical. See "branch.<name>.merge" for how to configure the upstream branch.
> 
> o  __simple__ - like upstream, but refuses to push if the upstream branch's name is different from the local one. This is the safest option and is well-suited for beginners.  
>   *It will become the default in Git 2.0*.
> 
> o  current - push the current branch to a branch of the same name.

Which option are you using (“`git config --global --get push.default`”)?

If you're not totally confident, use “`git config --global push.default simple`” to set this safe default value.

## `git push --force`

**Never use “`git push --force …`”**, use the ”`+branch`” syntax instead.

The “`git push -f …`” command will force the push of _every pushed branch_. Of course, “`git push -f`” (without remote nor refs) should be banned.

In the contrary, the “`git push origin master +develop`” form let you use normal push for the master branch, and will forces the push of the develop branch only. Way more safe to use.

Don't forget you can do many things in one command: “`git push origin develop +topic/topic1 :topic/topic2`” pushes develop, pushes-force topic/topic1 and deletes (on server) topic/topic2.

## `git pull`

Beware of “`git pull …`”, **it does not fetch others branches**!

The command “`git pull origin master`” only fetches the master branch [^pullfetch]. Use “`git fetch origin`” to fetch the origin's branches. In a more general way, use “`git fetch --all --prune`” to sync remotes.

[^pullfetch]: Don't hope `git pull` without other args will do the trick: it is just a shortcut for `git pull <the-remote> <the-branch>`, so the rule is _exactly_ the same.

Oh, did I say `git pull` may use the `--rebase` option automatically, depending on the configuration (can be depending on the branch)?

**You may stop using “`git pull …`” at all**

* You should use “`git fetch --all --prune`”, in order to synchronize your view of the remote repositories, then “`git merge origin/master`” or “`git rebase origin/master`.” Note that the `merge` and `rebase` commands don't use network, so they are faster than `pull`.

# Rewriting history

Always use the `-i` (`--interactive`) option, unless your are _very_ confident on what is going to be done.

It will lanch an editor, showing you what it plan to do… and let you change everything! Move a commit just before an other one, change some commit messages, merge a fix commit in its parent… Big powers.

Note: to cancel the rebase, _do not just quit the editor!!!_, clear the file and _save_, then quit. See the following section on editors.

Oh, and there are other nice options, like the `-p` which try to preserve merges… Pure awesomeness!

Warning: the “`git rebase`” command _changes the history of your branch_, so you can loose some informations (like a commit…).

A good habit when you want to do something tricky is to create a new branch, and do you you stuff here. When you've finish (or if something gone bad), just can compare with the original, untouched, branch. In case of problem, just come back to you original branch and delete this one. Magick.[^rebaserollback]

[^rebaserollback]: FYI there are other way to retrieve the previous state (reflog, ORIG_HEAD, …). Creating a new branch helps to stay confident and to compare branches in your favorite viewer (`git log`, `tig`, `gitk`, `gitx`, …).

# General

## Learn the Git internals

Understanding how Git manages commits will help you _a lot_ to understand commands[^internals].

What ever, Git internals is a cool project, it would be a shame to miss it ;-)

[^internals]: It is especially true for tags, branches and remotes (fetch/push/branch/merge/rebase).

## Editor

When Git launch your favorite editor, it starts by creating a file (in the $GIT_DIR) then open it and wait (fork-join style). When the editor is closed, Git looks at the file and do what it has to.

So quitting the editor won't say “No I don't want your file” but “Ok it perfect, use it like this.”

In the same way, if you use a graphical editor (like Sublime Text) make sure it won't be open in background. Every real text editor allow that.

_Emptying the file is the standard way to say “Please stop”_.

This is the way used by commit, rebase, tag, …

## Aliases

A  way to make you life easier, _without lost power_ is to create aliases.

This is pretty simple: you can create them directly in command line (“`git config --global alias.<name> <full command>`”) or by editing the `~/.gitconfig` file.

There are 2 types of aliases, the simple ones and the full commands. The simple ones are… well… simple. They are not recursive (can't do “`rb=rebase`” then “`rbi=rb -i`”) and limited to git commands. The full commands begin by a “`!`” and are shell commands (executed using `sh`). Of course these aliases can be recursives… because you can (and usually do) call git with these commands.

~~~bash
git config --global alias.rbi "rebase -i"
git config --global alias.x '!gitx'
~~~

Very important rule: **don't let aliases hide make you forget what you are doing**.

For instance, the alias “`ci=commit -a`” is dangerous: you'll soon forget that you're using this stupid “`-a`,” and you won't understand what's going on when you will be on a different system (or if you loose your conf). The alias “`fetchap=fetch --all --prune`” is fine: the “`ap`” part will make you notice what you're doing.

Exemples: I personally use “`me=merge`” “`meff=merge --ff-only`” and “`menf=merge --no-ff`.” This way allows me to save keystrokes while still have a full control of my workflow.

## Workflow

You should rarely use “`git merge`” without options. It is not a _bad_ command, but it tries to be smart, whereas you just want it to do _exactly_ what you want, like a merge (with a merge commit), a fast-forward merge (without merge commit) or a rebase.

Use “`git merge --no-ff …`” when merging a feature branch, and “`git rebase -i …`” when syncing (from a remote). This way gives you the control of what's going on.

## Working tree

There is a _important_ thing to remember: as soon as a file is committed, it is saved for at least 30 days.

There is an other _very important_ this to remember: until a file is committed, Git known nothing about it, so it can be lost.

So, to be clear and short, __never let a file uncommited for a long time__.

If you don't want it to be committed _now_, because you are waiting for [something really important here], then here a trick you could use:

~~~bash
git add . && git commit -m "Save in reflog" && git reset HEAD~1
~~~

For instance this command creates a new commit with every non-commited changes, then goes back and cancel this commit. Because it uses “`git reset`” (and _not_ the `--hard` option) every changes are still here. So we basically did nothing. But now, try this:

~~~bash
git reflog
~~~

Here is our cancelled commit. And because it references its parents, __the entire history__ has been saved, until it will be garbage collected. The default value makes the GC only delete objects older than 30 days, so this commit is basically here for one month… How cool :-)

Oh, but wait, if you really keep some file for some days without committing them, maybe you should consider using branches.

## Branches

Cool transition, isn't it?

Short presentation: a branch is a pointer to a commit. That's it.


~~~bash
cat .git/refs/heads/master
~~~

## Stash

The stash is a super cool tool with only one rule: __if you plan to keep a stash more than 5 minutes, use branches instead__.

There is almost only one real use case: _temporary_ cleaning your working directory, to launch some commands (testing the old comportment, generating documentation, etc) and then going back to  normal.

If you want to start a new feature or to change branch in order to [whatever you put here], then use branches.

FYI stashes are not saved in reflog, so when a stash is cleared, its changes are lost forever (unless they have been committed).

Frankly, I'll never repeat this enough, use branches.

# Bonus: Git's guts

Git's model is easy to picture for a developer : there are objects and references to these objects. Objects can reference other objects, and when an object is not anymore referenced it is garbage collected. That's it.

## Objects

They are 4 types of objects : blobs, trees, commits and tags (for those with message and/or signature).

When Git create a new object, it builds it then computes its SHA1 sum, which will be it's ID. Each object is stored gzipped in a file based on its ID (in `$GIT_DIR/objects`, using the two first digits for the directory name and the 30 others for the filename). So if you have a SHA1, you can easilly find the find and inspect it.

A blob represents a file's content. Only the content, doesn't care about the name of permissions of the file. If many files have the same content, only 1 blob object will be created. It sounds like a stupid optimization at first, but it will become essential later.

A tree represents a directory. Yes, it reminds the unix FS model (don't forget it has been created by the Linux's creator). It contains a list of names, with permissions and ID of blobs and trees.

A commit represents… a commit (tadam!). It contains basically the informations about the author (name, date, etc), the IDs of parents, the ID of the three object and the commit message. Easy. Note that it does not contains a diff, but a real tree. So now the “one content is stored only one time” become really interesting: even if a new tree is created, only updated files will lead to new blob. We have both full-informations-at-any-time _and_ not-too-much-space-wasted. But there's more on this later.

Oh, and the tag object is a bit like a commit, it contains some meta-informations (date, etc), the ID of the tagged commit, a message and (possibly) a cryptographic signature.

I didn't talk about stuffs like symbolic links etc, but it's no really important here.

See? I said a word about tags but I didn't talk about branches. Let see them.

## Refs

Refs are the reference to object, nothing more. In the Git world, it's just a file containing the SHA1 sum of a commit (not that it could be any other object type).

These files are stored in the `$GIT_DIR/refs/` directory, and are organized depending of their type (`heads` for branches, `tags` for tags, `remotes` for references fetch from remote repositories, etc).

Actually their full names are the one of the path (with `$GIT_DIR` as root), so you should refers as `refs/heads/develop`. But Git allows you to skip the `refs/heads/` part for branches and the `refs` part for others refs…

For instance, you super tag “v1.0” is just a file (`$GIT_DIR/refs/tags/v1.0`) containing either the SHA1 sum of the tagged commit or the SHA1 of the tag object (which references the commit) if any. In the same way, the `master` branch is nothing more than a `$GIT_DIR/refs/heads/master` containing the SHA1 of the commit.

You totally can create new branches by simply create a new file with the valid SHA1 sum in it.

They are some magick refs, managed by Git but that you can use (read only):

- HEAD : the current commit
- ORIG_HEAD : the previous HEAD (before a merge, a rebase, …)
- FETCHED_HEAD and some other

If you are in a branch, your HEAD contains the name of the branch (in the form: “`ref: refs/heads/master`”).

## Packs

Ok, I lied a bit when talking about objects and refs: they are not always stored in files. In fact, at some times Git takes some objects and put them into pack files. They are still files stored in `$GIT_DIR/objects`, but this time many objects live in one file. A cool effect of this is that Git can use smart diff when packing objects, so the effective used size if really small. A full Git repository's size is comparable to a SVN's working directory one.

It works the same way for refs, unused refs are moved into the `$GIT_DIR/packed-refs` file, but this one is not compressed, so you can play with it directly with your favorite editor[^gutspackeditor].

[^gutspackeditor]: Ok, I know, _real_ editors handle compressed files. But hey, the effect wouldn't be so cool if I'd say “this file is not compressed, which is not so important because you could edit it anyway.”

## Index

The index is a sort mystical file containing enough information to  create a new tree object. That's it. A new tree.

Creating a commit is in fact “telling the index to create a new tree then use its SHA1 when creating a commit object.”

## Operations.todo

- init
- add
- commit
- branch - tag
- checkout
- merge
- rebase