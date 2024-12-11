# common-commands.md

Hey future Git master! 🎮

Let's talk about Git - your coding time machine and collaboration superpower! Think of Git like a really smart save system in a video game. You know how games let you save at different points so you can try different strategies? Git does that for your code, but it's even better!

### Understanding Git (The Fun Way)

Imagine you're writing a story in a magical notebook. This notebook:

- Remembers every version of your story
- Lets you work on different versions at the same time
- Helps you work with friends without overwriting each other's changes
- Can merge different versions together

That's Git! Now let's learn how to use this magical notebook.

### Your First Git Commands

Let's start with the commands you'll use every single day. I'll explain what each one does and why you'd use it:

### Setting Up Your Identity

First, tell Git who you are (like creating your player profile):

```bash
git config --global user.name "Your Name"
git config --global user.email "your.email@example.com"
```

### Starting Your Git Journey

Creating a new project (or starting to track an existing one):

```bash
git init
```

This is like creating a new save file for your game. Git will now start tracking changes in this folder.

### The Three Stages of Git

Think of Git like preparing a package to send:

1. Working Directory (your desk where you're working)
2. Staging Area (the box where you put things you want to send)
3. Repository (the post office that keeps records of all packages)

Here's how to use them:

### Checking Status (Your Radar)

```bash
git status
```

This is like asking Git "What's changed since my last save?" It'll show you:

- Files you've modified
- New files Git doesn't know about
- Files ready to be committed

### Adding Files (Preparing Your Save)

```bash
git add filename.js        # Add a specific file
git add .                 # Add all files
```

Think of this like choosing which items to put in your package. You're telling Git "These changes are important, I want to save them."

### Committing Changes (Creating Your Save Point)

```bash
git commit -m "Add login feature"
```

This is like sealing the package with a label. The message should explain what changed and why. Some examples:

```bash
git commit -m "Fix navigation bar alignment"
git commit -m "Add user profile page"
git commit -m "Update database connection settings"
```

Pro tip: Write commit messages that would help "future you" understand what changed!

### Viewing History (Your Time Machine)

```bash
git log                   # See all commits
git log --oneline        # See condensed version
```

This shows you all your save points. Each commit has:

- A unique ID (like a serial number)
- The author (who made the change)
- The date (when it happened)
- The message (what changed)

### Branches (Parallel Universes)

Branches let you work on different versions of your code at the same time:

```bash
git branch               # See all branches
git branch feature-name  # Create new branch
git checkout branch-name # Switch to branch
```

Pro tip: Use `git checkout -b new-branch` to create and switch to a new branch in one command!

### Practical Example: A Typical Workflow

Let's say you're adding a new feature to your website. Here's how you'd do it:

```bash
# Create a new branch for your feature
git checkout -b add-login-page

# Make your changes...
# Now check what's changed
git status

# Add your changed files
git add src/components/Login.js
git add src/styles/login.css

# Create a save point
git commit -m "Add login page with email/password fields"

# Make more changes...
git add src/components/Login.js
git commit -m "Add form validation to login page"
```

### When Things Go Wrong (Don't Panic!)

We all make mistakes! Here's how to fix common ones:

#### Oops, I made a change I don't want to keep:

```bash
git checkout -- filename.js
```

#### Oops, I committed to the wrong branch:

```bash
git log -n 1            # Get the commit ID
git checkout right-branch
git cherry-pick commit-id
```

#### Oops, I need to change my last commit message:

```bash
git commit --amend -m "New message"
```

### Git Best Practices (Level Up Your Git Game!)

1. **Commit Often**: Make small, focused commits. It's like creating restore points in a game - the more you have, the less you lose if something goes wrong.

2. **Branch Names**: Use descriptive branch names:

   - feature/add-login
   - bugfix/fix-navbar
   - update/upgrade-react

3. **Commit Messages**: Write clear, descriptive messages:

   ```bash
   # Good:
   git commit -m "Add password reset functionality to login form"

   # Not so good:
   git commit -m "fixed stuff"
   ```

### Your Daily Git Routine

Here's what a typical day with Git looks like:

1. Start your day:

   ```bash
   git pull                  # Get latest changes
   git checkout -b my-task   # Start new task
   ```

2. While working:

   ```bash
   git status               # Check changes
   git add .               # Stage changes
   git commit -m "Message" # Save changes
   ```

3. End of day:
   ```bash
   git push                # Share your changes
   ```

### Practice Exercise

Let's try a simple exercise to practice these commands:

1. Create a new folder and initialize Git
2. Create a file called `app.js`
3. Add some code to the file
4. Stage and commit your changes
5. Create a new branch
6. Make more changes
7. Stage and commit on your new branch

Try it out! If you get stuck or want to learn about more advanced Git features, just let me know. We can explore topics like merging, rebasing, or handling merge conflicts next!

Remember: Git might seem complicated at first, but it's just like learning a new game - start with the basics, and you'll be a pro before you know it! Want to try any of these commands or learn about something specific?
