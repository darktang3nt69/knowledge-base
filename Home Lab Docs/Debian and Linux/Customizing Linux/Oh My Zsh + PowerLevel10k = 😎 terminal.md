## Pre-requisites

> if you're using [Windows](https://www.microsoft.com/en-us/windows), you can install and configure [WSL](https://docs.microsoft.com/en-us/windows/wsl/)

and I recommended to use [Ubuntu](https://ubuntu.com/) or [Debian](https://www.debian.org/) wsl plugin

Setup [zsh](https://www.zsh.org/)

in the command line type  

```bash
sudo apt-get install zsh
```

type **zsh**  

```bash
zsh
```

Install [Oh My Zsh](https://ohmyz.sh/)

```
sh -c "$(curl -fsSL https://raw.github.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"
```

[PowerLevel10k](https://github.com/romkatv/powerlevel10k)

- Install Powerlevel10k using the following command

```bash
git clone https://github.com/romkatv/powerlevel10k.git $ZSH_CUSTOM/themes/powerlevel10k
```

Then you need to enable it, change the value of ZSH_THEME to following in `~/.zshrc` file :  

```
ZSH_THEME="powerlevel10k/powerlevel10k"
```

Configure Powerlevel10k Theme

- Make sure your terminal font is `FiraCode NF`.

(https://dev.to/abdfnx/oh-my-zsh-powerlevel10k-cool-terminal-1no0#cheatsheet-for-windows)Cheat-sheet for Windows

if you've `Windows terminal` you can open your settings and in **UNIX** preferences and add `fontFace` prop,  
assign it to `FiraCode NF`.  

```
{
  "guid": "{YOUR_UNIX_GUID}",
  "hidden": false,
  "name": "Ubuntu",
  "source": "Windows.Terminal.Wsl",
  "fontFace": "FiraCode NF",
  "snapOnInput": true,
  "useAcrylic": true
}
```

> Windows Terminal url in Microsoft Store: [url](https://www.microsoft.com/en-us/p/windows-terminal/9n0dx20hk701)
> 
> Windows Terminal repo: [url](https://github.com/microsoft/terminal)

### [](https://dev.to/abdfnx/oh-my-zsh-powerlevel10k-cool-terminal-1no0#p10k-configure)**p10k configure**

type  

```bash
p10k configure
```

GIF

![x](https://res.cloudinary.com/practicaldev/image/fetch/s--71QSuVWr--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_66%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/i/xf9fk2sgux1niog4vhpy.gif)

you can choose your style...

> ## [](https://dev.to/abdfnx/oh-my-zsh-powerlevel10k-cool-terminal-1no0#plugins-optional-good-to-have)Plugins (Optional, Good to have!)

Clone plugins

- zsh-syntax-highlighting - It enables highlighting of commands whilst they are typed at a zsh prompt into an interactive terminal. This helps in reviewing commands before running them, particularly in catching syntax errors.

```bash
git clone https://github.com/zsh-users/zsh-syntax-highlighting.git ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting
```

- zsh-autosuggestions - It suggests commands as you type based on history and completions.

```bash
git clone https://github.com/zsh-users/zsh-autosuggestions ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-autosuggestions
```


- [exa](https://the.exa.website/): is a modern replacement for _ls_
```bash
sudo apt install exa
```


- [tran](https://github.com/abdfnx/tran): Securely transfer and send anything between computers with TUI.

```
curl -sL https://cutt.ly/tran-cli | bash
```

In `~/.zshrc` file replace the line starting with `plugins=()` to below line.  

```
plugins=( git zsh-syntax-highlighting zsh-autosuggestions )
```

> or exa  

```
if [ -x "$(command -v exa)" ]; then
    alias ls="exa"
    alias la="exa --long --all --group"
fi
```

> Some more official plugins - [ohmyzsh plugins](https://github.com/ohmyzsh/ohmyzsh/wiki/Plugins)

after all these steps type  

```
source ~/.zshrc
```