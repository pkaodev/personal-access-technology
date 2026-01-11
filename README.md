#!/usr/bin/env bash
# curl -fsSL https://raw.githubusercontent.com/pkaodev/personal-access-technology>/main/README.md | bash
set -e

# -----------------------------
# CONFIG
# -----------------------------
PRIVATE_REPO="pkaodev/_pat"
INSTALL_DIR="$HOME/.pat/"

# -----------------------------
# FUNCTIONS
# -----------------------------

install_gh_cli() {
    echo "[+] GitHub CLI not found. Installing..."
    if [[ "$(uname)" == "Darwin" ]]; then
        # macOS
        if ! command -v brew >/dev/null 2>&1; then
            echo "Homebrew not found. Please install Homebrew first: https://brew.sh/"
            exit 1
        fi
        brew install gh
    else
        # Linux (Debian/Ubuntu example)
        sudo apt update
        sudo apt install -y gh
    fi
}

authenticate() {
    echo "[+] Authenticating with GitHub..."
    gh auth login --hostname github.com --git-protocol ssh
}

clone_or_update_repo() {
    if [[ -d "$INSTALL_DIR/.git" ]]; then
        echo "[+] Repository exists. Fetching updates..."
        cd "$INSTALL_DIR"
        git fetch origin
    else
        echo "[+] Cloning private repository..."
        gh repo clone "_pat" "$INSTALL_DIR"
        cd "$INSTALL_DIR"
    fi
}

checkout_machine_branch() {
    MACHINE_BRANCH="machine/$(hostname -s)"
    echo "[+] Switching to machine-specific branch: $MACHINE_BRANCH"
    git checkout -B "$MACHINE_BRANCH"
}

install_app() {
    echo "[+] Installing app..."
    if [[ -f "Makefile" ]]; then
        make install
    else
        echo "No Makefile detected. Please add installation commands."
    fi
}

# -----------------------------
# MAIN SCRIPT
# -----------------------------

# Step 1: Ensure gh CLI
if ! command -v gh >/dev/null 2>&1; then
    install_gh_cli
fi

# Step 2: Authenticate
if ! gh auth status >/dev/null 2>&1; then
    authenticate
else
    echo "[+] Already authenticated with GitHub."
fi

# Step 3: Clone or update private repo
clone_or_update_repo

# Step 4: Checkout machine branch
checkout_machine_branch

# Step 5: Install
install_app

echo "[+] Installation complete!"