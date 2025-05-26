# Ansible Ubuntu 22.04 LTS Desktop Setup

This repository contains an Ansible playbook to automate the setup of a personal Ubuntu 22.04 LTS (Jammy Jellyfish) desktop environment tailored for development, reverse engineering (RE), and general productivity.

**Important:** This playbook is specifically designed and tested for **Ubuntu 22.04 LTS**. Due to differences in package availability, system configurations, or dependencies, it is not guaranteed to work correctly on other Ubuntu versions or other Linux distributions without modification.

## Purpose

The goal of this playbook is to provide a consistent, repeatable, and automated way to configure a fresh Ubuntu 22.04 LTS installation with a suite of essential tools and preferred settings, significantly reducing manual setup time.

## Prerequisites

Before running this playbook, ensure you have:

1.  A **fresh installation of Ubuntu 22.04 LTS Desktop**. A minimal installation is recommended, as the playbook will install necessary components.
2.  An **active internet connection** on the target machine (for downloading packages and cloning repositories).
3.  **`git` and `ansible` installed** on the target machine. You can install them with:
    ```bash
    sudo apt update
    sudo apt install git ansible -y
    ```
4.  The user running the playbook must have `sudo` privileges, as the playbook will prompt for the sudo password to perform administrative tasks.

## How to Use

1.  **Clone this repository** to your target Ubuntu 22.04 LTS machine:
    ```bash
    git clone https://github.com/k-karakatsanis/ansible-ubuntu-22-lts-desktop-setup.git
    cd ansible-ubuntu-22-lts-desktop-setup
    ```

2.  **Review and Customize Variables (Optional):**
    Open the `playbook.yml` file and review the `vars:` section. You might want to customize:
    * `python_versions`: List of Python minor versions to install via `pyenv` (e.g., `["3.12", "3.11"]`).
    * `default_python_version`: The default global Python version `pyenv` should set (e.g., `"3.12"`).
    * `install_binwalk`: Set to `true` or `false` to toggle Binwalk installation.
    * `ghidra_version` & `ghidra_release_date_for_zip`: **Crucial!** Ensure these match an existing downloadable release file from the official Ghidra GitHub releases page. The playbook constructs the download URL based on these.

3.  **Run the Ansible Playbook:**
    From within the cloned repository directory, execute the following command:
    ```bash
    ansible-playbook -i "localhost," --connection=local playbook.yml --ask-become-pass
    ```
    * You will be prompted for your `sudo` password (BECOME password).
    * The playbook will take a significant amount of time to run, especially on the first execution due to Python compilations and large downloads like Ghidra.
    
    **Note on Verbosity/Troubleshooting:**
    If you encounter issues or want to see more detailed output of what Ansible is doing, you can run the playbook with increased verbosity. Add `-v` for verbose, `-vv` for more verbose, or `-vvv` for debug-level verbosity to the command:
    ```bash
    ansible-playbook -i "localhost," --connection=local playbook.yml --ask-become-pass -vvv
    ```

## What This Playbook Does

This playbook automates the installation and basic configuration of the following:

**System & Core Utilities:**
* Updates `apt` cache.
* Installs essential build tools (`build-essential`, `pkg-config`, etc.).
* Installs common command-line utilities: `curl`, `wget`, `git`, `htop`, `tree`, `jq`, `hexedit`, `fzf`.
* Installs Uncomplicated Firewall (`ufw`).

**Firewall Configuration (UFW):**
* Sets default policies: deny incoming, allow outgoing.
* Enables the firewall. (No SSH allow rule is added by default for a local desktop setup).

**Shell Environment (Zsh):**
* Installs Zsh.
* Sets Zsh as the default shell for the invoking user.
* Installs Zsh plugins via `apt`: `zsh-autosuggestions`, `zsh-syntax-highlighting`.
* Installs Zsh plugin via Git: `zsh-history-substring-search`.
* Clones Powerlevel10k theme from Git to `~/powerlevel10k`.
* Configures `~/.zshrc` to:
    * Set up history.
    * Enable Zsh completion system with case-insensitivity.
    * Source installed Zsh plugins and Powerlevel10k.
    * Initialize `pyenv`.
    * Add common `ls` and `git` aliases.

**Text Editor (Vim):**
* Installs `vim-gtk3` (a more featured Vim).
* Creates an enhanced `~/.vimrc` with common settings (syntax highlighting, line numbers, indentation, mouse support, persistent undo, basic colorscheme).
* Creates the `~/.vim/undodir` directory for persistent undo.

**Containerization (Docker):**
* Installs Docker CE (Community Edition) from Docker's official repository.
* Installs Docker Compose plugin and Buildx plugin.
* Adds the invoking user to the `docker` group (requires logout/login to take effect).
* Ensures the Docker service is started and enabled.

**Development Environment (VS Code):**
* Installs Visual Studio Code from Microsoft's official repository.
* Installs the following extensions for the invoking user:
    * Python (`ms-python.python`)
    * Pylance (`ms-python.vscode-pylance`)
    * Black Formatter (`ms-python.black-formatter`)
    * Pyright (`ms-pyright.pyright`)
    * Ruff (`charliermarsh.ruff`)
    * ShellCheck (`timonwong.shellcheck`)
    * Hex Editor (`ms-vscode.hexeditor`)
    * Remote - SSH (`ms-vscode-remote.remote-ssh`)
    * Dev Containers (`ms-vscode-remote.remote-containers`)
    * Bash IDE (`mads-hartmann.bash-ide-vscode`)
    * Container Tools (`ms-azuretools.vscode-containers`)
    * GitLens (`eamodio.gitlens`)

**Python Version Management (pyenv):**
* Clones `pyenv` and `pyenv-virtualenv` from GitHub.
* Installs specified Python versions (defined in `python_versions` variable).
* Sets a global default Python version (defined in `default_python_version` variable).

**Reverse Engineering & Firmware Analysis Tools:**
* Enables i386 architecture for multiarch support.
* Installs common 32-bit and 64-bit dependencies often required by IDA Pro.
* Installs `binwalk` (if `install_binwalk` variable is `true`).
* Installs OpenJDK 21 (for Ghidra).
* Downloads and installs Ghidra to `/opt/ghidra_VERSION_PUBLIC`.
* Creates a symlink `/usr/local/bin/ghidra` for easy launching.

**Connectivity:**
* Installs Broadcom STA/WL Wi-Fi driver (`bcmwl-kernel-source`).

**System Cleanup:**
* Runs `apt autoremove` at the end to remove unused packages.

## Post-Execution Steps

**CRITICAL:** After the playbook successfully completes, you **MUST LOG OUT of your Ubuntu session and then LOG BACK IN.** A full **REBOOT** is recommended if the Wi-Fi driver was installed or changed.

This is necessary for:
* Zsh to become your active default shell.
* PATH modifications for `pyenv` in `~/.zshrc` to take full effect.
* Zsh plugins and Powerlevel10k to be sourced correctly.
* Your user to be recognized as part of the `docker` group (to run `docker` commands without `sudo`).
* Wi-Fi driver to become fully active.
* VS Code extensions to be fully available in new VS Code instances.

## Verification

After logging back in (or rebooting):

* **Zsh Shell:** Open a terminal. It should be Zsh. Type `echo $SHELL` (should output `/usr/bin/zsh`).
* **Zsh Features:** Test autosuggestions, syntax highlighting, and history search (e.g., Ctrl+R with fzf, or Up/Down arrows for history-substring-search).
* **Powerlevel10k:** The `p10k configure` wizard should run on the first launch of Zsh. Follow the prompts to customize your prompt. If it doesn't run, you can type `p10k configure` manually.
* **Python:**
    * `pyenv versions` (should show the versions you installed).
    * `python --version` or `python3 --version` (should show your `pyenv` global default).
* **Docker:** `docker run hello-world` (this should run without needing `sudo`).
* **Ghidra:** Type `ghidra` in the terminal.
* **VS Code:** Launch VS Code and check the "Extensions" panel to see if your specified extensions are installed and active.
* **Vim:** Open Vim and check for new settings (line numbers, mouse support, colorscheme).
* **Wi-Fi:** Check your network manager for available Wi-Fi networks.

## License

This project is licensed under the MIT License - see the `LICENSE` file for details.

---
