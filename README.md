# dotfiles

Managed with [chezmoi](https://chezmoi.io).

## New machine

```sh
sh -c "$(curl -fsLS get.chezmoi.io)" -- init --apply erikgaal
```

`chezmoi init` prompts for which **profiles** this machine should have, then
reads identity out of 1Password. Nothing secret lives in this repo.

### 1Password prerequisite

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

## Profiles

`core` is always applied. Everything else is opt-in per machine, chosen at
`chezmoi init` and stored in the local config.

| profile   | what it brings |
|-----------|----------------|
| `core`    | zsh + antidote, starship, mise, uv, git/jj/gh, CLI tooling, Ghostty, 1Password CLI |
| `dev`     | PHP 8.2/8.4 + composer, Postgres, Zed, OrbStack, TablePro, act/actionlint |
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
- `.chezmoiscripts/` — brew install, macOS defaults, Dock setup
- `.chezmoiignore` — gates whole files on profiles
