# 🏯 **<span style="text-decoration: double underline; color:rgba(10,130, 250)">Chapter 1: The Repository and the Commit</span>**

**Author:** Ángel F. Caravaca   
**Date:** 20/07/2026

---

In the previous chapter we configured and identity and a signature, both described as something that Git stamps onto your commits. However, we never explained what a commit actually is. Now that we have the credentials to "speak" with GitHub, and an identity that will be associated to you, what is left is the work itself, i.e., what Git records, and where it keeps it.

## 📂 **<span style='color:rgba(10,130,250)'><u> What is a repository? </u></span>**

A repository is just a directory in your tree that storages the special folder `.git`. There is nothing else to it. If that folder is present, the directory is a repository, whereas if you delete it, the directory instantly becomes an ordinary folder again, with your files untouched but their entire history gone.

>[!NOTE]
We are referring here to a local repository: one that lives only on your machine and has not yet been connected to GitHub (its **remote** counterpart). Keep this distinction in mind, since for now everything happens locally, with no other party involved.

To create the **local** repository, you do it with a single command, executed from inside the directory you want to track:

```sh
cd my-project
git init
```

After this, Git replies with something like `Initialized empty Git repository in <path>/.git/`. From this moment, every command you run inside that directory or any of its subdirectories is related to this `.git` folder (repository).

Now, One point worth stressing is *how Git knows it is inside a repository*, and more importantly, *which one*. Well, Git locates the repository by walking *upwards* from the current directory until it finds the first `.git` folder. Once found, the changes are instantly related to this repository.

>[!WARNING]
This is precisely why you shouldn't create a repository inside another: Git will treat the nested .git folder as ordinary content (or silently ignore it), leading to confusing, unpredictable behavior about which repository actually owns your files.


## 🌿 **<span style='color:rgba(10,130,250)'><u> The .git folder </u></span>**

Now, let us look inside the `.git` folder that was just created. Just run:

```sh
ls -a .git
```

To keep it simple:

- **`objects/`:** this is the database of the repository, thus it is the most important folder in the entire repository. Every file content, every directory listing and every commit you ever create is stored here as an immutable object. this is where your history physically lives.

- **`refs/`:** this folder holds text files, each a human-readable name that points to a commit. The typical case is a branch: a file like `refs/heads/main` whose entire content is the hash of the most recent commit on that branch. That hash is the identifier of a commit living in `objects/`. Notice that the ref doesn't hold the commit itself, only a pointer to it.  
It works like the contacts app on your phone: the entry "Mom" holds a phone number, nothing more. It doesn't store your conversations, it just tells you which number "Mom" maps to. In the same way, `refs/` doesn't store history; it only records which commit each name points to. When you make a new commit, Git rewrites the file so the name points to the new commit, and the branch moves forward on its own.  
Branches aren't the only kind of ref, though. The folder is organised into three subfolders:

| File in `refs/` | Contains | What the name means | Who moves it |
| --- | --- | --- | --- |
| `heads/main` | a commit hash | the latest commit on your local `main` | you, on every commit |
| `remotes/origin/main` | a commit hash | where `main` was on the remote last time you fetched | `git fetch` / `git pull` |
| `tags/v1.0` | a commit hash | the commit you marked as version 1.0 | nobody — it never moves |  

>[!IMPORTANT]
A tag like `v1.0` is a name you pin by hand to one specific commit to mark it as meaningful — almost always a released version. Unlike `main`, which Git created for you at your first commit, a tag exists only because you explicitly ran `git tag v1.0` on the commit you wanted to immortalise. And unlike a branch, a tag never moves: it stays nailed to that exact commit forever, so you can always return to precisely what you shipped.

- **`HEAD`:** file that indicates the branch you are working right now.

- **`index`:** this is the staging area of the repository, stored as a single file.

- **`config`:** the local configuration of the repository, such as the url to the remote repository this local repo is linked to.

- **`info/exclude`:** works exactly like `.gitignore`, but privately: it lives only in your `.git` folder, so it is never shared with anyone who clones the repository. We will cover `.gitignore` itself later in this course; think of this file as its local-only sibling.

- **`hooks/`:** scripts that Git can run automatically at specific points in your workflow, such as right before a commit or right before a push. The folder ships empty (with `.sample` examples), but this is exactly where tools like linters or formatters plug themselves in.

- **`logs/`:** keeps the *reflog*, a private record of every position `HEAD` has ever pointed to on your machine. It is not part of the project history and is never shared, but it is the safety net that lets you recover a commit even after a `reset` that seemed to erase it. More on this later in this course.

- **`ORIG_HEAD`:** a companion to the reflog. Before certain operations that can rewrite history quite drastically (`reset`, `merge`, `rebase`), Git saves your previous position here, so that `git reset --hard ORIG_HEAD` gives you a one-step way back if the operation goes wrong.

The rest (`branches/`, `description`, `COMMIT_EDITMSG`, `FETCH_HEAD`, `packed-refs`, etc) is machinery that Git manages purely for its own bookkeeping. Unlike the files above, there is no point in the course where you will need to open or reason about them; they appear and update automatically based on the operations you execute.