# Gitim - Git improved

## Prerequisites

- git - https://git-scm.com/book/en/v2/Getting-Started-Installing-Git
- fzf - https://github.com/junegunn/fzf
- bat - https://github.com/sharkdp/bat
- gh  - https://cli.github.com/
- tr  - (should be installed by default as part of coreutils)

## Setup

Download `git.sh` and place it in your `~/.bashrc.d` directory.

Add the following lines to your `~/.bashrc` file:
```bash
if [ -d ~/.bashrc.d ]; then
 for rc in ~/.bashrc.d/*; do
  if [ -f "$rc" ]; then
   . "$rc"
  fi
 done
fi
```
Add the following line to your `~/.bashrc` file to get Ctrl+g to work:
```bash
bind '"\C-g":"git_improved\n"'
```

> Change the GITHUB_URL in the top of this script to your GitHub URL before use!

## FZF Config

Add the following lines in your `~/.bashrc` file for a better experience with fzf:
```bash
export FZF_DEFAULT_COMMAND='rg --files --no-ignore --hidden --follow --glob "!.git/*"'
export FZF_DEFAULT_OPTS="--height 40% --margin 0.5% -1 --layout=reverse --multi --cycle --prompt='Search > ' --info=inline --border=top --color=border:bright-yellow --border-label='╢ FZF ╟' --border-label-pos=-3 --color=label:italic:bright-yellow --preview-label='╢ Preview ╟' --preview-label-pos=3 --preview='[[ \$(file --mime {}) =~ binary ]] && echo {} is a binary file || (bat --style=numbers --color=always {} || cat {}) 2> /dev/null | head -300'"
```

## Usage

Press `Ctrl+g` to run the script.

or

```
$ gitim
```

## The script

> Read the script before using it.

```bash
# README: Please read these comments before using this script

# NOTE: Gitim: Git improved
# Author: Christer Kilavik
# https://github.com/kilavila/gitim
# -----------------------------
# This script requires the following commands to be installed:
# git - https://git-scm.com/book/en/v2/Getting-Started-Installing-Git
# fzf - https://github.com/junegunn/fzf
# bat - https://github.com/sharkdp/bat
# gh  - https://cli.github.com/
# tr  - (should be installed by default as part of coreutils)
# -----------------------------
# -----------------------------

# USAGE: Place this script in your ~/.bashrc.d directory
# and add the following lines to your ~/.bashrc file:
  # if [ -d ~/.bashrc.d ]; then
  # 	for rc in ~/.bashrc.d/*; do
  # 		if [ -f "$rc" ]; then
  # 			. "$rc"
  # 		fi
  # 	done
  # fi
# -----------------------------
# Enter this command in your terminal: gitim
# -----------------------------
# Add the folling line to your ~/.bashrc file to get Ctrl+g to work:
# bind '"\C-g":"git_improved\n"'
# ------------------------------
# ------------------------------

# WARNING: READ THIS BEFORE USING THIS SCRIPT!
# ------------------------------
# Read through this script before using it.
# Use it and change it at your own risk.
# ------------------------------
# This script is provided AS-IS without warranty of any kind.
# It is not guaranteed that it will work on all systems.
# If you find any bugs, please report them to the author.
# ------------------------------
# -----------------------------

# TODO: Replace my Github URL with your own!
GITHUB_URL="https://github.com/kilavila"
COMMANDS="git add,git blame,git branch,git checkout,git commit,git fetch,git init,git log,git merge,git pull,git push,git rebase,git remote,git reset"

# INFO: GIT ADD
# Select 'Yes' to add all files
# Select 'No' to manually select files
# Use 'Tab'/'Shift+Tab' to select/deselect multiple files
# ------------------------------
git_add(){
  ADD_ALL=$(echo "Add all,Select files" | tr ',' '\n' | fzf --prompt='Confirm > ' --preview-window=hidden --border-label='╢ Git add ╟')

  if [ -z "$ADD_ALL" ]; then return
  elif [[ "$ADD_ALL" == "Add all" ]]; then git add :/
  else
    SELECTION=$(fzf --border-label='╢ Git add ╟')
    if [ -z "$SELECTION" ]; then return
    else git add "$SELECTION"
    fi
  fi
}

# INFO: GIT BLAME
# Select a file to view with git blame
# Select a line in the git blame view to checkout the commit
# ------------------------------
git_blame(){
  FILE=$(fzf --border-label='╢ Git blame ╟')

  if [ -z "$FILE" ]; then
    return
  fi

  SELECTION=$(git blame "$FILE" | fzf --height 100% --preview-window=hidden --border-label='╢ Git blame ╟')

  if [ -z "$SELECTION" ]; then
    return
  fi

  ID=$(echo "$SELECTION" | awk '{print $1}')
  git checkout "$ID"
}

# INFO: GIT BRANCH
# Select a branch to checkout
# ------------------------------
git_branch(){
  SELECTION=$(git branch -a | fzf --preview-window=hidden --border-label='╢ Git branch ╟')

  if [ -z "$SELECTION" ]; then
    return
  fi

  BRANCH=$(echo "$SELECTION" | sed -E 's/([^/]+\/[^/]+\/)?(\s+)?(\*\s+)?(.*)/\4/g')
  git checkout "$BRANCH"
}

# INFO: GIT CHECKOUT
# Enter a branch name or commit hash to checkout
# ------------------------------
git_checkout(){
  read -rp "Enter branch name or commit hash: " ID
  if [ -z "$ID" ]; then return
  else git checkout "$ID"
  fi
}

# INFO: GIT COMMIT
# Enter a commit message
# Uncomment the last line and comment out the rest to use f.ex vim
# ------------------------------
git_commit(){
  read -rp "Enter commit message: " MESSAGE
  if [ -z "$MESSAGE" ]; then return
  else git commit -m "$MESSAGE"
  fi

  # git commit
}

# INFO: GIT FETCH
# Fetch changes from a remote repository
git_fetch(){
	git fetch
}

# INFO: GIT INIT
# Initialize a repository on GitHub.com as well as a local repository
# Change the GITHUB_URL in the top of this script to your GitHub URL before use
git_init(){
  read -rp "Enter the name of the repository you want to initialize: " REPO
  if [ -z "$REPO" ]; then return; fi

  VISIBILITY=$(echo "public,private" | tr ',' '\n' | fzf --prompt='Select > ' --preview-window=hidden --border-label='╢ Git init ╟')
  if [ -z "$VISIBILITY" ]; then return; fi

  git init
  gh repo create "$REPO" --"$VISIBILITY"
  
  README=$(echo "Create README,Do NOT create README" | tr ',' '\n' | fzf --prompt='Select > ' --preview-window=hidden --border-label='╢ Git init ╟')
  if [ -z "$README" ]; then return
  elif [[ "$README" == "Create README" ]]; then
    touch README.md && echo "# $REPO" > README.md
  fi

  git add :/
  git commit -m "Initial commit"
  git remote add origin "$GITHUB_URL/$REPO.git"
  git push -u origin main
}

# INFO: GIT LOG
# View the commit history
# Select a commit to checkout
# ------------------------------
git_log(){
  SELECTION=$(git log --graph --oneline --decorate | fzf --preview-window=hidden --border-label='╢ Git log ╟')

  if [ -z "$SELECTION" ]; then
    return
  fi

  ID=$(echo "$SELECTION" | sed -E 's/(.*?)([|*])(\s+)([a-z0-9]{7})(.*)/\4/g')
  git checkout "$ID"
}

# INFO: GIT MERGE
# Select a branch to merge into the currently checked out branch
# Select a merge strategy to use for the merge
git_merge(){
  SELECTION=$(git branch -a | fzf --preview-window=hidden --border-label='╢ Git branch ╟')

  if [ -z "$SELECTION" ]; then
    return
  fi

  BRANCH=$(echo "$SELECTION" | sed -E 's/([^/]+\/[^/]+\/)?(\s+)?(\*\s+)?(.*)/\4/g')

  STRATEGY=$(echo "Accept both changes,Accept all incoming changes,Accept all current changes" | tr ',' '\n' | fzf --prompt='Select > ' --preview-window=hidden --border-label='╢ Git merge ╟')

  if [ -z "$STRATEGY" ]; then return
  elif [[ "$STRATEGY" == "Accept both changes" ]]; then git merge "$BRANCH"
  elif [[ "$STRATEGY" == "Accept all incoming changes" ]]; then git merge --strategy-option theirs "$BRANCH"
  elif [[ "$STRATEGY" == "Accept all current changes" ]]; then git merge --strategy-option ours "$BRANCH"
  fi

  git merge "$BRANCH"
}

# INFO: GIT PULL
# Pull changes from a remote repository
git_pull(){
  git pull
}

# TODO:
git_push(){
  git push || read -rp "Enter branch name to set upstream: " REPO && git push --set-upstream origin "$REPO"
}

# TODO:
# git_rebase(){}

# TODO:
# git_remote(){}

# INFO: GIT RESET
# Reset a branch to remote origin
# ------------------------------
git_reset(){
  read -rp "Enter the name of the branch you want to reset: " BRANCH

  if [ -z "$BRANCH" ]; then return
  fi

  CONFIRM=$(echo "Yes,No" | tr ',' '\n' | fzf --prompt='Confirm > ' --preview-window=hidden --border-label='╢ Git reset ╟')

  if [ -z "$CONFIRM" ]; then return
  elif [[ "$CONFIRM" == "Yes" ]]; then git reset --hard origin/"$BRANCH" && git clean -xdf
  else return
  fi
}

# INFO: GIT IMPROVED
# Select a command to run instead of the default git commands
# ------------------------------
git_improved(){
  COMMAND=$(echo "$COMMANDS" | tr ',' '\n' | fzf --height 40% --preview-window=hidden --border-label='╢ Git improved ╟')

  if [ -z "$COMMAND" ]; then return
  elif [[ "$COMMAND" == "git add" ]]; then git_add
  elif [[ "$COMMAND" == "git blame" ]]; then git_blame
  elif [[ "$COMMAND" == "git branch" ]]; then git_branch
  elif [[ "$COMMAND" == "git checkout" ]]; then git_checkout
  elif [[ "$COMMAND" == "git commit" ]]; then git_commit
  elif [[ "$COMMAND" == "git init" ]]; then git_init
  elif [[ "$COMMAND" == "git log" ]]; then git_log
  elif [[ "$COMMAND" == "git merge" ]]; then git_merge
  elif [[ "$COMMAND" == "git pull" ]]; then git_pull
  elif [[ "$COMMAND" == "git push" ]]; then git_push
  elif [[ "$COMMAND" == "git reset" ]]; then git_reset
  else echo "$COMMAND"
  fi
}

# INFO: ALIAS
# Change this alias to the command you want to run, f.ex:
# alias g="git_improved"
# Make sure your alias doesn't conflict with other aliases or commands
# ------------------------------
alias gitim="git_improved"
```
