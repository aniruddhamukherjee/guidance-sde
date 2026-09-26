# Markdown Guide & Hands-on Git/GitHub Practice Lab

Welcome to your hands-on workbook! This guide is split into two practical sections:
1. **Markdown (.md) Fundamentals**: What it is, how to format it, and how to preview it in VS Code and web browsers.
2. **Git & GitHub Practice Exercises**: Step-by-step, interactive challenges based on the concepts from [git-github-basics.md](file:///d:/work/code/guidance-sde/github/git-github-basics.md).

---

## Part 1: Markdown (.md) Fundamentals

### 1.1 What is Markdown?
Markdown is a lightweight markup language that allows you to format plain text using simple punctuation and symbols. It was created in 2004 by John Gruber and Aaron Swartz.

**Why developers use it**:
- **README files**: The homepage document of every GitHub project (`README.md`).
- **Pull Request & Issue descriptions**: Explaining bug fixes and features to teammates.
- **Technical Documentation**: Writing wikis, architectural decision records (ADRs), and guides.
- **Portability**: Plain text files that render beautifully across editors, GitHub, and websites.

---

### 1.2 Essential Markdown Syntax

#### Headings
```markdown
# Heading 1 (Page Title)
## Heading 2 (Main Section)
### Heading 3 (Subsection)
#### Heading 4
```

#### Text Emphasis
```markdown
**Bold text** or __Bold text__
*Italic text* or _Italic text_
***Bold and italic***
~~Strikethrough~~
```

#### Lists
```markdown
<!-- Bulleted list -->
- Item 1
- Item 2
  - Indented sub-item

<!-- Numbered list -->
1. First step
2. Second step
3. Third step

<!-- Interactive Task / Checklist -->
- [x] Completed task
- [ ] Incomplete task
```

#### Code Formatting
````markdown
<!-- Inline code snippet -->
Use the `git status` command to check modified files.

<!-- Multi-line Code Block with Syntax Highlighting -->
```python
def greet(name: str) -> str:
    return f"Hello, {name}!"
```

```bash
git checkout -b feature/login
git add .
git commit -m "Add login screen"
```
````

#### Links and Images
```markdown
<!-- Hyperlink -->
[Visit GitHub](https://github.com)

<!-- Relative link to a local file -->
[Read Basics Guide](./git-github-basics.md)

<!-- Image -->
![Alt text for accessibility](https://github.githubassets.com/images/modules/logos_page/GitHub-Mark.png)
```

#### Tables
```markdown
| Command | Action | Frequency |
| :--- | :--- | :--- |
| `git status` | Inspect state | Frequently |
| `git commit` | Save snapshot | Daily |
| `git clone`  | Download repo | Once per project |
```

#### Blockquotes & Callouts
```markdown
> **Pro Tip**: Keep your commits focused on a single logical change.
```

---

### 1.3 How to View & Preview Markdown Files

Since `.md` files are plain text, you need a renderer to see headings, tables, and formatting rendered as rich HTML.

#### In Visual Studio Code (VS Code)

VS Code has built-in, first-class Markdown preview support without needing any external tools:

1. **Side-by-Side Preview (Recommended)**:
   - Open any `.md` file in VS Code.
   - Press `Ctrl + K` then `V` (Windows/Linux) or `Cmd + K` then `V` (Mac).
   - *Or* click the **"Open Preview to the Side"** icon (a split page with a magnifying glass) in the top-right corner of the editor tab.
   - Any changes you type on the left update live in the preview on the right!

2. **Full Tab Preview**:
   - Press `Ctrl + Shift + V` (Windows/Linux) or `Cmd + Shift + V` (Mac) to toggle the current tab into rendered preview mode.

3. **Recommended VS Code Extensions for Markdown**:
   - **Markdown All in One**: Keyboard shortcuts, table formatting, and auto-generated tables of contents.
   - **Markdownlint**: Flags formatting mistakes and enforces consistent styling.
   - **Markdown Preview Enhanced**: Adds math (LaTeX/KaTeX), Mermaid diagrams, and PDF/HTML export.

---

#### In Web Browsers

There are three easy ways to view Markdown files in a browser:

1. **On GitHub (Native & Automatic)**:
   - Any `.md` file pushed to GitHub (especially `README.md`) is rendered automatically by GitHub's web interface with full styling.
2. **Browser Extension (For local files)**:
   - Install an extension like **Markdown Viewer** (available for Chrome, Edge, and Firefox).
   - In browser settings for the extension, enable **"Allow access to file URLs"**.
   - Drag and drop any `.md` file directly into a browser tab to view it rendered.
3. **Online Markdown Editors**:
   - Paste Markdown text into tools like [StackEdit](https://stackedit.io/app) or [Dillinger](https://dillinger.io/) for instant dual-pane rendering.

---

## Part 2: Hands-on Git & GitHub Practice Exercises

Ready to build muscle memory? Complete these 5 real-world exercises sequentially.

> [!TIP]
> Do these exercises in a temporary test directory on your machine (e.g. `C:\dev\git-practice` or a scratch folder) so you can experiment freely without worrying about breaking real projects.

---

### Exercise 1: The Local Sandbox & First Commit

**Goal**: Initialize a new repository from scratch, create a Markdown file, stage it, and commit it.

#### Instructions:
1. Open your terminal or PowerShell and create an isolated practice folder:
   ```bash
   mkdir git-practice-lab
   cd git-practice-lab
   ```
2. Initialize Git in this directory:
   ```bash
   git init
   ```
3. Create a new file named `profile.md` with the following content:
   ```markdown
   # Developer Profile
   - Name: Your Name
   - Goal: Master Git & GitHub
   ```
4. Run `git status`. Observe that `profile.md` is listed in red under **"Untracked files"**.
5. Stage the file:
   ```bash
   git add profile.md
   ```
6. Run `git status` again. Notice it is now green under **"Changes to be committed"**.
7. Commit your work:
   ```bash
   git commit -m "Add initial developer profile in profile.md"
   ```
8. Verify your history:
   ```bash
   git log --oneline
   ```

**Success Checkpoint**:
Your terminal should show a single commit with your message and a clean working tree (`nothing to commit, working tree clean`).

---

### Exercise 2: Branch Isolation & Context Switching

**Goal**: Experience how branches isolate work without affecting the `main` branch.

#### Instructions:
1. Ensure your default branch is named `main`:
   ```bash
   git branch -M main
   ```
2. Create and switch to a new feature branch called `feature/skills`:
   ```bash
   git switch -c feature/skills
   ```
3. Open `profile.md` and add a skills section to the bottom:
   ```markdown
   ## Technical Skills
   - Python
   - Git & GitHub
   - Markdown
   ```
4. Stage and commit this change on your feature branch:
   ```bash
   git add profile.md
   git commit -m "Add technical skills section"
   ```
5. **The Magic Step (Context Switching)**:
   Switch back to `main`:
   ```bash
   git switch main
   ```
   Open `profile.md` in VS Code. Notice how the "Technical Skills" section vanished! It is safely preserved on `feature/skills`, leaving `main` completely untouched.
6. Switch back to your feature branch:
   ```bash
   git switch feature/skills
   ```
   The skills section returns immediately.

---

### Exercise 3: Simulating and Resolving a Merge Conflict

**Goal**: Intentionally cause a merge conflict and learn how to resolve it with confidence.

#### Instructions:
1. While on `feature/skills`, edit the **Name** line in `profile.md`:
   ```markdown
   - Name: Developer (Feature Branch Version)
   ```
   Commit this change:
   ```bash
   git add profile.md
   git commit -m "Update name on feature branch"
   ```
2. Switch back to `main`:
   ```bash
   git switch main
   ```
3. On `main`, edit the **exact same line** in `profile.md` to something different:
   ```markdown
   - Name: Developer (Main Branch Version)
   ```
   Commit this change to `main`:
   ```bash
   git add profile.md
   git commit -m "Update name on main branch"
   ```
4. Now, attempt to merge `feature/skills` into `main`:
   ```bash
   git merge feature/skills
   ```
5. **Git will report a conflict!**:
   ```
   CONFLICT (content): Merge conflict in profile.md
   Automatic merge failed; fix conflicts and then commit the result.
   ```
6. Open `profile.md` in VS Code. You will see conflict markers:
   ```markdown
   <<<<<<< HEAD
   - Name: Developer (Main Branch Version)
   =======
   - Name: Developer (Feature Branch Version)
   >>>>>>> feature/skills
   ```
7. **Resolve the conflict**:
   - In VS Code, click **"Accept Both Changes"**, **"Accept Current Change"**, or manually edit the text to whatever final version you want (e.g., `- Name: Jane Doe`).
   - Make sure to remove all `<<<<<<<`, `=======`, and `>>>>>>>` markers.
8. Stage the resolved file and conclude the merge:
   ```bash
   git add profile.md
   git commit -m "Resolve merge conflict between main and feature/skills"
   ```
9. Check your commit graph:
   ```bash
   git log --oneline --graph
   ```

**Success Checkpoint**:
You will see the two branches converge into a single merge commit.

---

### Exercise 4: Working with a Remote Repository on GitHub

**Goal**: Link a local repo to GitHub, push a branch, open a Pull Request, and merge it.

#### Instructions:
1. Log into your GitHub account at [github.com](https://github.com).
2. Click the **"+"** icon in the top-right corner $\rightarrow$ **"New repository"**.
3. Repository name: `git-practice-lab`.
4. Choose **Public** or **Private**, and **leave all initialization options unchecked** (no README, no .gitignore, no license, since you already have local files).
5. Click **"Create repository"**.
6. GitHub will display commands under *"push an existing repository from the command line"*. Copy and run those in your terminal:
   ```bash
   git remote add origin https://github.com/<your-username>/git-practice-lab.git
   git push -u origin main
   ```
7. Refresh your GitHub repository page in the browser—your `profile.md` and commit history will now appear online!

---

### Exercise 5: The Full Pull Request (PR) Workflow

**Goal**: Follow the professional industry workflow used by engineering teams.

#### Instructions:
1. In your local repository, create a new branch:
   ```bash
   git switch -c feature/add-bio
   ```
2. Add a short bio to `profile.md`:
   ```markdown
   ## About Me
   Aspiring software engineer learning Git branching and team collaboration!
   ```
3. Commit and push the branch to GitHub:
   ```bash
   git add profile.md
   git commit -m "Add About Me bio section"
   git push -u origin feature/add-bio
   ```
4. **Open the Pull Request on GitHub**:
   - Go to your repository on GitHub.
   - Click the yellow/green button **"Compare & pull request"**.
   - Title: `Add About Me bio section`.
   - Description: Fill in what changed and why.
   - Click **"Create pull request"**.
5. **Review the PR Diff**:
   - Click the **"Files changed"** tab in the PR.
   - Green lines represent additions, red lines represent deletions.
6. **Merge the PR**:
   - Click the green **"Merge pull request"** button (or choose **"Squash and merge"**).
   - Click **"Confirm merge"**.
   - Click **"Delete branch"** on GitHub to keep remote branches tidy.
7. **Sync Local Machine & Clean Up**:
   - Switch back to your local `main`:
     ```bash
     git switch main
     git pull
     ```
   - Delete your local feature branch now that it has merged:
     ```bash
     git branch -d feature/add-bio
     ```

---

## Exercise Summary & Self-Check Checklist

| Exercise | Skill Tested | Completed? |
| :--- | :--- | :---: |
| **Ex 1** | Initializing, creating Markdown, staging, committing | [ ] |
| **Ex 2** | Branch creation, switching, and isolating edits | [ ] |
| **Ex 3** | Understanding and resolving merge conflict markers | [ ] |
| **Ex 4** | Linking local repo to GitHub and pushing upstream | [ ] |
| **Ex 5** | PR creation, reviewing diffs, merging, and syncing back | [ ] |

