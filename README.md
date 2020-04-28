# tipsgit
# Tips use Git command

# A better setup for Git

Git default configuration is good but it can be personalized to improve your workflow efficiency. 
Here are some good lines to put in your `~/.gitconfig`:

```ini
# The basics, who you commit as:
[user]
  name = dev
  email = dev@gmail.com
# Your Github username
[github]
  user = githubusername
# Some aliases to save 1000s keystrokes each year:
[alias]
  log = log --color
  co = checkout
  br = branch
  ci = commit
  st = status
  # Long but worth it, gives you output like: 
  # * 4be77ea  Add issue 42. 4 weeks ago by dev
  lg = log --graph --pretty=format:'%Cred%h%Creset %C(yellow)%d%Creset %s %Cgreen%ar%Creset by %C(yellow)%an%Creset' --abbrev-commit
  # Convenient to see diff in minified files
  dw = diff --color-words
# Add colors
[color]
  ui = true
  diff = auto
# Avoid messy merge commits with autorebase
[branch]
  autosetuprebase = always
# Push the current branch by default
[push]
  default = current
# Guess what you really meant
[help]
  autocorrect = 1
# Tell git you have a global .gitignore
[core]
  excludesfile = ~/.gitignore
# Remove usage hints
[advice]
  statusHints = false
# Tell where are the diff from,
# instead of using a and b notation
[diff]
  mnemonicprefix = true
  ```

# 5 Tips with git

**1\. Fetch a file from another branch without changing your current branch:**
```bash
$ git checkout other_branch -- path/to/file 
```

**2\. See what branches are merged:**
```bash
$ git checkout master 
$ git branch --merged
```
Use `--no-merged` to see the branches not yet merged.

**3\. Have Git auto-correct you:**
```bash
$ git config --global help.autocorrect 1
```
Example:
```bash
$ git comitm
WARNING: You called a Git command named 'comitm', which does not exist. 
Continuing under the assumption that you meant 'commit'
```

**4\. Show word changes in `diff`:**
```bash
$ git diff --word-diff 
```

**5\. Set remote branch tracking by default:**
```bash
$ git config --global push.default tracking
```

# How to rename a branch in Git?

Rename your local `tester` branch with `dev`:
```
git branch -m tester dev
```

Remember this will add the new branch with you `push`, but it won’t delete the old `tester` remote branch.

Add `-f --mirror` to rename the branch on the remote:
```
git push -f --mirror
```

If you just want to remove your remote tester branch, just use:
```
git push origin :tester
```

# Git stash

Ever started working on the wrong branch with Git? Use `gist stash`! Here is how to proceed in 3 steps:

**1\. Use stash to detach unstaged changes:**
```bash
$ git stash
```
**2\. Change branch:**

```bash
$ git checkout my-branch
```

**3\. Retrieve your changes as you left them:**
```bash
git stash pop
```

Git `stash` can do much more, see its options with `git stash help`.

# How to change your commit messages in Git?

At some point you’ll find yourself in a situation where you need edit a commit message.
That commit might already be pushed or not, be the most recent or burried below 10 other commits, but fear not, git has your back 🙂.

## Not pushed + most recent commit:
```bash
git commit --amend
```
This will open your `$EDITOR` and let you change the message. Continue with your usual `git push origin master`.


## Already pushed + most recent commit:

```bash
git commit --amend
git push origin master --force
```
We edit the message like just above. But need to `--force` the push to update the remote history.

⚠️ **But! Force pushing your commit after changing it will very likely prevent others to sync with the repo, if they already pulled a copy. You should first check with them.**

## Not pushed + old commit:
```bash
git rebase -i HEAD~X
# X is the number of commits to go back
# Move to the line of your commit, change pick into edit,
# then change your commit message:
git commit --amend
# Finish the rebase with:
git rebase --continue
```
Rebase opened your history and let you pick what to change. With edit you tell you want to change the message. Git moves you to a new branch to let you --amend the message. git rebase --continue puts you back in your previous branch with the message changed.

## Already pushed + old commit: 
Edit your message with the same 3 steps process as above (`rebase -i`, `commit --amend`, `rebase --continue`).
Then force push the commit:
```bash
git push origin master --force
```

⚠️ **But! Remember re-pushing your commit after changing it will very likely prevent others to sync with the repo, if they already pulled a copy. You should first check with them.**

# How to merge commits in Git?

Sometimes, you commit something and realize it needs some more fixing, and then some more.
Resulting in 3 or more commits for the same task.

You like to keep your `git log` clean and want to merge your commits. Here is how to do just that:


**Suppose we have this history:**
```bash
$ git log --oneline 
2750b94 Finally fix bug #84. 
da52aaa Fix bug #84. 
31f5476 Fixing bug #84. 
```

**We enter rebasing with:**
```bash
$ git rebase --interactive HEAD~3
# HEAD~3 says from HEAD include 3 commits
```

**Your `$EDITOR` will open with:**
```bash
pick 31f5476 Fixing bug #84. 
pick da52aaa Fix bug #84. 
pick 2750b94 Finally fix bug #84. 
```

**Update the file as follow:**
```bash
pick   31f5476 Fixing bug #84. 
squash da52aaa Fix bug #84. 
squash 2750b94 Finally fix bug #84.
```
Then quit and save, you will be prompted to enter a new commit message.

**Your new log history will be:**
```bash
$ git log --oneline 
38c0f13 Your new commit message about bug #84.
```

# Understand what `.gitignore` rule is ignoring your files

Ready to commit, you fire your `git status` and you don’t see your files 🤔.

Use this command to ask Git what rule is causing this ignore:
```bash
$ git check-ignore -v filename
```

For instance to see what rule ignores tests.pyc:
```bash
$ git check-ignore -v tests.pyc
.gitignore:1:*.pyc    tests.pyc
```
Here we see it’s the first line of the .gitignore file that is in effect. 
You can also use `git status --ignored` to show the files being ignored.


# How to grep search committed code in git?

Search working tree for text matching regular expression regexp:
```bash
git grep regexp 
```

Search working tree for lines of text matching regexp A or B:
```bash
git grep -e A --or -e B 
```

Search working tree for lines of text matching regexp A an B:
```bash
git grep -e A --and -e B 
```

Search all revisions between rev1 and rev2 for text matching regular expression regexp:
```bash
git grep regexp $(git rev-list rev1..rev2)
```
# How to update a GitHub forked repository?

So you hit "Fork" and now you have this repo copy on your Github account. Here is what to do to keep it up-to-date with the original repo.

**1. Add the original repo as remote, here called upstream:**
```bash
git remote add upstream https://github.com/author/repo.git
```

**2. Fetch all the branches of that upstream remote:**
```bash
git fetch upstream
```

**3. Merge an upstream branch:**
```bash
git merge upstream/master
```

# How to generate and apply patches with git?

It sometimes happen you need change code on a machine from which you cannot push to the repo.
You’re ready to copy/paste what `diff` outputs to your local working copy.

You think there must be a better way to proceed and you’re right. It’s a simple 2 steps process:

**1\. Generate the patch:**
```bash
git diff > some-changes.patch
```

**2\. Apply the diff:**

Then copy this patch to your local machine, and apply it to your local working copy with:
```bash
git apply /path/to/some-changes.patch
```

# Export your repo with `git archive`

See how to use `git archive` to export your repository.
For instance to get a copy of the code at a given commit, tag or branch.
Note: it will not include the .git folder, just the code.

**Create a Zip archive of the latest commit on the current branch:**
```bash
$ git archive -o latest.zip HEAD 
```

**Exporting the branch dev as a tarball:**
```bash
$ git archive -o branch_dev.tar dev
```
The filetype is inferred from the filename, one of `.tar`, `.tgz`, `.tar.gz`, `.zip`.

**By default git outputs on stdout convenient for piping to anything:**
```bash
$ git archive master | bzip2 | ssh user@host "cat > master.bz"
```
# Where to find changes due to git fetch

`fetch` is useful to pull changes from an upstream repo to merge some changes.
By default `git fetch upstream` will update or create all the remote branches locally.

Say you have `upstream/master` tracked by your local `master` branch.
Here is how to see the changes between both:

```bash
git log master..upstream/master
```
