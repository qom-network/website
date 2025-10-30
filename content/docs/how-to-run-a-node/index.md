---
title: "How to Run a Node"
description: "Step-by-step guide for running a node using BabelFish-VM."
slug: "how-to-run-a-node"
toc: true
---
#  BabelFish CLI – Installation Guide
## For download　[BabelFish CLI](https://github.com/WizardLatino/babefish-vm-test)

##  Prerequisites

### macOS (Intel & Apple Silicon)

Requires **Homebrew**. If not installed, run:

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"`
```
Then install **Qt6**:

```bash
brew install qt@6`

```

### Ubuntu

```bash
# Qt6 Core and Network components are required
sudo apt update
sudo apt install qt6-base-dev libqt6network6

```

---

## Download & Installation

### 1. Download the appropriate version

#### macOS Intel (x64)

```bash
curl -L -o babelfish-macos-intel.tar.gz https://github.com/WizardLatino/babefish-vm-test/releases/latest/download/babelfish-macos-intel.tar.gz
````
#### macOS Apple Silicon (ARM64)
```bash
curl -L -o babelfish-macos-apple-silicon.tar.gz https://github.com/WizardLatino/babefish-vm-test/releases/latest/download/babelfish-macos-apple-silicon.tar.gz`
```
#### Ubuntu (x64)

```bash
curl -L -o babelfish-ubuntu.tar.gz https://github.com/WizardLatino/babefish-vm-test/releases/latest/download/babelfish-ubuntu.tar.gz`
```
---

### 1.1 Verify File Integrity

#### macOS

````bash
shasum -a 256 babelfish-*.tar.gz
````
#### Ubuntu

```bash
sha256sum babelfish-*.tar.gz
```
Compare the hash with the one listed on the [GitHub Releases page](https://github.com/WizardLatino/babefish-vm-test/releases/latest).  
**Do not proceed if the checksums do not match.**

---

### 2. Extract and Verify

````bash
# Extract the archive
tar -xzf babelfish-*.tar.gz

# Enter the extracted directory
cd babelfish-*/

# Make the binary executable
chmod +x babelfish

# Verify it works by displaying help
./babelfish help
````
**Note:** The folder includes a helper script `babelfish-launcher.sh` for locating non-standard Qt installations.



### 3. Install to System

````bash
# Create user bin directory (Linux/macOS use .local/bin)
mkdir -p ~/.local/bin

# Copy the binary
cp babelfish ~/.local/bin/

# Ensure executable permissions
chmod +x ~/.local/bin/babelfish
````

### 4. Add to PATH (Permanent)

#### macOS (zsh)

````bash
echo 'export PATH="$HOME/.local/bin:$PATH"' >> ~/.zshrc source ~/.zshrc`
````
#### Ubuntu (bash)

```bash
echo 'export PATH="$HOME/.local/bin:$PATH"' >> ~/.bashrc source ~/.bashrc`
````


### 5. Verify Installation

```bash
# From any directory
babelfish help
```


##  Troubleshooting

If the `babelfish` command is not found after installation:

1. Verify that `~/.local/bin` is in your PATH: `echo $PATH`
2. Restart your terminal
3. Check permissions: `ls -la ~/.local/bin/babelfish`


---

### Qt6 Issues

If you encounter Qt6-related errors:

- Ensure Qt6 is properly installed via your package manager.
    
- For custom Qt installations, use the included helper script:
    
    `./babelfish-launcher.sh`
    

---

### Debugging  Issues
Run with the debug flag for detailed logs:

```bash
babelfish --debug help
```


## Uninstall

```bash
rm ~/.local/bin/babelfish`
```



## You can help us improve Babelfish by adding issues, ideas, or bug reports to the  [Babelfish Project Board!!](https://github.com/orgs/deep-thought-labs/projects/1)
