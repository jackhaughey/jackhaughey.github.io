# Creating Github Issues

I've started documenting my Red Hat studies in a GitHub repository, but rather than treating it as a collection of notes, I'm trying to follow a more realistic engineering workflow.

Each RHCSA topic is tracked as a GitHub Issue, developed in its own feature branch, and merged back into the main branch through a pull request once completed. This allows me to link requirements, documentation, commits, and completed lab work together in a way that mirrors how many engineering and DevOps teams manage projects.

The goal isn't just to learn Linux administration, but also to practise version control, documentation, and project management alongside the technical material.

## New issue

### Creating a new issue;
```
gh issue create \
  --title "Implement RHCSA Networking Labs" \
  --body "Create networking labs and documentation"
```

### To work on the Issue branch;
```
gh issue develop 1 --checkout
```

### Referencing the issue in commits;
```
git commit -m "Add networking section README (#1)"
```

## Merge to Main 

### When finished;
```
git push -u origin feature/1-networking-labs
```

### Create a Pull request;
```
gh pr create \
  --title "Implement RHCSA Networking Labs" \
  --body "Closes #1"
```

### Determine the current branch;
```
git branch
```

### Switch to the Main branch;
```
git checkout main
```

### Pull updates from Main
```
git pull origin main
```

### Merge the branches
```
git merge feature/8-networking-labs
```
