> **Mode:** Book
> **Pilot-Seat Standard**

# Introduction
**Git and GitHub** represent the industry standard for distributed version control and collaborative software development. Git is a command-line tool that runs locally on a developer's computer to track file changes and manage code history. GitHub is a cloud-based hosting platform that stores Git repositories, allowing global development teams to share code, perform peer reviews, and automate deployments.

# Why It Exists
Before version control systems were invented, developers working on a shared project had to manually copy code files to share them. If two developers edited the same file simultaneously, whoever saved last would silently overwrite the other developer's changes, leading to lost work. If a new feature caused system errors, finding when and where the bug was introduced required manually comparing files line-by-line. Engineers created Git to establish a distributed system that records a complete history of every line of code, manages multiple development paths simultaneously, and automates file merging.

# Problem It Solves
Git and GitHub solve lost work overwrites, messy filename duplicates (e.g. `code-v2-final.zip`), and collaborative merge blockages.

### Before Git (Manual File Sharing):
- Developers emailed zip folders of code, resulting in confusion over which file was the latest version.
- Simulating experiments or new features meant duplicating the entire project folder on disk.
- Undoing a bug required hitting "Undo" in an editor repeatedly or manually rewriting code from memory.

### After Git (Version Controlled Repository):
- Code history is tracked down to the exact second, showing who edited what line and why.
- Developers create lightweight branches to experiment safely without affecting the working production code.
- Merges are handled automatically by the Git engine; if conflicts occur, Git highlights them clearly for resolution.
- Pull requests on GitHub establish structured gates where code is reviewed and tested before merging.

# Core Concepts
To collaborate using Git, you must master commits, branches, and remote repositories:

1. **Repository (Repo):** The project folder tracked by Git. It contains a hidden `.git` directory holding the complete history and metadata of every file change ever made.
2. **Commit (Snapshot):** A saved snapshot of the exact state of files at a specific moment. Each commit has a unique identifier hash and a description message.
3. **Branching:** The practice of creating an isolated, alternative timeline of the codebase. The default branch holding production-ready code is typically named `main` or `master`.
4. **Merge & Pull Request (PR):**
   - **Merge:** Combining history and changes from one branch into another.
   - **Pull Request:** A GitHub interface page where a developer proposes merging their branch into the main branch, allowing team discussion and code review.

# Architecture / Components
The flow of file changes through the three local Git zones to a remote GitHub repository:

```text
  [ Local Machine ]
  ┌────────────────────────────────────────────────────────┐
  │ [ Working Directory ]  ──> edit files                  │
  │          │                                             │
  │          ▼ (git add)                                   │
  │ [ Staging Area ]       ──> files prepared to be saved  │
  │          │                                             │
  │          ▼ (git commit)                                │
  │ [ Local Repository ]   ──> snapshots saved in history  │
  └──────────┬─────────────────────────────────────────────┘
             │
             ▼ (git push)
  [ Remote GitHub Cloud ]  ──> shared team code vault
```

- **Working Directory:** The local folder where you actively edit code.
- **Staging Area (Index):** The draft board where you select which modified files you want to include in your next snapshot.
- **Local Repository:** The local database holding all completed commits.
- **Remote Repository:** The cloud hosted copy of the repository on GitHub.

# Workflow
The standard lifecycle of making a code change and sharing it on GitHub:

```text
Step 1: Create a new branch for your feature: `git checkout -b add-login-page`.
                             ↓
Step 2: Edit your code files locally in your Working Directory.
                             ↓
Step 3: Add modified files to the Staging Area: `git add login.js`.
                             ↓
Step 4: Save the snapshot to your Local Repository: `git commit -m "Add secure login form"`.
                             ↓
Step 5: Upload your local branch to the remote cloud: `git push origin add-login-page`.
                             ↓
Step 6: Open a Pull Request (PR) on GitHub. Once approved, merge it into the `main` branch.
```

# Real World Examples
Think of Git and GitHub as **writing a collaborative book with a time-machine catalog**.
- **Without Git:** 5 writers are editing a book. You copy the file `chapter1.txt` and edit page 10. Your friend copies the same file and edits page 20. When you try to merge your drafts, you have to read both copies line-by-line to copy paste edits into a new file: `chapter1-final-edit.txt`. This is manual merging and causes chaos.
- **With Git:**
  - **Repository:** The library vault where the book draft is stored.
  - **Commit:** Taking a Polaroid photo of the manuscript page. Every time you finish a page, you take a photo (**Commit**), write a description on the back (*"Finished paragraph 3"*), and file it. If you make a mistake on page 20, you don't erase it; you flip back to the photo of page 19 to see exactly how it looked.
  - **Branching:** Drafting alternative chapters. You want to see if the main character dying makes the book better. You start a new branch (**Alternative timeline**). You write the death scene. The other writers continue working on the main draft where the character is alive.
  - **Merge Conflict:** You decide to merge the death scene back into the main book. However, another writer edited a conversation the character had on the same page. The publisher (**Git**) holds up both pages and says: *"You both edited the same line. I cannot print both. You must cross out one version before we print."*
  - **GitHub:** The publisher's central cloud office. You write your chapters locally on your laptop (**Local repo**), and mail them to the central office (**GitHub**) so the editors can read, review (**Pull Request**), and approve them.

# Implementation
Here are the daily Git commands developers use to manage code files:

### 1. Initializing and Saving Changes
```bash
# Initialize a new Git repository in your current folder
git init

# View the status of your files (untracked, modified, or staged)
git status

# Add a specific file to the Staging Area
git add server.js

# Add all modified and new files to the Staging Area
git add .

# Save the staged files as a commit with a descriptive message
git commit -m "feat: implement user registration endpoint"
```

### 2. Branching and Merging
```bash
# List all local branches
git branch

# Create and switch to a new branch named 'feature-oauth'
git checkout -b feature-oauth

# Switch back to the main branch
git checkout main

# Merge the 'feature-oauth' changes into the active branch (main)
git merge feature-oauth

# Delete the feature branch locally once merged
git branch -d feature-oauth
```

### 3. Collaborating with Remote GitHub repositories
```bash
# Link your local repository to a remote GitHub repository
git remote add origin https://github.com/user/my-repo.git

# Upload your local commits on 'main' to GitHub
git push -u origin main

# Download the latest changes from GitHub and merge them into your local branch
git pull origin main
```

# Best Practices
- **Write Clear, Imperative Commit Messages:** Write commit messages that describe what the commit does in the present tense (e.g. `feat: add password reset` or `fix: resolve memory leak`), rather than vague descriptions like `stuff` or `fixes`.
- **Commit Early and Often:** Keep commits small and focused. A single commit should address a single logical change. This makes it easy to locate and roll back specific updates if a bug is discovered.
- **Never Push Secrets or Configs:** Never commit private API keys, database passwords, or `.env` files to your Git repository. Create a `.gitignore` file to instruct Git to ignore these files completely.

# Industry Standards
Professional teams use defined **branching strategies** (like Gitflow or Trunk-Based Development). In Trunk-Based Development, developers merge small, frequent commits directly into the `main` branch daily, using feature flags to hide incomplete features. This avoids massive, complex merge conflicts at the end of a sprint.

# Common Mistakes
- **Committing Secrets to GitHub:** Accidental pushing of a `.env` file containing API keys to a public GitHub repository. Attackers run scripts scanning GitHub constantly for secrets and will exploit leaked keys within seconds. Always include a `.gitignore` file before running `git add`.
- **Detached HEAD State:** Switching to a past commit ID and making edits. Edits made in this state are not saved to any branch and will disappear if you switch branches. Always create a new branch if you want to experiment on past commits.
- **Force Pushing to Main (`git push --force`):** Overwriting the remote GitHub history with your local history. If your local history is outdated, this will wipe out your colleagues' commits on GitHub, resulting in catastrophic loss of team code. Never force-push to shared branches.

# Security & Performance Considerations
- **Git Repo Bloat:** Git stores a complete copy of every version of every file in the history. If you commit large binary files (like images, zip archives, or video clips), your `.git` folder will balloon to gigabytes, making `git clone` operations incredibly slow. Use **Git LFS (Large File Storage)** for large media files.
- **Pull Request Gatekeeping:** Configure branch protection rules on GitHub to block developers from writing or pushing code directly to the `main` branch. Require all merges to pass automated tests (CI pipelines) and receive at least one peer approval.

# Related Technologies
- **Git LFS (Large File Storage):** An extension that replaces large files with text pointers inside Git, storing the actual files on cloud storage.
- **GitHub Actions:** GitHub's built-in CI/CD automation pipeline system.

# Summary

## What We Learned
- Git is a local version control utility; GitHub is a cloud hosting platform for collaboration.
- Files transition through the Working Directory, Staging Area, and Local Repository before being pushed to GitHub.
- Branching allows developers to experiment in safe alternative timelines before merging.

## Key Takeaways
- Use a `.gitignore` file immediately when starting any project to block password leaks.
- Write descriptive, short commit messages that explain what the change accomplishes.
- Protect your `main` branch on GitHub by requiring code reviews and automated tests for all Pull Requests.

# Keywords
- Git
- GitHub
- Repository
- Commit
- Branch
- Merge
- Pull Request (PR)
- .gitignore
- Staging Area
- Merge Conflict

# Glossary

| Term | Meaning |
|---|---|
| Repository | The project directory tracked by Git, containing code files and complete change history. |
| Commit | A saved historical snapshot of the tracked files in a Git repository. |
| Branch | An independent timeline of code changes branching off the main codebase. |
| Merge Conflict | A state where Git blocks a merge because two commits edited the same line of a file, requiring manual choice. |
| Pull Request | A GitHub proposal requesting to merge changes from one branch into another, used for code review. |
| .gitignore | A text file listing folder and file patterns that Git should ignore and never track. |

## Next Recommended Chapters
- 03-CI-CD-Fundamentals.md
