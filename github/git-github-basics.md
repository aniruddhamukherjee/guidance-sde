# Git & GitHub Essentials: A Beginner Developer's Guide

Welcome to Git and GitHub! Version control is one of the most fundamental skills you will use every day as a software engineer. This guide breaks down the core concepts, daily commands, branching strategies, and collaboration workflows in simple, practical terms.

> [!TIP]
> Looking for step-by-step hands-on exercises and a Markdown guide? Check out the companion [git-github-practice.md](./git-github-practice.md) lab.

---

## Table of Contents

1. [Git vs. GitHub: What's the Difference?](#1-git-vs-github-whats-the-difference)
2. [The Core Mental Model](#2-the-core-mental-model)
3. [One-Time Setup](#3-one-time-setup)
4. [The Daily Local Workflow](#4-the-daily-local-workflow)
5. [Branching: Working in Isolation](#5-branching-working-in-isolation)
6. [Syncing Remote & Local: Push and Pull](#6-syncing-remote--local-push-and-pull)
7. [Pull Requests (PRs): Team Collaboration](#7-pull-requests-prs-team-collaboration)
8. [Resolving Merge Conflicts](#8-resolving-merge-conflicts)
9. [Best Practices & Pro Tips](#9-best-practices--pro-tips)
10. [Quick Command Cheat Sheet](#10-quick-command-cheat-sheet)
11. [Useful Study Resources & Links](#11-useful-study-resources--links)

---

## 1. Git vs. GitHub: What's the Difference?

A common point of confusion for beginners is treating Git and GitHub as the same tool. They are related, but distinct:

| Concept | Git | GitHub |
| :--- | :--- | :--- |
| **What it is** | A command-line version control software tool | A cloud-hosted platform built around Git |
| **Where it runs** | Locally on your computer | In the cloud (web browser / servers) |
| **Primary purpose** | Tracks history, versions, and changes to files | Enables team collaboration, code reviews, issue tracking, and CI/CD |
| **Network needed?** | No. Works completely offline | Yes. Requires an internet connection to sync |

> **Analogy**: Git is like the **engine** inside a car. GitHub is like the **highway network and garage** where many drivers share the road and coordinate.

---

## 2. The Core Mental Model

Git tracks your files across four main areas:

```
[Working Directory] --(git add)--> [Staging Area] --(git commit)--> [Local Repository] --(git push)--> [Remote (GitHub)]
        ^                                                                                                      |
        |-------------------------------------(git pull)-------------------------------------------------------|
```

1. **Working Directory (Untracked / Modified)**: The real files on your hard drive you are actively editing.
2. **Staging Area (Index)**: The draft zone where you choose *which* changes you want to package into your next snapshot.
3. **Local Repository (`.git`)**: The permanent commit history stored locally on your machine.
4. **Remote Repository (GitHub)**: The central copy hosted on GitHub so teammates can access your code.

---

## 3. One-Time Setup

Before making your first commit, tell Git who you are. This information is attached to every commit you make.

```bash
# Configure your name and email
git config --global user.name "Your Name"
git config --global user.email "your.email@example.com"

# Set the default main branch name (industry standard)
git config --global init.defaultBranch main

# Verify your settings
git config --list
```

---

## 4. The Daily Local Workflow

### 1. Download an Existing Project (Clone)
To download a GitHub repository to your computer:
```bash
git clone https://github.com/username/repository-name.git
cd repository-name
```

### 2. Check Your Status
Always check what Git sees before running actions:
```bash
git status
```
This tells you which branch you are on and which files are untracked, modified, or staged.

### 3. Stage Your Changes (`git add`)
Move specific files into the Staging Area:
```bash
# Stage a specific file
git add filename.py

# Or stage all modified and new files in the current folder
git add .
```

### 4. Inspect Differences (`git diff`)
See exactly what lines changed:
```bash
# View unstaged changes
git diff

# View changes that have already been staged
git diff --staged
```

### 5. Save a Snapshot (`git commit`)
A commit captures a permanent checkpoint in your project:
```bash
git commit -m "Add user authentication endpoint"
```

> **Writing Good Commit Messages**:
> - Use the imperative mood: `"Fix login validation bug"` instead of `"Fixed bug"` or `"Fixes"`.
> - Keep the first line concise (under 50-72 characters).
> - Describe *why* and *what*, not just file names.

---

## 5. Branching: Working in Isolation

### Why Branch?
Imagine working directly on `main` (production code) while developing a half-finished feature that breaks the build. If someone needs to deploy a hotfix, your broken code blocks everyone.

**Branches create parallel universes.** You create a branch, write and test your feature safely, and only combine it back into `main` when it is reviewed and ready.

```
       (feature-user-login)  o---o---o
                            /         \
(main)  o------------------o-----------o (merged)
```

### Common Branch Commands

```bash
# Create and switch to a new branch in one command
git checkout -b feature/login-page

# Alternatively (modern Git syntax):
git switch -c feature/login-page

# List all local branches (current branch has an asterisk *)
git branch

# Switch between existing branches
git switch main
# or: git checkout main

# Delete a branch locally after it has been merged
git branch -d feature/login-page
```

### Branch Naming Conventions
Adopt clean, descriptive branch names:
- `feature/user-profile-api`
- `bugfix/cart-discount-calculation`
- `chore/upgrade-dependencies`
- `docs/api-getting-started`

---

## 6. Syncing Remote & Local: Push and Pull

### `git push` (Upload to GitHub)
When you commit locally, GitHub does not know about it until you push:

```bash
# First time pushing a new branch:
# The '-u' (upstream) flag links your local branch to the remote branch
git push -u origin feature/login-page

# On subsequent pushes to this branch, simply run:
git push
```

### `git pull` (Download from GitHub)
When teammates push code or your PR is merged, update your local repository:

```bash
# Switch to main and get the latest changes
git switch main
git pull
```

> **Under the Hood**: `git pull` is actually shorthand for:
> 1. `git fetch`: Downloads changes from the remote without modifying your working files.
> 2. `git merge`: Integrates those downloaded changes into your current active branch.

---

## 7. Pull Requests (PRs): Team Collaboration

A **Pull Request** (PR) is a GitHub feature (not a raw Git command). It is a formal request asking your team:
> *"I have built this feature on my branch. Please review my code and merge it into `main`."*

### The Complete PR Lifecycle

```
1. Create Branch  -->  2. Commit & Push  -->  3. Open PR on GitHub
        ^                                               |
        |                                               v
 6. Delete Branch <--   5. Merge PR      <--  4. Code Review & Tests
```

#### Step 1: Push Your Branch
Push your feature branch with your commits to GitHub (`git push -u origin feature/my-feature`).

#### Step 2: Open the Pull Request
1. Go to the repository on GitHub in your browser.
2. You will usually see a banner: **"Compare & pull request"**. Click it.
3. If not, go to the **Pull requests** tab and click **New pull request**.
4. Set the **base** branch to `main` and the **compare** branch to `feature/my-feature`.

#### Step 3: Write a Clear PR Description
Explain your work to your reviewers:
- **What changed?** (Summary of feature or bug fix)
- **Why?** (Ticket/Issue reference, business motivation)
- **How to test?** (Manual steps or automated test results)
- **Screenshots / GIFs**: Extremely helpful for UI changes!

#### Step 4: Code Review and Iteration
- Teammates will comment, ask questions, or request changes.
- To update your PR: **Do not open a new PR!** Just make changes locally, commit them, and run `git push`. GitHub updates the open PR automatically.

#### Step 5: Merge Options on GitHub
Once approved and CI checks pass, the PR can be merged:
- **Create a merge commit**: Preserves every individual commit and adds a merge record.
- **Squash and merge** *(Recommended for clean history)*: Combines all your branch commits into one single tidy commit on `main`.
- **Rebase and merge**: Replays commits one-by-one onto `main` without a merge commit.

#### Step 6: Clean Up
Click **Delete branch** on GitHub, then clean up locally:
```bash
git switch main
git pull
git branch -d feature/my-feature
```

---

## 8. Resolving Merge Conflicts

### When Does a Conflict Happen?
A conflict occurs when two branches edit the exact same lines of code in a file and Git cannot decide which version is correct.

### How to Resolve a Conflict:
1. Try to pull or merge changes:
   ```bash
   git switch feature/my-feature
   git merge main
   ```
2. Git marks the conflict in the affected file:
   ```diff
   <<<<<<< HEAD (Current change on your branch)
   const apiURL = "https://api.staging.example.com";
   =======
   const apiURL = "https://api.prod.example.com";
   >>>>>>> main (Incoming change from main)
   ```
3. Open the file in your code editor (e.g. VS Code will show helpful conflict buttons).
4. Edit the file to keep the desired code and remove the `<<<<<<<`, `=======`, and `>>>>>>>` markers.
5. Save the file, stage it, and finalize the merge:
   ```bash
   git add filename.js
   git commit -m "Resolve merge conflict between main and feature branch"
   git push
   ```

---

## 9. Best Practices & Pro Tips

1. **Always Use a `.gitignore` File**:
   Never check in `node_modules/`, `.env` files, build output (`dist/`, `target/`), or API keys.
2. **Pull Before You Start**:
   Before creating a new branch, switch to `main` and run `git pull` so your branch starts from the freshest code.
3. **Keep Branches and PRs Small**:
   A PR with 50 lines changed gets reviewed in 10 minutes. A PR with 1,500 lines changed might take days and hide bugs.
4. **Never Force Push to Shared Branches**:
   Avoid `git push --force` on `main` or branches multiple people are using, as it overwrites remote history.

---

## 10. Quick Command Cheat Sheet

| Task | Command |
| :--- | :--- |
| **Check repository status** | `git status` |
| **View history log** | `git log --oneline --graph` |
| **Create and switch to branch** | `git switch -c <branch-name>` |
| **Switch to existing branch** | `git switch <branch-name>` |
| **Stage changes** | `git add <file>` or `git add .` |
| **Commit staged changes** | `git commit -m "<message>"` |
| **Push branch to remote (first time)** | `git push -u origin <branch-name>` |
| **Push further updates** | `git push` |
| **Update local branch from remote** | `git pull` |
| **Discard unstaged changes in a file** | `git restore <file>` |
| **Unstage a staged file** | `git restore --staged <file>` |
| **Delete local branch** | `git branch -d <branch-name>` |

---

## 11. Useful Study Resources & Links

Here are some of the best, battle-tested resources to deepen your Git and GitHub understanding:

### Interactive & Visual Learning
- [Learn Git Branching](https://learngitbranching.js.org/): An interactive, visual sandbox that walks you through Git branching, rebasing, and merging step-by-step.
- [Visualizing Git](https://git-school.github.io/visualizing-git/): An open playground where you can execute Git commands in a sandbox and watch the commit tree render in real-time.
- [Oh My Git!](https://ohmygit.org/): An open-source game designed to teach Git concepts through real-time visualization of internal data structures.

### Official Guides & Interactive Practice
- [GitHub Skills](https://skills.github.com/): Free, official interactive tutorials built directly into GitHub repositories (covers First Day on GitHub, Reviewing PRs, Merge Conflicts, and more).
- [Pro Git Book (Official Free Book)](https://git-scm.com/book/en/v2): The comprehensive, free official manual written by Scott Chacon and Ben Straub.
- [GitHub Official Git Cheat Sheet](https://training.github.com/downloads/github-git-cheat-sheet.pdf): A printable quick-reference guide published by GitHub.

### Troubleshooting & Getting Unstuck
- [Dangit, Git!?! (Oh Shit, Git!)](https://dangitgit.com/): Plain-English solutions for everyday Git mistakes ("I committed to the wrong branch!", "I made a typo in my commit message!").
- [Flight Rules for Git](https://github.com/k88hudson/git-flight-rules): An exhaustive reference guide for what to do when you encounter tricky or unexpected Git situations.

### Video Tutorials (YouTube)
- [Git and GitHub for Beginners - Crash Course (freeCodeCamp)](https://www.youtube.com/watch?v=RGOj5yH7evk): A comprehensive, complete beginner-friendly video walkthrough covering core Git commands, GitHub interface, and branch management.
- [Git Tutorial for Beginners: Learn Git in 1 Hour (Programming with Mosh)](https://www.youtube.com/watch?v=8JJ101D3knE): A concise, practical 1-hour video explaining the fundamental concepts, workflow, and command line tools.

