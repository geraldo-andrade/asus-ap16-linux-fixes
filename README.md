# ASUS ProArt P16 fixes

Ansible playbooks for the 2025 ASUS ProArt P16 H7606WX (AMD Ryzen AI 9 HX 370 + NVIDIA) on Pop!_OS 24.04.

## What it does

- `main.yml` — entry point; runs both playbooks below in one invocation.
- `hibernate.yml` — swapfile-based hibernation on the encrypted root: `resume=` / `resume_offset=` kernel options via kernelstub, fstab swapfile entry, polkit override restoring hibernate for active local users, plain-dm-crypt cryptswap disable. Includes pre-flight DMI/CPU/GPU/OS checks (skip with `hardware_check: false`).
- `keyboard-backlight.yml` — loads `asus_nb_wmi` in the initramfs, init-premount hook lights the keyboard backlight at the LUKS passphrase prompt, seeds the systemd-backlight save file so it stays lit at the login screen.
- `files/` — support files: `10-enable-hibernate.rules` (polkit) and `asus-kbd-backlight` (initramfs hook).

All playbooks are idempotent — safe to re-run after kernel/initramfs updates.

## Installing Ansible (Homebrew)

1. Install Homebrew:

   ```sh
   /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
   ```

2. Post-install setup — add Homebrew to your PATH (Linux default prefix; the installer prints these exact commands, adjust for your shell). The shell/terminal emulator is unknown, so this appends the setup to *every* profile that exists in your home folder: `~/.profile` (login shells), `~/.bashrc` (interactive bash), `~/.zprofile` / `~/.zshrc` (zsh). Missing files are skipped; if you have several, all of them get the lines:

   ```sh
   for f in ~/.profile ~/.bashrc ~/.zprofile ~/.zshrc; do
       [ -f "$f" ] || continue
       echo '# Homebrew' >> "$f"
       echo 'eval "$(/home/linuxbrew/.linuxbrew/bin/brew shellenv)"' >> "$f"
   done
   eval "$(/home/linuxbrew/.linuxbrew/bin/brew shellenv)"
   ```

   Re-running appends harmless duplicates. Open a new terminal and confirm with `brew --version`.

3. Install Ansible:

   ```sh
   brew install ansible
   ```

   Verify: `ansible-playbook --version`.

## Running

Prerequisites: sudo access on the machine; Ansible installed (see above if `ansible-playbook` is missing).

```sh
ansible-playbook -i localhost, -c local -K main.yml
```

- `-c local` — required: this host has no sshd, so the default ssh transport fails with "Connection refused". (Alternative: define an inventory entry with `ansible_connection: local`.)
- `-K` / `--ask-become-pass` — prompts for your sudo password at the `BECOME password:` prompt. Use this instead of `--ask-pass`, which is for SSH authentication and does not apply here.

To run a single fix, replace `main.yml` with `hibernate.yml` or `keyboard-backlight.yml`.

### Notes

- The playbooks must run with root privileges (`become: true`).
- On first run, `hibernate.yml` may create a swapfile matching your RAM size and rebuild the initramfs; allow a few minutes.
- Re-run after kernel updates so initramfs picks up the backlight hook and resume parameters.
