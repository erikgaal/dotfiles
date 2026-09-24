# dotfiles

Managed with [chezmoi](https://chezmoi.io).

## New machine

```sh
sh -c "$(curl -fsLS get.chezmoi.io)" -- init --apply erikgaal
```

`chezmoi init` prompts for which **profiles** this machine should have, then
for your identity (name, email, GPG signing key). If the 1Password CLI is
installed it offers to read those from 1Password instead. Nothing secret lives
in this repo.

### Fedora / Asahi Linux

To read identity from 1Password, install and sign in to `op` first (optional;
without it `chezmoi init` just prompts):

```sh
sudo rpm --import https://downloads.1password.com/linux/keys/1password.asc
sudo tee /etc/yum.repos.d/1password.repo <<'EOF'
[1password]
name=1Password Stable Channel
baseurl=https://downloads.1password.com/linux/rpm/stable/$basearch
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://downloads.1password.com/linux/keys/1password.asc
EOF
sudo dnf install -y 1password-cli
op account add --address my.1password.com
eval "$(op signin --account my.1password.com)"
```

Then:

```sh
sh -c "$(curl -fsLS get.chezmoi.io)" -- init --apply erikgaal
```

There is no Homebrew on Linux. Packages come from dnf (plus the mise,
1Password and Ghostty COPR repos), `mise` for tools Fedora lacks, and Flathub.
The install script also makes zsh the login shell. Mac-only apps (AeroSpace,
Raycast, TablePro, …) are simply absent; see `packages.linux` for what each
profile installs there.

### 1Password (optional)

Identity is read once, at `chezmoi init` time, from an item named `Dotfiles`
in the `Private` vault of the `my.1password.com` account. The resolved values
are written to `~/.config/chezmoi/chezmoi.toml`, which is *not* part of this
repo — so `chezmoi apply` does not need 1Password afterwards.

Create it once with:

```sh
op item create --category='Secure Note' --title='Dotfiles' --vault='Private' \
  'name[text]=Erik Gaal' \
  'email[text]=me@erikgaal.nl' \
  'signingkey[text]=<gpg-key-fingerprint>' \
  'work name[text]=<work name>' \
  'work email[text]=<work email>'
```

The two `work *` fields are only read when the `work` profile is enabled.
Leave the signing key empty when prompted to disable commit signing.

## Profiles

`core` is always applied. Everything else is opt-in per machine, chosen at
`chezmoi init` and stored in the local config.

| profile   | what it brings |
|-----------|----------------|
| `core`    | zsh + antidote, starship, mise, uv, git/gh, CLI tooling, Ghostty, 1Password CLI |
| `dev`     | PHP 8.2/8.4 + composer, Postgres, Zed, OrbStack (Docker on Linux), TablePro, act/actionlint |
| `ai`      | Claude, ChatGPT, ccusage, herdr, worktrunk |
| `desktop` | AeroSpace, Ice, Raycast, browsers, fonts, mac utilities |
| `work`    | Superscript git identity, Tuple, Linear |
| `hobby3d` | Blender, Godot, Inkscape, Scribus, PrusaSlicer |
| `audio`   | BlackHole, rekordbox, switchaudio-osx |

Change profiles later by editing `~/.config/chezmoi/chezmoi.toml` and running
`chezmoi apply`. Package installs re-run automatically when the list changes.

Note: `brew bundle` installs but never uninstalls. Turning a profile *off*
stops future installs; remove the packages by hand if you want them gone.

## Layout

- `.chezmoidata/packages.yaml` — packages, grouped by profile
- `.chezmoidata/defaults.yaml` — macOS `defaults write` settings
- `.chezmoiscripts/` — brew / dnf+flatpak+mise installs, macOS defaults, Dock setup
- `.chezmoiexternal.toml.tmpl` — antidote on Linux (brew provides it on macOS)
- `.chezmoiignore` — gates whole files on profiles
