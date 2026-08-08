# AGENTS.md

## Installing Ansible (Homebrew)

If Ansible is not available (`which ansible-playbook` fails), install Homebrew first:

```sh
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

Then run the post-install PATH setup printed by the installer (Linux default prefix). The shell/terminal emulator is unknown, so append to *every* profile that exists in the home folder — `~/.profile` (login shells), `~/.bashrc` (interactive bash), `~/.zprofile` / `~/.zshrc` (zsh). If several exist, all of them get the setup; missing files are skipped:

```sh
for f in ~/.profile ~/.bashrc ~/.zprofile ~/.zshrc; do
    [ -f "$f" ] || continue
    echo '# Homebrew' >> "$f"
    echo 'eval "$(/home/linuxbrew/.linuxbrew/bin/brew shellenv)"' >> "$f"
done
eval "$(/home/linuxbrew/.linuxbrew/bin/brew shellenv)"
```

Finally install Ansible via Homebrew:

```sh
brew install ansible
```

On this machine Homebrew lives at `/home/linuxbrew/.linuxbrew` (brew 6.x, ansible 14.x) — `brew --version` and `ansible-playbook --version` confirm availability.

## Running the Ansible playbooks

- To run `main.yml` (or `hibernate.yml` / `keyboard-backlight.yml`) with root privileges, the become password is in `$HOME/.rootpass`. Read it from that file and feed it to the `BECOME password:` prompt.
- NEVER print, echo, log, or otherwise expose the password contents or the file's contents in the transcript, tool output, or any artifact.
- Use the local connection — this host has no sshd, so the default ssh transport fails with `Connection refused`. Always pass `-c local` (or an inventory entry with `ansible_connection: local`).
- Recommended invocation (keeps the secret out of argv and the transcript):

```sh
test -s "$HOME/.rootpass" && ansible-playbook -i localhost, -c local -K main.yml <<< "$(cat "$HOME/.rootpass")"
```

- Do not copy the password anywhere else or persist it in the repo.
