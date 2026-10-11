English | [简体中文](README_CN.md)

# Dotfiles

Personal dotfiles managed with [chezmoi](https://www.chezmoi.io/) and versioned with Git. The repository lives at `~/dotfiles`, separately from the machine maintenance workspace.

The current baseline has been verified on one Omarchy/Linux machine. The repository is intended to support multiple machines; additional platform and machine-specific configuration is still in progress.

## Managed configuration

| Component | What is included |
| --- | --- |
| Ghostty | Fonts, window preferences, key bindings, Omarchy theme reference and a Linux crash workaround |
| herdr | A small set of UI preferences, with omitted settings using official defaults |
| Git | Identity, aliases and GitHub/Gist credential helpers that find `gh` through `PATH` |
| Bash | `.bashrc`, `.bash_profile`, `.profile`, optional dependency loading and a per-machine proxy switch |
| mise | Tool version requests and upgrade preferences |
| Neovim | Lua configuration, plugin lockfile and the Omarchy theme symlink |
| Fontconfig | Monospace font preferences and Simplified Chinese font fallback rules |

Application caches, sessions, downloaded tools and font files are outside this configuration baseline. The Fontconfig backup file was also excluded.

## Repository layout

```text
~/dotfiles/
├── README.md                  # English overview
├── README_CN.md               # Chinese overview
├── .chezmoiroot               # Selects home/ as the source root
├── docs/
│   └── SETUP.md               # Detailed setup and maintenance notes (Chinese)
└── home/
    ├── .chezmoi.toml.tmpl     # Generates the local chezmoi configuration
    ├── dot_bashrc.tmpl
    ├── dot_bash_profile
    ├── dot_profile
    └── dot_config/            # Application configuration
```

Only `home/` is the deployment source root. The README files and `docs/` stay in the repository.

chezmoi names encode target attributes: `dot_` becomes a leading dot, `.tmpl` marks a template, and `symlink_` declares a symbolic link. Ordinary source files and their deployed copies are separate files.

## Initialize an existing checkout

With the repository already available at `~/dotfiles` and Git and chezmoi installed:

```bash
chezmoi init --source "$HOME/dotfiles"
chezmoi diff
```

Initialization generates `~/.config/chezmoi/chezmoi.toml` and asks whether this machine should use the local proxy at `127.0.0.1:7897`. The default is disabled; subsequent initialization reuses the saved choice.

Review the proposed changes before applying them:

```bash
chezmoi apply
chezmoi diff
```

The managed applications and their dependencies need to be installed separately. Run `mise install` from the home directory to install the declared tools. In Neovim, `:Lazy restore` restores plugin revisions from `lazy-lock.json`. The required font families and their current Arch package names are recorded in the setup notes.

For a machine without Omarchy, adapt the theme configuration before applying the complete baseline. The Neovim theme symlink depends on Omarchy's current theme directory, and Ghostty includes `async-backend = io_uring`. Git identity is currently shared configuration; most mise tools request `latest`, while Node is pinned to `26.8.2`.

## Everyday workflow

Edit a source configuration, inspect the changes, then apply them:

```bash
EDITOR=nvim chezmoi edit ~/.config/ghostty/config
chezmoi diff ~/.config/ghostty/config
chezmoi apply ~/.config/ghostty/config
```

Review and commit the source changes:

```bash
git -C ~/dotfiles add home/
git -C ~/dotfiles diff --cached
git -C ~/dotfiles commit -m "Describe the configuration change"
```

If an application changes a plain configuration file directly, import that file again with `chezmoi add`. Examples include the mise tool list and Neovim's plugin lockfile. For `.bashrc`, edit the source template and preserve its conditional logic.

To choose the proxy setting again:

```bash
chezmoi init --source "$HOME/dotfiles" --prompt
chezmoi diff ~/.bashrc
chezmoi apply ~/.bashrc
```

Disabling the setting removes proxy exports from the generated file; it does not clear variables in already running shells. See the setup notes for the cleanup command.

## Documentation and next steps

See [Setup and maintenance notes (Chinese)](docs/SETUP.md) for initialization details, configuration mappings, font dependencies, commit history, validation results and recovery instructions.

As of 2026-10-11, the first batch is complete. Remote Git backup, SSH configuration, 1Password secret templates and deployment testing on another machine are pending. The initializer declares 1Password `account` mode with `prompt = false`; the current dotfiles do not retrieve secrets from 1Password. GitHub authentication uses `gh`.
