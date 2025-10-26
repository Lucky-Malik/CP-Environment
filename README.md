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
