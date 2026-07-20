# 🐥 **<span style="text-decoration: double underline; color:rgba(10,130, 250)">Chapter 1: The Repository and the Commit</span>**

**Author:** Ángel F. Caravaca   
**Date:** 18/07/2026

---

In the previous chapter we configured an identity and a signature, both described as something that Git stamps onto your commits. However, we never explained what a commit actually is. Now that we have the credentials to "speak" with GitHub, and an identity that will be associated to you, what is left is the work itself, i.e., what Git record, and where it keeps it.

## 📂 **<span style='color:rgba(10,130,250)'><u> What is a repository? </u></span>**

A repository is just a directory in your tree that storages the special folder ```.git```. There is nothing else to it: no registry entry, no background service, no hidden index elsewhere on your machine. If that folder is present, the directory is a repository.If you delete it, the directory instantly becomes an ordinary folder again, with your files untouched but their entire history gone.

You create it with a single command, executed from inside the directory you want to track:

```bash
cd my-project
git init
```

Git replies with something like `Initialized empty Git repository in /home/user/my-project/.git/`, and from that moment every command you run inside that directory (or any of its subdirectories) is interpreted against that repository. 

>[!NOTE]
> This repository is created on your local machine, meaning it has not been uploaded to GitHub yet. To upload it, follow the specific process detailed later.

This is worth stressing: Git locates the repository by walking *upwards* from your current directory until it finds a `.git` folder. That is why `git status` works the same whether you run it at the root of the project or three levels down inside `images/`.

Let us look inside the folder that was just created:

```bash
ls -a .git
# HEAD  config  description  hooks/  info/  objects/  refs/
```

Two of these entries carry most of the weight, and both will reappear throughout this collection:

- **`objects/`** is the database. Every file content, every directory listing and every commit you ever create is stored here as an immutable object. This is where your history physically lives.
- **`refs/`** holds the pointers, i.e., human-readable names such as `main` or `v1.0` that point at objects inside the database. A branch, as we will see in Chapter 3, is little more than a file in here containing a single identifier.

The remaining entries are supporting infrastructure: `config` stores the repository-local configuration (this is exactly where a non-`--global` `user.email` from Chapter 0 ends up), `HEAD` records where you currently are, and `hooks/` contains scripts that can run automatically before or after certain operations.

Two consequences follow from this design, and they are the reason Git behaves so differently from older version control systems:

1. **The history is local.** Cloning a repository copies the whole `objects/` database to your machine, so you can inspect the full history, create branches and commit while completely offline. GitHub is not the source of truth; it is one more copy of the same database that happens to be reachable by everyone.
2. **The repository is self-contained.** Moving the directory elsewhere, or renaming it, changes nothing, because all the metadata travels inside `.git`. There are no absolute paths tying the project to a location on disk.

> [!WARNING]
> Never run `git init` inside a directory that is already tracked by another repository, and never create a repository in your home folder by accident. In the second case Git would attempt to track every file in your account, which is both slow and confusing. If it happens, `rm -rf .git` in the wrong place cleanly undoes the mistake, but be certain of the directory you are in before running it.

## 🌎 **<span style='color:rgba(10,130,250)'><u> The three areas of git </u></span>**

Once the repository exists, the natural question is how a change travels from your editor to the history. Git does not take you from one to the other in a single step: it interposes an intermediate area, and understanding it is the single most useful thing you can learn about the tool. Every file in your project is, at any given moment, in one of three areas:

- **Working directory:** the files as they exist on disk, exactly what your editor opens and saves. This is the only area you manipulate directly, and it is inherently unstable, since you break things here while you experiment.

- **Staging area (or *index*):** a draft of your next commit. When you mark a change as ready, Git copies its current content into this area and holds it there. Nothing is permanent yet, and you can add to it, remove from it or rebuild it as many times as you wish.

- **Repository (history):** the sequence of commits stored inside `.git/objects`. Once a change lands here it is permanent, immutable, and safe to share.

The two commands that move work between these areas are the ones this chapter is named after:

```
working directory  ──[ git add ]──▶  staging area  ──[ git commit ]──▶  repository
```
The obvious objection is why the middle area exists at all, since a single command could take the files straight from disk to the history. The reason is that **what you have changed and what you want to record are rarely the same thing**. In the course of a single session you might fix a bug, correct three typos in the documentation and leave a debugging statement half-written in a third file. Committing all of that at once produces a commit that cannot be described in one sentence, and therefore cannot be understood, reviewed or reverted as a unit.

The staging area lets you compose the commit deliberately: you stage the bug fix alone and commit it, then stage the documentation and commit that separately, all while the unfinished debugging code stays untouched in your working directory. The history you produce reflects the logic of the project rather than the chronology of your afternoon, and it is worth insisting that this is a deliberate design choice, not an inconvenience. In fact, the ability to reason about history, which is what Chapter 2 relies on, depends entirely on commits being coherent units.

There is also a fourth place that files can occupy, sitting outside this flow entirely: files that Git has been told to ignore, such as build artefacts, virtual environments or credentials. They live in your working directory but Git deliberately pretends not to see them. We will deal with them in Chapter 2, when we introduce `.gitignore`.

## 💊 **<span style='color:rgba(10,130,250)'><u> Read the status of your repo </u></span>**

Since files silently move between these areas, you need a way to ask Git where everything currently stands. That is `git status`, and it is the command you will run more often than any other:

```bash
git status
```

In a repository with pending work, the output looks roughly like this:

```bash
On branch main
Your branch is up to date with 'origin/main'.

Changes to be committed:
  (use "git restore --staged <file>..." to unstage)
        modified:   Chapter01.md

Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
        modified:   README.md

Untracked files:
  (use "git add <file>..." to include in what will be committed)
        images/three-areas.png
```

Read it as a direct map of the three areas described above:

- **Changes to be committed** is the staging area. Everything listed here will be part of your next commit, and nothing else will.
- **Changes not staged for commit** is the working directory. Git already knows about these files, since they exist in the history, but the modifications you have just made have not been marked as ready.
- **Untracked files** are files Git has never seen before. They are in your directory but not under version control at all, and Git will keep ignoring them until you `git add` them for the first time.

The distinction between the last two categories is the one beginners tend to miss. *Modified* means "I know this file and its content has changed"; *untracked* means "this file has never existed as far as my database is concerned". A modified file already has a history you can consult; an untracked one has nothing, and would simply vanish if you deleted it.

Notice as well that the output is not merely descriptive but instructional, as each section tells you the command that undoes or advances it. This is worth exploiting rather than memorising commands blindly.

For a compact view once you are comfortable with the concepts:

```bash
git status -s       # short format
# M  Chapter01.md   ← first column = staging area, second = working directory
#  M README.md
# ?? images/three-areas.png
```

The two-column layout is a literal rendering of the model: the left column reports the staging area and the right one the working directory. A file appearing as `MM` has been staged and then modified again afterwards, which means the version you are about to commit is *not* the one currently on your disk.

> [!NOTE]
> Running `git status` before every `add` and before every `commit` costs nothing and prevents the two most common accidents in day-to-day work: committing a file you did not intend to include, and believing you committed something that in fact never left your working directory.


## 💾 **<span style='color:rgba(10,130,250)'><u> Add vs Commit</u></span>**

With the model in place, the two commands become unambiguous. `git add` selects, whereas `git commit` records.

### `git add`

This command copies the current content of a file into the staging area:

```bash
git add Chapter01.md          # a single file
git add images/               # every file inside a directory
git add .                     # everything in the current directory and below
git add -p                    # interactively, hunk by hunk within a file
```

Three points deserve attention here.

First, `git add` is also what starts tracking a file. For an untracked file the command means "begin versioning this", and for an already tracked one it means "include these modifications in the next commit". It is the same command doing what is conceptually the same job: *this content should be part of the next snapshot*.

Second, **`git add` captures the content at the instant you run it**, not a live reference to the file. If you stage a file and then keep editing it, the staging area still holds the earlier version, which is exactly what the `MM` status above was telling you. Running `git add` again overwrites the staged copy with the newer content.

Third, `git add .` is convenient and, for that reason, the usual source of accidental commits: configuration files, notes, large binaries. The habit that avoids this is not to stop using it, but to run `git status` immediately afterwards and read what you actually staged. The `-p` variant is the opposite extreme and genuinely useful when a single file contains two unrelated changes, since it walks you through the file fragment by fragment and asks whether each one should be staged.

### `git commit`

This command takes whatever is in the staging area and writes it permanently into the repository:

```bash
git commit -m "Explain the three areas of Git"
```

The `-m` flag supplies the message inline. Without it Git opens your configured editor, which is the better option whenever the change deserves more than one line, since it lets you write a short subject line, leave a blank line and then explain the reasoning underneath.

The message is not decoration. Six months from now, `git log` is what you will use to reconstruct why a decision was made, and a history full of `fix`, `changes` and `asdf` is functionally the same as no history at all. Two conventions carry most of the value:

- Write the subject line in the imperative mood, describing what the commit does to the codebase: `Add SSH flowchart`, not `Added SSH flowchart` or `Now the flowchart is there`. This matches the phrasing Git itself uses in the messages it generates for merges and reverts.
- Explain the *why* in the body, not the *what*. The diff already shows what changed, and it does so more precisely than any prose could; what it cannot show is the reason.

There is one shortcut worth knowing, and one caveat that comes with it:

```bash
git commit -am "Fix broken link in README"
```

The `-a` flag stages every **already tracked** modified file and commits in one step. It does not touch untracked files, so a newly created file will be silently left out. It is a reasonable shortcut for small, self-contained edits, but it deliberately bypasses the staging area, so it should not become your default.

> [!NOTE]
> A commit is local. Nothing has been sent anywhere, and `git commit` does not contact GitHub in any way, so it works perfectly with no internet connection at all. Publishing the commits is the job of `git push`, which belongs to Chapter 6, when we introduce remotes.


## 🎡 **<span style='color:rgba(10,130,250)'><u> What is a commit? </u></span>**

We can now answer the question the chapter opened with. A commit is a **complete snapshot of your project at a moment in time**, together with the metadata that explains its origin.

This is the point where most mental models built on other tools break down. It is tempting to picture a commit as a set of differences applied on top of the previous one, since that is how `git diff` and `git log -p` present it, and it is how older systems genuinely worked. Git does the opposite: **each commit stores the full state of the tree**, not the changes leading to it. The diffs you see on screen are computed on demand by comparing two snapshots, and are not what is stored on disk.

Concretely, every commit object contains:

- A reference to the complete tree of files as it stood at that instant.
- The identifier of its **parent** commit, i.e., the state that immediately preceded it.
- The **author** and the **committer**, each with a name, an email and a timestamp. These are precisely the values you configured in Chapter 0.
- The **message** you wrote.
- The **GPG signature**, when the commit is signed.

Hash all of that together and you obtain the commit's identity, the forty-character SHA-1 you see in `git log`:

```bash
commit f19bb23a8c4d5e6f7a8b9c0d1e2f3a4b5c6d7e8f
Author: Ángel F. Caravaca <afdezcaravaca@gmail.com>
Date:   Fri Jul 18 11:25:00 2026 +0200

    Rename chapters with zero padding for correct ordering
```

That identifier is not a sequential number assigned by Git; it is a cryptographic hash **derived from the content itself**, which has three consequences that explain much of Git's behaviour:

1. **Commits are immutable.** Changing anything at all, i.e., a single character of the message, one byte of a file or the timestamp, produces a different hash, and therefore a different commit. This is why Chapter 8 is titled *rewriting* history rather than *editing* it: you never modify a commit, you create a replacement.
2. **The history is a chain, and tamper-evident.** Since each commit records its parent's hash, and its own hash is computed over that reference, altering any commit invalidates the identifier of every commit that follows it. You cannot quietly rewrite the past.
3. **Identical content produces identical objects.** Storing complete snapshots sounds wasteful, but a file that has not changed between two commits is not stored twice: both trees simply point at the same object, since the content, and therefore the hash, is the same. Only what actually changed occupies new space.

The parent reference also explains a fact we will rely on repeatedly. Because a commit points backwards at its parent, and a commit can have more than one parent (which is exactly what a merge is, as we will see in Chapter 4), the history is not a straight line but a **directed acyclic graph**. Branches, merges and rebases are all operations on that graph, and none of them will seem mysterious once you accept that a commit is a node holding a snapshot and a pointer to where it came from.

One last practical note. A commit records the state of the tree, so it also records the *absence* of a file: deleting a file and committing produces a snapshot in which the file does not appear, while it remains perfectly recoverable from every earlier commit. Nothing is ever lost as long as it was committed at least once, and that guarantee is what makes it safe to experiment. It is also the reason for the habit worth adopting from now on: **commit early and often**. A commit is cheap, entirely local, and is the only thing that turns work into something Git can restore.


## 🧭 **<span style='color:rgba(10,130,250)'><u> Summary </u></span>**

| Command | What it does |
| ------- | ------------ |
| `git init` | Creates the `.git` folder, turning the directory into a repository |
| `git status` | Shows where each file stands across the three areas |
| `git status -s` | The same, in a compact two-column format |
| `git add <file>` | Copies the current content of a file into the staging area |
| `git add -p` | Stages changes selectively, fragment by fragment |
| `git commit -m "..."` | Writes the staging area into the history as a new commit |
| `git commit -am "..."` | Stages every tracked modified file and commits in one step |

In the next chapter we will start reading and navigating the history we have just learned to create, and we will see what `HEAD` actually is and how to undo work at each of the three stages.
