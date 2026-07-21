# <img src="https://cdn.simpleicons.org/github/ffffff" width="30"/> Git From Scratch

![Status](https://img.shields.io/badge/status-active-brightgreen?logo=git&logoColor=white)
![Git](https://img.shields.io/badge/Git-F05032?style=flat&logo=git&logoColor=white)
![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)

A hands-on collection of explanatory markdowns for learning Git (and GitHub) from the ground up. Each chapter focuses on a different aspect of Git, from initial configuration to day-to-day workflows. I will use this repository as a laboratory and playground, so any procedure can be analysed and understood simply by following the evolution of the repository. Note that this collection is aimed at me primarily, but I think it can be helpful if you are just getting started with Git.

The content was developed with the assistance of Claude (Anthropic's AI), which answered all my questions about this tool. I drew my own conclusions from its explanations, so if you spot any mistakes, feel free to open an issue or reach out :)

---

## 📚 Contents

| Chapter | Topics |
| ------- | ------ |
| [**Chapter 0: Git Configuration**](./Chapters/00-Git-Configuration.md) | Git vs GitHub · authentication (SSH/HTTPS) and identification · commit signing (GPG) |
| [**Chapter 1: The Repository and the Commit**](./Chapters/01-The-Repository-and-the-Commit.md) | `init` · the three areas (working directory / staging / history) · `add`, `commit`, `status` · a commit as a snapshot |
| [**Chapter 2: Navigating History and Undoing Changes**](./Chapters/02-Navigating-History-and-Undoing-Changes.md) | `log`, `diff`, `show` · HEAD as a pointer · `restore`, `reset`, `revert` · `.gitignore` |
| [**Chapter 3: Branches**](./Chapters/03-Branches.md) | a branch as a movable pointer · `branch`, `switch` · why branches are cheap |
| [**Chapter 4: Integrating Branches: Merge**](./Chapters/04-Integrating-Branches-Merge.md) | fast-forward vs merge commit · the commit graph (DAG) |
| [**Chapter 5: Conflicts**](./Chapters/05-Conflicts.md) | why conflicts happen · anatomy of conflict markers · resolving them |
| [**Chapter 6: Remotes: The Bridge to the Cloud**](./Chapters/06-Remotes-The-Bridge-to-the-Cloud.md) | `clone`, `remote` · `fetch` vs `pull` · `push` · tracking branches (`origin/main`) |
| [**Chapter 7: Collaboration and Pull Requests**](./Chapters/07-Collaboration-and-Pull-Requests.md) | fork · pull requests · review · the GitHub Flow |
| [**Chapter 8: Rebase and Rewriting History**](./Chapters/08-Rebase-and-Rewriting-History.md) | `rebase` vs `merge` · interactive rebase · when NOT to rewrite history |
| [**Chapter 9: Workflows and Tags**](./Chapters/09-Workflows-and-Tags.md) | Git Flow vs trunk-based · `tag` and releases |
| [**Chapter 10: Useful VSCode Extensions**](./Chapters/10-Useful-VSCode-Extensions.md) | practical extensions for day-to-day Git work |

---

## 🗂️ Repository Structure

```
git-from-scratch/
├── images/                                        # Images in the chapters
├── Chapters/
│   ├── 0-Git-Configuration.md
│   ├── 1-The-Repository-and-the-Commit.md
│   ├── 2-Navigating-History-and-Undoing-Changes.md
│   ├── 3-Branches.md
│   ├── 4-Integrating-Branches-Merge.md
│   ├── 5-Conflicts.md
│   ├── 6-Remotes-The-Bridge-to-the-Cloud.md
│   ├── 7-Collaboration-and-Pull-Requests.md
│   ├── 8-Rebase-and-Rewriting-History.md
│   ├── 9-Workflows-and-Tags.md
│   └── 10-Useful-VSCode-Extensions.md
└── LICENSE
```

---