# Git way of working
Almost everyone have their way of working with git. All have their tweaks and
tricks, some are using git integrated in a IDE and others use the command line,
some have multiple git aliases and other are using just the default
functionality. This scattered way of working can make the git history chaotic
and hard to read.

Bellow is a try to present a simple way of working that have helped me and
many others to remove some of the burdens of gits all commands and secure a neat
and tidy git history.

## Commit messages
This is a highly opinionated topic which differs a lot from developer to
developer, team to team and company to company. It is important that everyone have
agreed to a single way of working. Conformity makes the git history easier to 
follow for everyone.

Remember you do not write the commit message for your present you, it's for
everyone else and your self in the future.

I recommend everyone to read the article on the subject written by [Chris Beam](https://chris.beams.io/posts/git-commit/#imperative)
you will get a good head start on how to write commit messages and why it is
important.

## Update your local branch
With bitbucket, gitlab and github default way of working it is hard to reduce
the amount of merge commits and if developers use `git pull` as the default
update method of their feature branches it will create merge commits all the
time and it will be a pain to read the git history.

A better way is to use `git fetch` and then `git rebase`. This will not
create any unwelcome merge commits and allows for a quick look into
the remote branch before you rebase.

`git fetch` will update the remote branches (origin/*) in your local repo.

`git rebase` will fast-forward the current local branch.

## Fixup way of working
Using `git commit --fixup=<hash>` helps a lot when adding more content to a
commit in the git history and do `git rebase --autosquash` before pushing to
squash the `fixup` commits.

There are a good reason to use this when fixing comments on a Pull Request.
There wont be any commits merged just for fixing comments in your
PR, this will ensure a cleaner git history and thus make it much easier to
read.

### Example workflow
0. Create local branch `git checkout -b feature_x origin/dev`
1. Do some work and create commits
2. Push your feature branch `git push origin HEAD:feature_x` and create the PR
3. Wait for comments...
4. Fix comments `git commit --fixup=<hash>`
5. When finished fixing comments run `git rebase --interactive --autosquash`
6. Push with force `git push --force` since you rewrote the history

## When `fixup` won't work
There are cases when the `fixup` wow will not work, then you have to go back to
old style `git rebase --interactive` and edit the commit or commits that you
have comments on.

The most common case is when the file you have comments on has been updated in
several commits and the update you want to do is in a early commit. If you
would use the `fixup` wow you'll move all the changes done later into the
earlier commit.

### add --patch
When your change in a file have different context and should be in separate
commits, there's a great addition to the `git add` [command](https://git-scm.com/docs/git-add#Documentation/git-add.txt---patch), `--patch | -p`.
This will give you an interactive add where you can add small hunks of code or split
them up to be even smaller and by that make it possible to have separate commits
for the context differences.

Usage example `git add --patch /src/path/to/file`
use h to get the help menu or y or n to pick or discard a hunk.

## Update, split and do magic with commits
TBD

## Some handy git aliases
```
[alias]
	st = status -u
	co = checkout
	br = branch

        # dumps git log as oneliner to stdout
	lo = !git --no-pager log --oneline

        # get all files in a commit, defaults to HEAD
        files = "!f() { git diff-tree --no-commit-id --name-only -r ${1-HEAD}; }; f"

        # get all commit from HEAD to tracking branch HEAD or supplied branch
	one = "!f() { git --no-pager log --oneline ${1-$(git rev-parse --abbrev-ref --symbolic-full-name '@{u}')}..HEAD; }; f"

        # for dev staging branch
	dev = "!f() { git --no-pager log --oneline ${1-origin/dev}..HEAD; }; f"
```
