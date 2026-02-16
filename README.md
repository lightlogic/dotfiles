# Dotfiles (bare repository) setup

This repository is managed as a **bare Git repository** located at:

-   **Git directory (bare repo):** `~/.dotfiles/`
-   **Work tree (your real files):** `~/`

This means: - Git history lives in `~/.dotfiles/` - The tracked files
are the *actual* files in your home directory (`~/.vimrc`, `~/.zshrc`,
etc.) - You do **not** clone dotfiles into a separate folder and symlink
everything. - You must **not** run plain `git` inside `~/.dotfiles/`
(there is no work tree there).

If you try, Git will complain: \> fatal: this operation must be run in a
work tree

------------------------------------------------------------------------

## Why use a bare repo for dotfiles?

### Pros

-   No symlink clutter.
-   Files live where applications expect them.
-   Clean version control for multiple dotfiles.

### Cons

-   You must use a wrapper command so Git knows:
    -   where the repository is (`--git-dir=...`)
    -   where the work tree is (`--work-tree=...`)
-   You should hide untracked files from status output.

------------------------------------------------------------------------

## Core idea: use a wrapper command

### One-off command

Use this form if no alias is defined:

``` bash
/usr/bin/git --git-dir="$HOME/.dotfiles/" --work-tree="$HOME" <command>
```

### Recommended alias

Add this to your shell config (`~/.zshrc` or `~/.bashrc`):

``` bash
alias dotfiles='/usr/bin/git --git-dir=$HOME/.dotfiles/ --work-tree=$HOME'
```

Reload your shell:

``` bash
source ~/.zshrc
```

Now use:

``` bash
dotfiles status
dotfiles add ~/.vimrc
dotfiles commit -m "Track vim config"
```

------------------------------------------------------------------------

## Initial setup

``` bash
git init --bare "$HOME/.dotfiles"
alias dotfiles='/usr/bin/git --git-dir=$HOME/.dotfiles/ --work-tree=$HOME'
dotfiles config --local status.showUntrackedFiles no
```

The `status.showUntrackedFiles no` setting prevents Git from listing
your entire home directory.

------------------------------------------------------------------------

## Daily workflow

### Check status

``` bash
dotfiles status
```

### Add files

``` bash
dotfiles add ~/.vimrc
dotfiles add ~/.vim/
```

### Commit

``` bash
dotfiles commit -m "Update vim config"
```

### Fetch remote changes

``` bash
dotfiles fetch origin
```

### Pull (recommended with rebase)

``` bash
dotfiles pull --rebase origin main
```

### Push

``` bash
dotfiles push origin main
```

### First push (set upstream)

``` bash
dotfiles push -u origin main
```

------------------------------------------------------------------------

## Remote setup

### Add remote

``` bash
dotfiles remote add origin git@github.com:<USER>/<REPO>.git
```

### Verify remote

``` bash
dotfiles remote -v
```

------------------------------------------------------------------------

## Install on a new computer

### 1. Clone the bare repository

``` bash
git clone --bare git@github.com:<USER>/<REPO>.git "$HOME/.dotfiles"
```

### 2. Define the alias

``` bash
alias dotfiles='/usr/bin/git --git-dir=$HOME/.dotfiles/ --work-tree=$HOME'
```

### 3. Hide untracked files

``` bash
dotfiles config --local status.showUntrackedFiles no
```

### 4. Checkout files

``` bash
dotfiles checkout
```

If you see conflicts such as: \> error: The following untracked working
tree files would be overwritten by checkout

You can back them up first:

``` bash
mkdir -p "$HOME/.dotfiles-backup"
dotfiles checkout 2>&1 | egrep "\s+\." | awk '{print $1}' | while read -r f; do
  mkdir -p "$HOME/.dotfiles-backup/$(dirname "$f")"
  mv "$HOME/$f" "$HOME/.dotfiles-backup/$f"
done

dotfiles checkout
```

Or force overwrite:

``` bash
dotfiles checkout -f
```

### 5. Persist alias

Add to `~/.zshrc` or `~/.bashrc`:

``` bash
alias dotfiles='/usr/bin/git --git-dir=$HOME/.dotfiles/ --work-tree=$HOME'
```

------------------------------------------------------------------------

## Safety checks

``` bash
dotfiles rev-parse --is-bare-repository
dotfiles rev-parse --is-inside-work-tree
```

------------------------------------------------------------------------

## Common issues

### fatal: this operation must be run in a work tree

Use the wrapper command (`dotfiles ...`) instead of plain `git`.

### Push rejected (fetch first)

Remote contains commits you do not have locally:

``` bash
dotfiles pull --rebase origin main
dotfiles push origin main
```

If you intentionally want to overwrite remote:

``` bash
dotfiles push --force-with-lease origin main
```

------------------------------------------------------------------------

## What to track

Track: - Configuration files (`~/.vimrc`, `~/.zshrc`, etc.) - Config
directories (`~/.config/nvim/`) - Useful scripts (`~/bin/`)

Avoid tracking: - Caches - Generated plugin directories - Large
binaries - Secrets (tokens, private keys)

------------------------------------------------------------------------

## Quick reference

``` bash
dotfiles status
dotfiles add ~/.vimrc
dotfiles commit -m "message"
dotfiles fetch origin
dotfiles pull --rebase origin main
dotfiles push origin main
dotfiles log --oneline --decorate --graph --max-count=20
```
