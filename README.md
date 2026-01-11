#!/usr/bin/env bash
# curl -fsSL https://raw.githubusercontent.com/pkaodev/personal-access-technology/main/install.sh | bash
set -euo pipefail

install_homebrew() {
    NONINTERACTIVE=1 /bin/bash -c \
        "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)" \
        >/dev/null
    eval "$(/opt/homebrew/bin/brew shellenv 2>/dev/null || /usr/local/bin/brew shellenv)"
}

install_gh_cli() {
    if [[ "$(uname)" == "Darwin" ]]; then
        command -v brew >/dev/null 2>&1 || install_homebrew
        brew install gh >/dev/null
    else
        sudo apt-get update -qq
        sudo apt-get install -y gh >/dev/null
    fi
}

authenticate() {
    gh auth login \
        --hostname github.com \
        --git-protocol ssh \
        --web \
        --skip-ssh-key \
        >/dev/null
}

clone_or_update_repo() {
    if [[ -d "$HOME/.pat/.git" ]]; then
        git -C "$HOME/.pat" fetch -q origin
    else
        gh repo clone pkaodev/_pat "$HOME/.pat" >/dev/null
    fi
}

checkout_machine_branch() {
    git -C "$HOME/.pat" checkout -q -B "machine/$(hostname -s)"
}

install_app() {
    if [[ -f "$HOME/.pat/Makefile" ]]; then
        make -C "$HOME/.pat" remote-install >/dev/null
    else
        echo "Error: Makefile not found in repository." >&2
        exit 1
    fi
}

# -----------------------------
# MAIN
# -----------------------------

command -v gh >/dev/null 2>&1 || install_gh_cli
gh auth status --hostname github.com >/dev/null 2>&1 || authenticate
clone_or_update_repo
checkout_machine_branch
install_app

echo "PAT installation complete."
