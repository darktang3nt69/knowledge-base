1. Installation of zsh and ohmyzsh and ohmyposh:
```bash
sudo apt install zsh
sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
sudo curl -s https://ohmyposh.dev/install.sh | sudo bash -s
```

2. Select themes from here: https://ohmyposh.dev/docs/themes and add oh-my-posh prompt to .zshrc: https://ohmyposh.dev/docs/installation/prompt:
```bash
source $ZSH/oh-my-zsh.sh

eval "$(oh-my-posh init zsh --config ~/catppuccin_frappe.omp.json)" # ~/catppuccin_frappe.omp.json is the theme.
```

3. Install powerlevel10k:
```bash
git clone --depth=1 https://github.com/romkatv/powerlevel10k.git ~/powerlevel10k
echo 'source ~/powerlevel10k/powerlevel10k.zsh-theme' >>~/.zshrc
exec zsh
p10k configure
```

4. Make `zsh` default shell:
```
sudo chsh -s $(which zsh)
```

5. Auto completion and suggestions:
```bash
sudo git clone https://github.com/zsh-users/zsh-autosuggestions ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-autosuggestions
sudo git clone https://github.com/zsh-users/zsh-syntax-highlighting.git ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting
```

6. Add the following to plugins section of .zshrc:
```
plugins=(
  git
  docker
  zsh-autosuggestions
  zsh-syntax-highlighting
  )
```

7. Install `colorls`:
```bash
sudo apt-get update
sudo apt-get install ruby3.0-dev
gem install colorls
```

8. Add aliases:
```
alias l="colorls -A --sd"
alias ll="colorls -lA --sd"
```

9. Error related to history file:
```bash
%% Check if the permissions or the ownership have to be changed. %%
ls -ld $(dirname $HISTFILE)
sudo chown $(whoami) $(dirname $HISTFILE) 

# Add the following to .zshrc file
if [ -f "$HOME/.zsh_history1" ]; then
    chmod 600 "$HOME/.zsh_history1"
fi

HISTFILE="$HOME/.zsh_history1"
HISTSIZE=10000
SAVEHIST=10000
setopt SHARE_HISTORY
```