# Git way of working
Almost everyone have their way of working with git. All have their tweaks and
tricks, some are using git integrated in a IDE and others use the command line,
some have multiple git aliases and other are using just the default
functionality. This scattered way of working can make the git history chaotic
and hard to read.

Below is an attempt to present a simple workflow that has helped me and many
others reduce the complexity of Git's numerous commands and maintain a clean,
organized Git history.

## Commit messages
This is a highly opinionated topic that varies greatly between developers,
teams, and companies. It is crucial for everyone to agree on a unified
workflow, as conformity makes the Git history easier for everyone to understand
and follow.

__Remember, commit messages are not written for your present self. They’re for
everyone else and your future self.__

I recommend everyone to read the article on the subject written by [Chris Beam](https://chris.beams.io/posts/git-commit/#imperative)
you will get a good head start on how to write commit messages and why it is
important.

## Update your local branch
With bitbucket, gitlab and github default way of working, it is hard to reduce
the amount of merge commits, and if developers use `git pull` as the default
update method of their feature branches, it will create merge commits and it
it is not easy to follow and read the git history.

A better way is to use `git fetch` and then `git rebase`. This won't create any
unwelcome merge commits, and also allows for a quick look into the remote
branch before you update your branch with `git rebase`.

`git fetch` updates the remote branches (origin/*) in your local repo.

`git rebase` applies your commits on top of the branch you are
rebasing onto.

## Fixup way of working
Using `git commit --fixup=<hash>` helps a lot when adding more content to a
commit and do `git rebase --autosquash` before pushing to
squash the `fixup` commits to apply the changes.

I would argue that using the `fixup` workflow is a good practice when addressing
comments on a Pull Request. This approach prevents the creation and merging of
unnecessary `fix comments` commits, ensuring a cleaner Git history that is
easier to follow and read.

### Example workflow
0. Create local branch `git checkout -b feature_x origin/main`
1. Do some work and create commits
2. Push your feature branch `git push origin HEAD:feature_x` and create the PR
3. Wait for comments...
4. Fix comments `git commit --fixup=<hash>`
5. When finished fixing comments run `git rebase --interactive --autosquash`
6. Push with force `git push --force` since you rewrote the history

### Working in detached head
The example below will show when you are working on detached head(no local
branch created).

```
git checkout origin/main
...
# create commits
...
git push origin HEAD:refs/heads/feature_x
...
# fix comments with fixup
git add file
git commit --fixup=<hash>
...
git rebase -i --autosquash origin/main
git push origin --force HEAD:refs/heads/feature_x
```

### Local branch not based on a remote branch
Another scenario where you might want to rebase directly against `origin/main` is
when you create a local branch not based on a remote branch and later add a
tracking branch to enable a simple push. In this case, after pushing your
changes, if you add a fixup commit and then rebase, you may notice that only
the fixup or new commits made after the push are visible, rather than the full
commit history. To resolve this, you need to rebase against the branch from
which your branch originally diverged, such as `origin/main`. See the example
below.

```
git checkout -b feature_y
git branch -u origin/feature_y
...
git push
...
git add file
git commit --fixup=<hash>
git rebase -i --autosquash origin/main
...
git push --force

```

### Keep base
A useful trick when working with fixup commits and rebases is to use the
`--keep-base` option. This ensures that your branch does not update to include
changes from the base branch, but instead only applies the fixup commits. The
advantage is that when you push to the remote, the reviewer only needs to
review the fixup changes, rather than any new changes introduced by a rebase
without `--keep-base`. If you need to update your branch to the latest version,
it’s better to do so either before or after applying the fixups, if possible.

```
git rebase -i --autosquash --keep-base origin/main
```

### Push fixups to the remote
There are use-cases where pushing the fixup commits for review. This is useful
if you want the reviewer to see what changes you have made before applying
them. But this can also happen by accident, then there are one important thing
to keep in mind when you do that.

__Make sure that you have some mechanism to prevent the branch from getting
merged before the fixups is applied__

#### Fixup commits in Github
For github you can use github action plugins, or you could write your own
small script/plugin.

It is also possible to configure your github repository to only allow signed
commits, together with the `fixup` alias I have below in the
.git_aliases file, will ensure that no fixup commits get merged to the main
branch by accident.

## When `fixup` won't work
There are cases when the `fixup` wow will not work, then you have to go back to
old style `git rebase --interactive` and edit the commit or commits that you
have comments on.

### add --patch
When your change in a file have different context and should be in separate
commits, there's a great addition to the `git add` [command](https://git-scm.com/docs/git-add#Documentation/git-add.txt---patch), `--patch | -p`.
This will give you an interactive add where you can add small hunks of code or split
them up to be even smaller and by that make it possible to have separate commits
for the context differences.

Usage example `git add --patch /src/path/to/file`
use h to get the help menu or y or n to pick or discard a hunk.

## Update, split and do magic with commits

### Extract/remove file from commit
There are times where you regret adding a file to a commit. Maybe you run
`git commit -a` and now want the file removed.
There are multiple ways to solve this depending on the situation.
I mostly use three different ways.

1. Remove the file from the commit
```
git checkout HEAD~1 -- path/to/file
git commit --amend
```

2. Reset the commit and start over
```
git reset --soft HEAD~1
```

3. Remove from commit and keep the file change

   There isn't a straightforward built-in command to remove a file from a commit
   and keep the file diff unstaged, and there are cases it can be tricky.

   It can usually be done with the below command sequence.
   ```
   git reset HEAD^ -- path/to/file
   git commit --amend
   ```

   I have written a git alias that makes this quick and easy, you only need to run
   `git ef path/to/file` and the file is removed from the commit and file change
   gets unstaged.

   You can find the alias below in the link to the .git_aliases.

## Git aliases
Download the [.git_aliases](https://raw.githubusercontent.com/hk4n/hk4n.github.io/refs/heads/main/git-wow/.git_aliases) file to you home dir and add this to your `~/.gitconfig`
```
[include]
    path = .git_aliases
```
