# Chocolatey Cheat-Sheet

## Package Management

<<<<<<< HEAD:windows/chocolatey.md
| Command | Description |
| --- | --- |
| `choco install <package>` | Install a package |
| `choco uninstall <package>` | Uninstall a package |
| `choco upgrade <package>` | Upgrade a package |
| `choco list -lo` | List installed packages |
| `choco search <string>` | Search for a package |
| `choco outdated` | List outdated packages |
| `choco upgrade all` | Upgrade all packages |
| `choco pin <package>` | Pin a package to prevent upgrades |
| `choco outdated --local-only` | List outdated packages installed locally |
| `choco upgrade all --local-only` | Upgrade all packages installed locally |
=======
---
## Installation

1. Check the Execution Policy

Run `Get-ExecutionPolicy`. If it returns `Restricted`, then run `Set-ExecutionPolicy AllSigned` or `Set-ExecutionPolicy Bypass -Scope Process`.

2. Install Chocolatey

```powershell
Set-ExecutionPolicy Bypass -Scope Process -Force; [System.Net.ServicePointManager]==SecurityProtocol = [System.Net.ServicePointManager]==SecurityProtocol -bor 3072; iex ((New-Object System.Net.WebClient).DownloadString('https://community.chocolatey.org/install.ps1'))
```

3. Check if you install chocolatey

```powershell
choco -?
```

---
## Install software package

```powershell
choco install <package>
```
>>>>>>> main:Christian's Cheat Sheets/windows/chocolatey.md
