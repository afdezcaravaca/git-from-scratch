# 🚢 **<span style="text-decoration: double underline; color:rgba(10,130, 250)">Chapter 2: Navigating History and Undoing Changes</span>**

**Author:** Ángel F. Caravaca   
**Date:** 21/07/2026

---

In the previous chapter we established what a commit is and how it moves through the three areas of Git. That gave us a way to *record* history, but not yet a way to *read* it or *undo* it. Right now every commit you have made lives inside `.git/objects`, yet we have no way to inspect it. In this chapter we will learn how to trace what we did, compare two moments in time, and undo a change without losing our work.

## 📜 **<span style='color:rgba(10,130,250)'><u> Reading the history </u></span>**

The most direct way to trace what you did in the repository is `git log`, which lists commits from the most recent to the oldest, walking backwards through the parent chain we described in Chapter 1. The command is:

```sh
git log
```
Git will instantly reply with a list of blocks like this one:

```bash
commit <hash - identifier>
Author: user.name <user.email>
Date:   full date

    commit message
```
The default view of this command, although useful, is quite simple, and it only displays the metadata of the commits. You can get more information using flags. These flags don't change what `git log` does, they only change how much of each commit is shown and how it is formatted. Some examples are:

```sh
git log --oneline           # one line per commit: short hash + subject
git log --oneline --graph   # adds a text graph of branches and merges
git log -p                  # shows the full diff introduced by each commit
git log --stat              # shows only which files changed, and by how much
```

>[!NOTE]
> VSCode offers extensions (such as Git Graph or GitLens) that display this same
> information graphically, instead of as text in the terminal. <span style='color:rgba(210,110,50)'> **For example:** </span>, the
> commit graph you get from `git log --oneline --graph` is shown visually in the
> *Graph* view of the Source Control panel. A later chapter is dedicated to these
> extensions, but it is worth knowing that these commands exist before moving on
> to the prettier versions built on top of them.

Beyond controlling the format and how much of each commit is shown, you can also narrow the history down to the commits you care about, instead of scrolling through all of it. Some flags that come in handy are:

```sh
git log -n 5                     # only the last 5 commits
git log --author="Ángel"         # commits by a specific author
git log --since="2 weeks ago"    # commits after a given date
git log -- <file>                # commits that touched this file only
git log --grep="fix"             # commits whose message matches this string
```

## 🕵 **<span style='color:rgba(10,130,250)'><u> Comparing two states </u></span>**

Where `git log` tells you *what you did*, `git diff` tells you *what you changed*, expressed as additions and deletions of lines. 

To understand how to use this command, recall the three areas from Chapter 1: working directory, staging area, and repository. The command `git diff` always compares two of them, and which two depends on the flags you give it:

| Command | Compares | Intuition |
| --- | --- | --- |
| `git diff` | Working area vs staging area | What you have edited but not staged yet |
| `git diff --staged` | Staging area vs `HEAD` | What you are about to commit, compared to your last commit in the current branch  |
| `git diff HEAD` | Working area vs `HEAD` | Everything you have changed since your last commit in the current branch, staged or not |

>[!IMPORTANT]
This is the one of the most common source of confusion for beginners: plain `git diff` does **not** show you what has changed since your last commit, it shows you what is *unstaged*. If you have already staged everything with `git add`, plain `git diff` prints nothing, even though your working directory clearly differs from the last.

Three things to keep in mind while running this command under any of its variants:

- **File header:** each file in the diff opens with the two versions being compared. Both paths are usually identical, since it is the same file at two moments in time.

```diff
--- a/File   # the old version
+++ b/File   # the new version
```
- **Line prefixes:** every line carries a one-character prefix telling you what happened to it. They are also colored to ease the comprehension. Moreover, note that there is no prefix for *modified*, since editing a line is recorded as a removal followed by an addition.
```diff
+> this line was added
-> this line was removed
>   this line is unchanged context, shown only to orient you
```

- **Block header:** each block of changes starts with a line such as `@@ -7,3 +7,4 @@`. Read it as coordinates: the block covers 3 lines starting at line 7 of the old version, and 4 lines starting at line 7 of the new one.

Once you are comfortable using `git diff` to compare your current work, it is worth knowing that the command is far more powerful than that. Any two points in the history can be compared, not just the three areas: two arbitrary commits, a commit against your working directory, or the tips of two branches.

```sh
git diff <hash1> <hash2>       # between two specific commits
git diff HEAD~2 HEAD           # between two commits ago and now (see HEAD~n below)
git diff <branch1> <branch2>   # between the last commits of two branches
```

To find the hashes you need here, run `git log`, as it is the command that tells you *what you did*, and the short hash printed by `git log --oneline` is enough to identify a commit.

>[!TIP]
> When you run `git log` or `git diff`, keep in mind that:
> - Press `q` to exit.
> - Press `Space` to move one page.
> - Press `b` to return one page.
> - `↑` / `↓` to advance one line.
> - `g` / `G` to go to the beginning or the end.
> - `/text` to search a specific word or text. In this option, press `n` / `N` to move to the next or the previous match. (`/New function`)

## 🫆 **<span style='color:rgba(10,130,250)'><u> Inspecting a single commit </u></span>**

While `git diff` compares two points, `git show` looks at a single one. It prints the metadata of a commit followed by the diff it introduced against its parent, so it behaves like a "combination" of `git log` and `git diff`. It also accepts the formatting flags of both:

```sh
git show                  # the commit HEAD points to, i.e. your last one
git show <hash>           # a specific commit: metadata + the diff it introduced
git show <hash> --stat    # the same, but only which files changed and by how much
```

In particular, I would like to mention a second use of this command, which is where it becomes genuinely handy. You can ask for the content of a single file *as it existed in that commit*:

```sh
git show <hash>:path/to/file   # the file as it was in that commit
git show HEAD::path/to/file    # the file as it was in your last commit
```

<span style='color:rgba(210,110,50)'> **For example:** </span> if you want to check what `README.md` looked like ten commits ago, you do not need to move anywhere: `git show HEAD~10:README.md` prints it straight to the terminal, leaving your working directory exactly as it is.


## 📌 **<span style='color:rgba(10,130,250)'><u> HEAD: your position in the history </u></span>**

In Chapter 1, we introduced `HEAD` as a file that tells Git which branch you are on. It is worth expanding on that definition now, because it has already appeared in previous commands, and the ones that follow are defined entirely in terms of it.

Actually, `HEAD` is a pointer to a branch that points to the last commit in that branch. You could say *a pointer to a pointer*. In short, `HEAD` contains a path to a file in `refs/heads/`, which is your current *branch*. Inside that branch file there is a hash that is related to a file in `objects/`, i.e. the last commit in that branch.

Understanding this is quite useful, as it gives you an intuition of how the previous commands work. The image below shows how Git resolves `HEAD` when you invoke it in the `git diff HEAD` command.

<div align='center'>
<img src="../images/Head.png" width=500>
</div>

As mentioned earlier, you can also control how far back you go from `HEAD`, i.e., how many commits you want to travel back in time:
<div align='center'>

| Reference | Means |
| --- | --- |
| `HEAD` | the current commit |
| `HEAD~1` (or `HEAD^`) | the parent of the current commit |
| `HEAD~2` | the grandparent — two steps back |
| `HEAD~n` | *n* steps back, following the first parent each time |
| | |
</div>

Next image provides an intuitive scheme of what `git restore` does. In particular:

- `git restore <file>`: in the scheme this is represented with the folder. Notice that it appears blue in the last commit but yellow in the working area, meaning you have modified it since you committed. If you run `git restore <folder>`, it goes back to being blue, i.e. to the state Git had recorded for it. I focus on the folder just for comprehension, but keep in mind that `git restore` only touches the paths you give it: running it on a single file leaves everything else in the working area exactly as it is. How you may feel it is as if you returned to the last commit, undoing every change you made to that file.

- `git restore --staged`: in the scheme this is represented with the excel file. Notice that the excel sits in the staging area with a red cross. This is what you get once you run `git restore --staged <excel>`: the excel, which was staged and ready to be committed, stops being there. Note that the file itself is untouched in your working area. The only thing you undo is the `git add`.

>[!NOTE]
> I hope these graphs clarify the concepts rather than confuse them. I came up with them by myself, so please feel free to provide a better representation.

<div align='center'>
<img src='../images/GitRestore.png' width=1000>
</div>

## ↩️ **<span style='color:rgba(10,130,250)'><u> Undoing changes: restore, reset, and revert </u></span>**

Git offers three distinct commands to undo something, and the reason there are three, rather than one, is that *undo* can mean different things depending on where the change lives and whether it has already been shared with others. Confusing them is the most common way to lose work by accident, so the goal of this section is to make the boundaries between them unambiguous.

The question to ask yourself before typing any of these commands is always: **which area am I trying to undo, and has this been pushed already?**

### `git restore`: undoing in the working directory or staging area

This is the narrowest and safest of the three, and it operates *before* history is involved at all, i.e., only on the working directory and the staging area.

<div align='center'>

| Command | Effect | Intuition |
| --- | --- | --- |
|`git restore <file>` | Discard changes in the working directory, back to the last commit. | It's like undoing all the changes with `ctrl + z` but faster |
| `git restore --stage <file>` | Unstage a file keeping the edits in the working directory | It just undo the `git add <file>` |
| | | |
</div>

One important thing to keep in mind is that `git restore <file>` is destructive on the working directory, since the edits you made are overwritten with the version from the last commit, with no further confirmation. In contrast, `git restore --staged`, is harmless as it only moves the file back from the staging area to the working directory, so your edits are not touched at all, only *unmarked* as ready to commit.


### `git reset`: moving the branch pointer

`git reset` operates one level up: it moves what the current branch points to, which is to say, it rewrites which commit `HEAD` (and therefore your branch) considers "the latest one". Everything after the target commit is left behind, unreachable from the branch, though not necessarily deleted yet.

To make it simpler, you travel back in time to a previous commit, and your branch travels with you. Everything you did between that commit and where you were is left behind: not erased, just forgotten by the branch, which no longer counts those commits as part of its history. From there you carry on working, and your next commits grow from that point instead, opening the line named *New timeline* in the diagram.

Next image provides an intuitive scheme of what `git reset` does. In particular:

You created the repository and started making changes and committing them. That's the whole horizontal line. At some point, you decided to undo a lot of work by returning to the fourth commit. From there your branch carries on, opening the new timeline drawn in red. Notice that all the commits between the point you were at and the commit you returned to are "forgotten": they are not erased, but your branch no longer counts them as part of its history. This is represented by the gray and dotted line.

>[!NOTE]
> I hope these graphs clarify the concepts rather than confuse them. I came up with
> them by myself, so please feel free to provide a better representation.

<div align="center">

<img src="../images/GitReset.png" width=1000>

</div>

Like every command presented so far, `git reset` accepts flags. Here they are not merely cosmetic, since the command takes a target commit and a *mode* that decides which of the three areas are dragged back along with the branch. The branch pointer always moves; what changes is whether the staging area and the working directory are rewritten to match.

<div align="center">

| Mode | Branch pointer | Staging area | Working directory |
| --- | --- | --- | --- |
| `git reset --soft` | moved | unchanged (keeps old staged state) | unchanged |
| `git reset (--mixed)` | moved | reset to match `<commit>` | unchanged. It's the default mode. |
| `git reset --hard` | moved | reset to match `<commit>` | reset to match `<commit>` |
| | | | |
</div>

