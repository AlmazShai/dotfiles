# Dotfiles

## Setup
```
git clone --recurse-submodules --bare https://github.com/AlmazShai/dotfiles.git $HOME/.dotfiles
alias config='/usr/bin/git --git-dir=$HOME/.dotfiles/ --work-tree=$HOME'
config config --local status.showUntrackedFiles no
config checkout
```
