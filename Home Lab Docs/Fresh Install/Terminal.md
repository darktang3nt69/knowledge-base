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