# Basics

## init

- Creates a `.git` folder in the project and the branch will be `master`:

```dos
git init
```

- Create a `.gitignore` file in the root of the project and put things that you don't want to be committed such as `*.log`.

## config

```dos
git config --global user.name "Ben"
git config --global user.email "Ben@test.com"
```

### SSH (secure shell) network protocol

- Open `Git Bash`.
- Execute:

```dos
ssh-keygen -t rsa -b 4096 -C "john.doe@gmail.com"
```

- Hit `Enter` for the default path to store the public key (which is used to lock).
- Add a `passphrase` so if someone gained access to your private key, he wouldn't do anything with it.
- ensure the ssh-agent is running by executing:

```dos
eval "$(ssh-agent -s)"

Agent pid 219
```

```dos
ssh-add -K /Users/you/.ssh/id_rsa
```

## status

- Shows the status:

```dos
git status
```

> Untracked -> Unstaged -> Staged -> committed

## Change status

### From `untracked/unstaged` to `staged`

- If you want to have different commits at a point in time, put different files in the `staged` status:

```dos
git add <filename>
```

- If you want to have one commit at a point in time, put all files in the `staged` status:

```dos
git add -A
```

```dos
git add .
```

- We also have `git rm <filename>` which removes the file from git and also the file system.

### From `staged` to `committed`

```dos
git commit -m "First Commit"
```

- If you want to have `multi-line` commit:

```dos
git commit
```

- and then, type in the editor (`Vi`) that will be opened and quit out of it by pressing `esc` and then typing `:wq`.

## Log and show

- Shows the log.
- HEAD means where we are here now.

```dos
git log

commit 8fa64a1aa578994050e71f106fc59d9fe28d71ae (HEAD -> master)
Author: JohnDoe <john.doe@gmail.com>
Date:   Mon Apr 27 10:38:22 2020 +1000

    commit 2

commit a58ce2331209f77107316b4a68a749e6047f5aff
Author: JohnDoe <john.doe@gmail.com>
Date:   Mon Apr 27 10:36:52 2020 +1000

    First commit
```

- With `q`, you can exit out of log.
- To see details of a commit:

```dos
git show 8fa64a1aa578

commit 8fa64a1aa578994050e71f106fc59d9fe28d71ae
Author: JohnDoe <john.doe@gmail.com>
Date:   Mon Apr 27 10:38:22 2020 +1000

    commit 2

diff --git a/a.txt b/a.txt
index 6d20e91..4fa7bd3 100644
--- a/a.txt
+++ b/a.txt
@@ -1 +1 @@
-hi there
\ No newline at end of file
+hi there modified
\ No newline at end of file
```

## Tags

- Tass are for releases.
- When we are in a state that we want to refer in the future (-a for annotate):

```dos
git tag -a v1.0.0 -m "first version"
```

- This tag is associated with the commit that we are at right now. If we wanted to tag a previous commit:

```dos
git tag -a v0.0.0 a58ce2331209f7  -m "0 version"
```

- The following will show all the tags up to the current commit:

```dos
git tag

v0.0.0
v1.0.0
```

- Now we can see commits by:

```dos
git show v0.0.0
```

## Comparing

### Shows changes (between `unstaged` and `last commit`) of a particular file

```dos
git diff a.txt

diff --git a/a.txt b/a.txt
index 3f86214..9dde370 100644
--- a/a.txt
+++ b/a.txt
@@ -1 +1 @@
-hi there modified hey hey
\ No newline at end of file
+hi there modified hey   hey
\ No newline at end of file
```

### Shows changes (between `unstaged` and `last commit`) of all file

```dos
git diff

diff --git a/a.txt b/a.txt
index 3f86214..9dde370 100644
--- a/a.txt
+++ b/a.txt
@@ -1 +1 @@
-hi there modified hey hey
\ No newline at end of file
+hi there modified hey   hey
\ No newline at end of file
diff --git a/b.txt b/b.txt
index 41f4d24..73a9270 100644
--- a/b.txt
+++ b/b.txt
@@ -1 +1 @@
-heeey
\ No newline at end of file
+heeey  s
\ No newline at end of file
```

### Shows changes (between `staged` and `last commit`) of all file

```dos
git diff --staged
```

- Of course, you can give it just one file too.

## Reset

### From `committed` to `staged`

- We should enter the commitSha that we are resetting to (it can be any previous commits):

```dos
git reset 2ebbe954815756d4a2214 --soft
```

### From `committed` to `unstaged`

- We should enter the commitSha that we are resetting to (it can be any previous commits):

```dos
git reset 2ebbe954815756d4a2214
```

### From `committed` to `another commit`

- We should enter the commitSha that we are resetting to (it can be any previous commits):

```dos
git reset 2ebbe954815756d4a2214 --hard
```

### From `staged` to `unstaged`

```dos
git reset
```

- Or, for just one file:

```dos
git reset <filename>
```

### From `unstaged` to `last commit`

```dos
git checkout -- <filename>
```

## Checkout to a commit

- You can checkout to a certain commit by the commitSha (or its tag, if it is any):

```dos
git checkout 21d09c41ddccfa3f
```

or

```dos
git checkout v1.0.0
```

- Note that in this way, you are in 'detached HEAD' state. You can look around, make experimental changes and commit them, and you can discard any commits you make in this state without impacting any branches by switching back to a branch. But if you want to create a new branch to retain commits you create, you may do so by using -c with the switch command. Example:

```dos
git switch -c <new-branch-name>
```

## Blame

- Shows history of changes of a file:

```dos
git blame <filename>
```

- If we want to be specific for a specific line:

```dos
git blame <filename> -L1
```

## Bisect

- You have a bug and you want to know which commit created that bug:

```dos
git bisect start
git bisect bad
git bisect good ef9946377a330524dd1

Bisecting: 0 revisions left to test after this (roughly 1 step)
[e2f16da1c4d13205c93cf3f801e6a6ca4f336d58] v2
```

- It automatically changes the commit, we run the program, and says is it bad or good:

```dos
git bisect bad

Bisecting: 0 revisions left to test after this (roughly 0 steps)
[9a30910b5e1af7e5918cbde20f7c2b329a34259c] final
```

- At the end, it found the culprit:

```dos
git bisect good

e2f16da1c4d13205c93cf3f801e6a6ca4f336d58 is the first bad commit
commit e2f16da1c4d13205c93cf3f801e6a6ca4f336d58
Author: JohnDoe <john.doe@gmail.com>
Date:   Mon Apr 27 14:43:17 2020 +1000
```

# Branch

## Create and checkout to a branch

![](/md/branch.png)

- To see on what branch you are right now:

```dos
git branch

* master
```

- To create a new branch:

```dos
git branch New_Branch
```

- To change branch:

```dos
git checkout New_Branch
```

```dos
git branch

* New_Branch
  master
```

- To combine creating and changing branch:

```dos
git checkout -b New_Branch
```

## Merge a branch

- First make sure that you are on the master branch:

```dos
git checkout master
```

- Then,

```dos
git merge New_Branch
```

## Delete a branch

```dos
git branch -d New_Branch
```

## Cherry pick

- Cherry picking is the act of picking a commit from a branch and applying it to another. For example, a developer working on a feature branch finds a bug and creates an explicit commit patching this bug. This new patch commit can be cherry-picked directly to the master branch to fix the bug before it effects more users:

```dos
git checkout master
git cherry-pick 4a170d2e91e770bae7a5c
```

- Now, that commit is added to the master branch.
- If we want to just add the changes to the code but not commit them right away:

```dos
git cherry-pick 4a170d2e91e770bae7a5c --no-commit
```

# Remote

## Clone

- Copy the address from github.
- Then,

```dos
git clone https://github.com/JohnDoe/Materials.git
```

## Adding remote

- Alternatively, If we wanted to add an origin to a project:

```dos
git remote add origin https://github.com/JohnDoe/x.git
```

- A project can have more than one remote. The second one should be named something else like `backup`.

## Push to origin

- After committing to the local repository, the local will be ahead of `origin/master` by 1 commit.
- Then in case of cloning,

```dos
git push
```

- Or, in case of adding (and it is the first time you want to push):

```dos
git push -u origin master
```

- Or, in case of adding (and it is not the first time you want to push):

```dos
git push
```

- Tags won't be pushed by default, to push tags separately (it will create releases):

```dos
git push --tags
```

# Workflow (work on a new feature from origin/master)

1. `git clone https://github.com/JohnDoe/Materials.git`
2. `git checkout -b Feature_branch`
3. Commit small changes.
4. `git push`
5. The first time, git will suggest: `git push --set-upstream origin Feature_branch`.
6. If this `Feature_branch` has been updated in the origin by others, you have to
   - first `git pull`,
   - fix conflicts and commit,,
   - `git push` again.
7. Go to github and click on `Compare and pull request`.
8. Add reviewer.
9. After approving changes, click on `Merge pull request`.
10. If there are conflicts,
    - in the master branch, `git pull`,
    - in the `Feature_branch`, `git merge master`,
    - fix conflicts and commit,
    - `git push` again.

# Workflow (continue to work on a feature branch)

1. `git clone https://github.com/JohnDoe/Materials.git`
2. `git checkout -t origin/Feature_branch`. Alternatively, you could `git fetch` to fetch all remote branches and then `git checkout Feature_branch`.
3. Commit small changes.
4. `git push` will push it to `Feature_branch` on remote.
5. If this `Feature_branch` has been updated in the origin by others, you have to
   - first `git pull`,
   - fix conflicts and commit,
   - `git push` again.
