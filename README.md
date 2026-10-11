English | [简体中文](README_CN.md)

# Dotfiles

Personal dotfiles managed with [chezmoi](https://www.chezmoi.io/) and versioned with Git. The repository lives at `~/dotfiles`, separately from the machine maintenance workspace.

Agent maintenance instructions: [AGENTS.md](AGENTS.md).

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
| SSH | Templates that retrieve server details and public keys from 1Password; private keys are used by the SSH agent |
| Fcitx5 / Rime | Input method list, selected behavior settings and the Rime Ice schema selection |

Application caches, sessions, downloaded tools and font files are outside this configuration baseline. The Fontconfig backup file was also excluded.

## Repository layout

```text
~/dotfiles/
├── README.md                  # English overview
├── README_CN.md               # Chinese overview
├── AGENTS.md                  # Instructions for agents maintaining this repository
├── LICENSE                    # MIT license for original contributions
├── .chezmoiroot               # Selects home/ as the source root
├── docs/
│   ├── SETUP.md               # Detailed setup and maintenance notes (Chinese)
│   ├── SSH.md                 # SSH and 1Password setup and validation (Chinese)
│   └── INPUT-METHOD.md        # Fcitx5 / Rime configuration and restore dependencies (Chinese)
└── home/
    ├── .chezmoi.toml.tmpl     # Generates the local chezmoi configuration
    ├── dot_bashrc.tmpl
    ├── dot_bash_profile
    ├── dot_profile
    ├── dot_local/             # Personal Rime configuration patch
    ├── private_dot_ssh/       # SSH configuration and public key templates
    └── dot_config/            # Application configuration
```

Only `home/` is the deployment source root. The README files and `docs/` stay in the repository.

chezmoi names encode target attributes: `dot_` becomes a leading dot, `.tmpl` marks a template, and `symlink_` declares a symbolic link. `private_` restricts target permissions; it does not encrypt repository contents. Ordinary source files and their deployed copies are separate files.

## Initialize an existing checkout

On a compatible machine with Git and chezmoi installed, clone the repository if `~/dotfiles` does not exist yet:

```bash
git clone https://github.com/szchan/dotfiles.git "$HOME/dotfiles"
```

A complete restore also requires the 1Password desktop app and `op` CLI, CLI integration and the SSH agent enabled, and access to the referenced items. The SSH configuration currently uses the Linux agent socket path; adapt it before using another operating system. See [SSH setup notes (Chinese)](docs/SSH.md).

With the repository available at `~/dotfiles`:

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

The managed applications and their dependencies need to be installed separately. Run `mise install` from the home directory to install the declared tools. In Neovim, `:Lazy restore` restores plugin revisions from `lazy-lock.json`. The required font families and their current Arch package names are recorded in the setup notes. Restoring the input method also requires Fcitx5, Rime and the Rime Ice schema; desktop startup and input module integration are configured separately. See [input method notes (Chinese)](docs/INPUT-METHOD.md).

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

If an application changes a plain configuration file directly, import that file again with `chezmoi add`. Examples include the mise tool list and Neovim's plugin lockfile. For `.bashrc`, edit the source template and preserve its conditional logic. Maintain the SSH configuration and public key as source templates too, so rendered values do not replace the 1Password references.

To choose the proxy setting again:

```bash
chezmoi init --source "$HOME/dotfiles" --prompt
chezmoi diff ~/.bashrc
chezmoi apply ~/.bashrc
```

Disabling the setting removes proxy exports from the generated file; it does not clear variables in already running shells. See the setup notes for the cleanup command.

## Documentation and next steps

See [Setup and maintenance notes (Chinese)](docs/SETUP.md) for initialization details, configuration mappings, font dependencies, commit history, validation results and recovery instructions.

As of 2026-10-11, the first batch is complete and backed up to the public GitHub repository [szchan/dotfiles](https://github.com/szchan/dotfiles). The second-batch SSH and 1Password templates have been verified, committed and pushed as `89603c1`, and the user confirmed successful logins to both servers. Five Fcitx5 / Rime configuration files have been imported and verified against the local files; this input method configuration and documentation await commit and push. Deployment testing on another machine is pending. 1Password uses `account` mode with `prompt = false`; GitHub authentication continues to use `gh`.

## License

Original configuration changes and documentation in this repository are licensed under the [MIT License](LICENSE), copyright 2026 Shaoze Chen.

Included third-party files retain their original licenses. The Neovim starter includes an [Apache-2.0 license](home/dot_config/nvim/LICENSE), which is preserved.
