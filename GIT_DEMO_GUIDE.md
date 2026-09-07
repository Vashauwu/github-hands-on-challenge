# 🧪 Demo Guide: Git with a Jupyter Notebook

A step-by-step script for showing your students the most common Git commands,
using the `analysis_notebook.ipynb` + `sales_data.csv` project as a base.

Each section shows: the **command**, what's happening, and (when relevant)
what change to make in the notebook before running it, so real, visible
diffs and commits are generated.

---

## 0. Before you start

1. Create an empty repository on GitHub (no README, no license) — this way
   `git remote add` + `git push` will work without conflicts.
2. Have this folder (`demo-git-en/`) ready on your machine, with Jupyter
   installed if you plan to show the notebook actually running.

---

## 1. Initialize the local repository

```bash
cd demo-git-en
git init
git status
```

Explain: `git init` creates the hidden `.git` folder. `git status` shows
that all files are "untracked".

---

## 2. First commit

```bash
git add .
git status
git commit -m "Initial commit: notebook and starting data"
git log
```

- `git add .` moves the files to the *staging area*.
- `git status` (again) shows the difference between staged/unstaged.
- `git commit` saves a "snapshot" of the project.
- `git log` shows the history.

---

## 3. Connect to GitHub and push

```bash
git branch -M main
git remote add origin https://github.com/YOUR-USERNAME/YOUR-REPO.git
git push -u origin main
```

Show the repo updating live on the GitHub website.

---

## 4. Seeing a change and its diff

Edit the notebook: add a new cell at the end that computes **total
revenue** (`units_sold * unit_price`), for example:

```python
df['revenue'] = df['units_sold'] * df['unit_price']
df.groupby('product')['revenue'].sum().sort_values(ascending=False)
```

Then:

```bash
git status
git diff
```

`git diff` shows what changed line by line (notebook diffs are in JSON, so
it looks a bit "noisy" — a good moment to mention tools like `nbdime` for
cleaner notebook diffs).

```bash
git add analysis_notebook.ipynb
git commit -m "Add total revenue calculation by product"
```

---

## 5. Working with branches (branch + switch/checkout)

```bash
git branch revenue-chart
git switch revenue-chart
# (classic alternative: git checkout -b revenue-chart)
```

Now on that branch, add a cell with a revenue chart:

```python
revenue = df.groupby('product')['revenue'].sum().sort_values(ascending=False)
revenue.plot(kind='bar', title='Total revenue by product')
plt.ylabel('Revenue ($)')
plt.show()
```

```bash
git add analysis_notebook.ipynb
git commit -m "Add revenue by product chart"
git log --oneline --graph --all
```

This last command is great for **visualizing branches**.

---

## 6. Merge without conflicts

```bash
git switch main
git merge revenue-chart
git log --oneline --graph --all
```

Explain *fast-forward* vs. *merge commit* (depends on whether `main`
changed in the meantime or not).

---

## 7. Causing and resolving a conflict (very instructive)

1. On `main`, edit the notebook's title (first markdown cell) to:
   `# Sales Analysis 2025 - School Supplies Store`. Commit:
   ```bash
   git add analysis_notebook.ipynb
   git commit -m "Update notebook title"
   ```
2. Create another branch from the commit *before* that change (or simply
   create a new branch and edit that same line differently, e.g.
   `# 📈 Sales Report - School Supplies Store`), and commit there too.
3. Go back to `main` and try the merge:
   ```bash
   git switch main
   git merge other-branch
   ```
4. Git will flag a conflict in the notebook. Show the
   `<<<<<<<`, `=======`, `>>>>>>>` markers, edit them by hand, then:
   ```bash
   git add analysis_notebook.ipynb
   git commit -m "Resolve title conflict"
   ```

---

## 8. `.gitignore` in action

Generate Jupyter checkpoints by opening and saving the notebook (this
creates `.ipynb_checkpoints/`), or create a virtual environment
(`python -m venv venv`).

```bash
git status
```

Show that these folders **don't appear** because they're in `.gitignore`.
A good moment to explain why not everything should be versioned.

---

## 9. Saving work in progress with `stash`

Start editing the notebook (e.g. add an incomplete cell) but don't commit,
and pretend you need to switch tasks urgently:

```bash
git stash
git status          # the change "disappeared" temporarily
git stash list
git stash pop        # get it back
```

---

## 10. `revert` vs `reset`

Make an intentionally "bad" commit (e.g. delete an important cell):

```bash
git add analysis_notebook.ipynb
git commit -m "Experimental change (we'll regret this)"
```

**Option A – revert (safe, recommended in teams):**
```bash
git revert HEAD
```
Creates a new commit that undoes the previous one, without erasing history.

**Option B – reset (rewrites history, use with care):**
```bash
git reset --soft HEAD~1   # undoes the commit, keeps changes staged
git reset --hard HEAD~1   # undoes the commit and the change entirely (destructive!)
```

---

## 11. Pulling changes from the remote

If you made changes directly on GitHub (e.g. edited the README from the
web), show:

```bash
git pull origin main
```

And if you want to show how to start fresh from a remote repo:

```bash
git clone https://github.com/YOUR-USERNAME/YOUR-REPO.git student-copy
```

---

## 📋 Quick reference cheat sheet

| Command | What it does |
|---|---|
| `git init` | Creates a new repo |
| `git clone <url>` | Downloads an existing repo |
| `git status` | What changed / what's staged |
| `git add <file>` | Stages changes for commit |
| `git commit -m "msg"` | Saves a snapshot of the project |
| `git log --oneline --graph --all` | Visual history |
| `git diff` | View line-by-line changes |
| `git branch <name>` | Create a branch |
| `git switch <branch>` / `git checkout <branch>` | Switch branches |
| `git merge <branch>` | Merge branches |
| `git remote add origin <url>` | Connect to GitHub |
| `git push` / `git pull` | Push / pull changes |
| `git stash` / `git stash pop` | Save work in progress |
| `git revert <commit>` | Undo with a new commit (safe) |
| `git reset --hard <commit>` | Rewrite history (destructive) |

---

## 💡 Presentation tip

Keep two windows open: one with the terminal and one with GitHub in the
browser (the "Commits" tab and the repo's "Network graph"). Every `git
push` is reflected live — that's the biggest visual payoff for your
students.
