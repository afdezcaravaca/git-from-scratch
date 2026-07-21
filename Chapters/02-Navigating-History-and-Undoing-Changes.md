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

>[!TIP]
> When you run `git log`, keep in mind that:
> - Press `q` to exit.
> - Press `Space` to move one page.
> - Press `b` to return one page.
> - `↑` / `↓` to advance one line.
> - `g` / `G` to go to the beginning or the end.
> - `/text` to search a specific word or text. In this option, press `n` / `N` to move to the next or the previous match. (`/New function`)