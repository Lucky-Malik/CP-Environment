
# My Personal Competitive Programming Environment

> **Private single-developer setup — not for public GitHub upload**

> **Reason for private status:** this repository bundles local contest tool credentials, private judge configs, and proprietary resource files (problem sets, recorded test cases, personal credentials). Treat it as a personal workspace — **do not upload to any public remote**. Follow the local-only setup instructions below to reproduce on your own PC.

---

## Project overview

This project is a reproducible personal environment optimized for competitive programming practice and contest preparation. It centralizes: local problem libraries, fast compilation and run workflows, per-language templates, stress-testing harnesses, automated testing against custom input, an integrated editor/IDE setup (VS Code / Neovim), and helpful scripts to manage contests and mock up time-limited runs.

Although written so it *resembles* a single-repo project, it is deliberately designed as a private, single-person setup that contains sensitive files (API keys, private datasets, testcases). The README below explains how to recreate the environment locally on your PC (Windows / macOS / Linux) without pushing any secret material to GitHub.

---

## Key features

- Language templates and fast build scripts for C++, Java, Python, and Rust.
- One-command compile & run with input redirection and timeouts.
- Stress-test harness for randomized testing between reference and candidate solutions.
- Local problem library with tagging and metadata (difficulty, topic, tags).
- VS Code and Neovim recommended config files (keybindings, linters, snippets).
- Preconfigured Git hooks for local linting only (no remote push hooks that leak secrets).
- Optional Docker container for a fully reproducible toolchain.

---

## Tech stack

- Languages: Bash, Python (3.10+), C++ (17/20), Java 17, Rust (optional)
- Tools: gcc/clang, g++, make, gradle/maven (Java), cargo (Rust)
- Editors: Visual Studio Code (recommended) or Neovim
- Optional: Docker / docker-compose

---

## Security & privacy note (important)

This environment intentionally stores private problem sets and local judge data. Do **not** upload the folder to any public Git provider. If you want to share code, create a sanitized copy that removes the following before publishing:

- `secrets/` or files containing API keys
- `private_tests/` directory
- Any `.env` or `credentials.*` files
- Local judge binaries or proprietary problem PDFs

Use `git filter-repo` or `git rm --cached` + new commit to scrub history if necessary.

---

## How to reproduce on your PC (step-by-step)

> These instructions assume you have basic familiarity with a terminal and installing tools on your OS. Replace commands with the appropriate package manager commands for your platform.

### 0) Folder layout (what you'll create locally)

```
~/cpp-env/                 # root folder for the environment
├─ templates/              # language templates (cpp, java, py)
├─ scripts/                # run, compile, stress-test scripts
├─ problems/               # local problem files and testcases
├─ private_tests/          # large private input datasets (keep local only)
├─ editor-config/          # VSCode / Neovim settings & snippets
├─ docker/                 # optional Dockerfile + compose
└─ README.md               # this file (local copy)
```

### 1) Create project folder

```bash
mkdir -p ~/cpp-env && cd ~/cpp-env
git init
```

> NOTE: Do **not** add a remote. Keep the repo local. If you must push to a remote, use a private, access-controlled server (self-hosted GitLab or private GitHub repo with strict access) and remove secrets before pushing.

### 2) Install language toolchains

#### On Ubuntu/Debian

```bash
sudo apt update
sudo apt install -y build-essential gdb python3 python3-pip openjdk-17-jdk curl git
# Optional: rust
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
```

#### On macOS (Homebrew)

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
brew install gcc gdb python openjdk@17 rust
```

#### On Windows

Use WSL2 (recommended) and follow the Linux instructions, or install MinGW / MSVC toolchain and Python from python.org. VS Code + Remote - WSL extension works nicely.

### 3) Add language templates

Create a `templates/` folder and add starter templates. Example C++ template `templates/main.cpp`:

```cpp
#include <bits/stdc++.h>
using namespace std;
int main(){
    ios::sync_with_stdio(false);
    cin.tie(nullptr);
    // read input
    return 0;
}
```

Create helper script `scripts/run_cpp.sh`:

```bash
#!/usr/bin/env bash
# usage: ./run_cpp.sh file.cpp input.txt
g++ -std=gnu++17 -O2 -pipe "$1" -o tmp && timeout 2s ./tmp < "${2:-/dev/stdin}"; rm -f tmp
```

Make it executable: `chmod +x scripts/*.sh`.

### 4) Problem library structure

Inside `problems/` create directories per problem with `statement.md`, `in.txt`, `out.txt`, and metadata `meta.json`.

Example:

```
problems/array-sum/
  statement.md
  in.txt
  out.txt
  meta.json  # e.g. {"tags":["arrays","implementation"],"difficulty":1000}
```

### 5) Stress-testing harness

`scripts/stress_test.py` (simple idea): it runs a generator to produce random inputs, runs a reference solution and your candidate solution, compares outputs, and prints counterexample if mismatch. Keep the reference solution in `private_tests/reference/`.

### 6) Editor / IDE setup (VS Code recommended)

- Install extensions: C/C++ (ms-vscode.cpptools), Code Runner, Python, Java Extension Pack
- Place `editor-config/.vscode/settings.json` with recommended settings (format on save off for contests, custom tasks for build & run).

Example `tasks.json` tasks can run `${workspaceFolder}/scripts/run_cpp.sh` on the active file.

### 7) Optional Docker container

If you want EXACT reproducibility, build a small Dockerfile that installs compilers and sets `/workspace` as mount:

```dockerfile
FROM ubuntu:24.04
RUN apt update && apt install -y build-essential python3 openjdk-17-jdk
WORKDIR /workspace
CMD ["/bin/bash"]
```

Run with: `docker build -t cp-env . && docker run --rm -it -v "$PWD":/workspace cp-env`

### 8) Local-only git best practices

- Add `.gitignore` with entries for `private_tests/`, `secrets/`, `.env`.
- Never commit API keys. Use local-only `secrets/` and add it to `.gitignore`.
- For accidental commit of secrets, use `git filter-repo` to purge them from history.

---

## Usage examples

### Compile and run a C++ file with sample input

```bash
./scripts/run_cpp.sh templates/main.cpp problems/array-sum/in.txt
```

### Run a timed mock contest (30 minutes)

A simple script can open a timer and provide a chosen problem:

```bash
./scripts/mock_contest.sh problems/array-sum 30
```

### Add a new problem to your library

1. Create a new directory under `problems/`.
2. Add `statement.md`, provide at least `in.txt` and `out.txt` (sample tests).
3. Add tags to `meta.json`.

---

## Troubleshooting

- **Compilation errors**: ensure correct compiler flags. Use `-Wall -Wextra` while developing.
- **Timeouts**: increase `timeout` wrapper or optimize algorithm.
- **Editor integration**: check that `tasks.json` points to correct script paths.

---

## Why not upload this to GitHub?

This setup intentionally collects private judge data, local credentials, and large test archives. Uploading it publicly risks leaking private problem sets and keys. If you wish to publish parts of the setup (templates, scripts), create a sanitized export:

```bash
mkdir ../cpp-env-sanitized
cp -r templates scripts editor-config README.md ../cpp-env-sanitized/
# do not copy private_tests or secrets
```

Then create a new public repo with only the sanitized content.

---

## Final notes

This README gives a full, local-first blueprint to reproduce a personal competitive programming environment on your PC. If you want, I can now:

- generate `scripts/` files (run scripts, mock contest, stress tester) in this repo structure,
- produce `editor-config/.vscode/tasks.json` and sample `launch.json`,
- create the Dockerfile and `docker-compose.yml`.

Tell me which of those you'd like next and for which OS (Linux/macOS/Windows WSL).

=======
My Personal Competitive Programming Environment

This is a detailed look at my personal, highly-optimized environment for competitive programming. I've built this from the ground up to be as fast, minimal, and efficient as possible, integrating all my tools into a single, cohesive system.

A Note on Distribution

This project is a personal setup and is not intended for public distribution or hosting on platforms like GitHub. It involves deep system-level modifications, including a custom-compiled Linux kernel, a personal package repository, and hard-coded paths specific to my local machine.

The setup guide below is detailed, but it assumes you are starting with the project's complete archive file (cp-env-archive.tar.gz) on your local PC and are comfortable with advanced Linux system administration.

Core Features

I designed this system to automate every part of the competitive programming workflow:

CP Control Center (TUI): A central, full-screen terminal application (cp-cli) built in Python. It's the "cockpit" for everything. It fetches contest data from the Codeforces and CLIST APIs and provides an interactive menu to manage contests.

Custom-Tuned Linux Kernel: I maintain a custom-compiled Arch Linux kernel. I've stripped all unused drivers and modules and tuned the CPU scheduler specifically for desktop responsiveness and low-latency compilation.

Personal pacman Repository: All my custom tools, scripts, and dotfiles are compiled into packages (PKGBILD) and served from a personal pacman repository. This means I can install my cp-cli or my entire neovim config with a single pacman -S command.

"Contest Mode": A one-click script that prepares my entire desktop for a contest. It mutes notifications, sets a "focus" wallpaper, and (most importantly) triggers a custom systemd target (contest.target) that stops all non-essential services like Bluetooth, file syncing, etc., to free up 100% of system resources.

Automatic Project Scaffolder: When I select "Launch Contest" from the cp-cli TUI, a script automatically parses the contest URL, creates a directory for each problem (e.g., A/, B/, C/), downloads all sample cases as in1.txt, out1.txt, etc., and copies my base template.cpp into each folder.

Offline Documentation: Includes zeal for instant, offline access to C++, Python, and algorithm documentation.

Technology Stack

Operating System: Minimal Arch Linux

Kernel: Custom-compiled Linux Kernel

Package Management: pacman, PKGBILD

Core TUI: Python 3 + Textual

System Services: systemd

Scripting: bash, Python

GUI / Helpers: zenity (for the one-time setup wizard)

My Dotfiles: Neovim (for editing), Fish (as shell), tmux (for terminal management)

Local Setup Guide (From Archive)

This guide assumes you have the project archive (cp-env-archive.tar.gz) and are running a minimal Arch Linux installation.

Step 1: Prerequisites

You must be on an Arch Linux-based system. You must be comfortable compiling a kernel, managing systemd services, and editing pacman configuration files as root.

Step 2: Unpack the Project

Unpack the complete project archive.

# Unpack all kernel sources, scripts, and package builds
tar -xvf cp-env-archive.tar.gz -C /opt/my-cp-env
cd /opt/my-cp-env


Step 3: Compile and Install the Custom Kernel

This is the most critical step. I've included my pre-configured .config file.

cd /opt/my-cp-env/kernel

# Copy my kernel config to the source directory
cp my-kernel.config .config

# (Optional) Review my settings
make menuconfig

# Compile the kernel using all available cores
make -j$(nproc)

# Install the modules and the kernel
sudo make modules_install
sudo make install

# WARNING: You MUST reboot your system now to load the new kernel
# before proceeding.
sudo reboot


Step 4: Set Up and Use the Local pacman Repo

After rebooting into the new kernel, you need to set up the local pacman repository that contains all my custom tools.

Build All Packages: I have a simple build script that iterates through all PKGBILD files (for cp-cli, dotfiles, etc.) and builds them.

cd /opt/my-cp-env/pacman-repo
./build-all-packages.sh


Add Local Repo to pacman.conf: Edit /etc/pacman.conf as root and add this above the standard repositories:

[my-custom-repo]
SigLevel = Optional TrustAll
Server = File:///opt/my-cp-env/pacman-repo/


Sync and Install Tools: Now, sync pacman and install everything from the local repo.

# Sync the new repository
sudo pacman -Syy

# Install all my custom tools in one go
sudo pacman -S cp-cli my-dotfiles-fish my-dotfiles-neovim contest-mode zeal


Step 5: First-Time Setup Wizard

My dotfiles package includes a first-run script. Log out and log back in. A graphical wizard (zenity) should pop up asking for your Codeforces handle.

If it does not run, you can trigger it manually:

/usr/bin/first-run-setup.sh


This saves your handle to ~/.config/cp-cli/config.ini, which the TUI reads.

Step 6: Launch the Environment

The installation is complete. My fish shell config, neovim setup, and all tools are now active.

Simply open a terminal and run the main application:

cp-cli


The TUI will launch, fetch the latest contests, and you can start practicing or join a live contest.

>>>>>>> 519c28112c18e1f4f13297400189db2b116d49f4
