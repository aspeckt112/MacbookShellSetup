# zsh fast-local setup (no oh-my-zsh)

macOS `zsh` + `oh-my-posh`, no framework.

## Brew deps

```sh
brew install fzf ripgrep fd zoxide eza bat
```

## One-time

```sh
mkdir -p ~/.config/zsh/plugins
git clone https://github.com/Aloxaf/fzf-tab ~/.config/zsh/plugins/fzf-tab
```

## `~/.zshrc` additions

```zsh
# --- completions (must be before fzf-tab) ---
# brew site-functions holds _brew; zsh bundles _git
if type brew &>/dev/null; then
  FPATH="$(brew --prefix)/share/zsh/site-functions:$FPATH"
fi
autoload -Uz compinit
compinit -d ~/.zcompdump

zstyle ':completion:*' matcher-list 'm:{a-z}={A-Za-z}'
zstyle ':completion:*' menu select
zstyle ':completion:*' use-cache on
zstyle ':completion:*' cache-path ~/.zcompcache

# --- history (fast local, shared) ---
HISTFILE=~/.zsh_history
HISTSIZE=50000
SAVEHIST=50000
setopt SHARE_HISTORY HIST_IGNORE_ALL_DUPS HIST_REDUCE_BLANKS EXTENDED_HISTORY INC_APPEND_HISTORY

# --- fzf + rg + fd ---
# rg feeds CTRL-T, fd feeds ALT-C
export FZF_DEFAULT_COMMAND='rg --files --hidden --glob "!.git"'
export FZF_CTRL_T_COMMAND="$FZF_DEFAULT_COMMAND"
export FZF_ALT_C_COMMAND='fd --type d --hidden --exclude .git'
eval "$(fzf --zsh)"
# CTRL-R history, CTRL-T files, Option+C dirs (Option as Meta)

# --- fzf-tab (after compinit) ---
source ~/.config/zsh/plugins/fzf-tab/fzf-tab.plugin.zsh
zstyle ':fzf-tab:*' fzf-flags --height=40% --layout=reverse --border
zstyle ':fzf-tab:*' group-switch enable

# --- zoxide (after compinit) ---
eval "$(zoxide init zsh)"
# z <query>, zi (fzf picker)

# --- eza / ll ---
alias ls='eza --icons --group-directories-first'
alias ll='eza -lh --icons --git --group-directories-first'
alias la='eza -lah --icons --git --group-directories-first'
alias lt='eza --tree --level=2 --icons'

# --- fzf previews ---
export FZF_CTRL_T_OPTS="--preview 'bat -n --color=always {} 2>/dev/null || eza --tree --level=1 --color=always {} 2>/dev/null || head -100 {}'"
export FZF_ALT_C_OPTS="--preview 'eza --tree --level=1 --color=always {}'"
zstyle ':fzf-tab:complete:cd:*' fzf-preview 'eza -1 --color=always $realpath'
```

## Ghostty: make Option+C work

`fzf` binds dir-jump to `^[c` (`\ec`), i.e. `ESC` then `c` = `Option+C`.
On Mac `ALT` = `Option`, and Ghostty must send it as Alt:

```sh
# ~/.config/ghostty/config
macos-option-as-alt = true
# use `left` / `right` instead of `true` to keep one Option key for glyphs (ç, ©)
```

Reload Ghostty. Test: `Option+C` should open the dir picker.
Other terminals: Terminal.app `Use Option as Meta key`, iTerm2 `Left Option → Esc+`.

## Usage

* `TAB`: `git checkout <TAB>`, `brew <TAB>`, `cd <TAB>`, `z <TAB>` — type to filter, `Enter` select, `ESC` cancel
* `CTRL-R`: fuzzy history (shared across sessions)
* `CTRL-T`: find files (`rg --files`), inserts path — e.g. `vim <CTRL-T>`
* `Option+C`: find dirs (`fd --type d`), jumps there — previews via `eza`
* `z proj`: jump to learned dir, `zi`: fzf picker, history auto-learns as you `cd`
* `ls` / `ll` / `la` / `lt`: `eza` (icons, git status, tree)
* `rg "pat" -g "*.ts"`: content search, respects `.gitignore`

## Verify

```sh
exec zsh
git <TAB>
brew <TAB>
z --help; zi; ll; lt
```
