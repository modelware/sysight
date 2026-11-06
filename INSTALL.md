# Installing Sysight

Sysight comes as one command, `sysight`, and a VS Code extension that carries it. The KerML 1.0
and SysML 2.0 standard libraries are built in, so nothing else is needed.

| | Command line | VS Code |
|---|---|---|
| macOS (Apple silicon, Intel) | [install script](#macos-and-linux) | [extension](#vs-code) |
| Linux (x86-64, ARM; glibc 2.35 or later) | [install script](#macos-and-linux) | [extension](#vs-code) |
| Windows (x86-64; ARM under emulation) | [install script](#windows) | [extension](#vs-code) |

## macOS and Linux

```sh
curl -fsSL https://github.com/modelware/sysight/releases/latest/download/install.sh | sh
```

This downloads the latest release for your machine, checks it against the release's
`SHA256SUMS`, and installs `sysight` in `~/.local/bin`. If that directory is not on your `PATH`,
the script says so and shows the line to add, for example:

```sh
echo 'export PATH="$HOME/.local/bin:$PATH"' >> ~/.zshrc    # or ~/.bashrc, ~/.profile
```

Options, set before `sh`:

```sh
curl -fsSL https://github.com/modelware/sysight/releases/latest/download/install.sh | SYSIGHT_VERSION=0.1.0 sh
curl -fsSL https://github.com/modelware/sysight/releases/latest/download/install.sh | SYSIGHT_INSTALL_DIR=/usr/local/bin sh
```

| Variable | Meaning | Default |
|---|---|---|
| `SYSIGHT_VERSION` | the version to install | the latest |
| `SYSIGHT_INSTALL_DIR` | where to put `sysight` | `~/.local/bin` |
| `SYSIGHT_DOWNLOAD_BASE` | download from a mirror instead of GitHub | |

## Windows

In PowerShell:

```powershell
powershell -c "irm https://github.com/modelware/sysight/releases/latest/download/install.ps1 | iex"
```

This installs `sysight.exe` in `%LOCALAPPDATA%\sysight\bin` and adds that directory to your
user `PATH`; open a new terminal afterwards. The same variables apply, set first:

```powershell
$env:SYSIGHT_VERSION = "0.1.0"
irm https://github.com/modelware/sysight/releases/latest/download/install.ps1 | iex
```

## VS Code

Until the extension is in the VS Code Marketplace, install it from a release:

1. From the [latest release](https://github.com/modelware/sysight/releases/latest), download
   the `.vsix` for your platform:

   | Platform | File |
   |---|---|
   | macOS, Apple silicon | `sysight-darwin-arm64-<version>.vsix` |
   | macOS, Intel | `sysight-darwin-x64-<version>.vsix` |
   | Linux, x86-64 | `sysight-linux-x64-<version>.vsix` |
   | Linux, ARM | `sysight-linux-arm64-<version>.vsix` |
   | Windows | `sysight-win32-x64-<version>.vsix` |

2. In VS Code, run **Extensions: Install from VSIX…** from the Command Palette and choose the
   file; or, in a terminal, `code --install-extension sysight-<platform>-<version>.vsix`.

The extension carries its own `sysight`, so it works without the command-line install. To use
`sysight` in a terminal too (checks, queries, scripts, AI agents), install the command as above;
the extension can use that one instead through the `sysight.path` setting.

## Manual download

Each release also has an archive per platform (`sysight-<target>.tar.gz`, or `.zip` on Windows)
holding the binary, its license and the third-party notices. Check it against `SHA256SUMS`
(`shasum -a 256 -c SHA256SUMS --ignore-missing`), unpack it, and put `sysight` on your `PATH`.

The binaries are not yet signed. On macOS, a binary downloaded with a browser (not with the
install script) is blocked the first time; allow it with
`xattr -d com.apple.quarantine /path/to/sysight`. On Windows, SmartScreen may ask you to confirm
a browser-downloaded `sysight.exe` (*More info → Run anyway*).

## Check that it works

```sh
sysight --version
cd my-model          # any folder with .sysml or .kerml files
sysight check        # starts the folder's model server and reports diagnostics
```

No configuration is needed: the folder is the workspace, and the standard libraries are used.
Sysight writes its server's state to `.sysight/` in that folder (add it to your `.gitignore`).
`sysight help` lists every command.

## Update

Run the install command again; it replaces `sysight` with the latest release. Then
`sysight restart` in a workspace whose server is running, so it uses the new version. For the
extension, install the new `.vsix` the same way.

## Uninstall

- **macOS, Linux:** `rm ~/.local/bin/sysight` (or where you installed it), and the cache
  `rm -rf ~/.cache/sysight`.
- **Windows:** delete `%LOCALAPPDATA%\sysight` and remove its `bin` from your user `PATH`.
- **VS Code:** uninstall the extension from the Extensions view.
- In your models, the `.sysight/` folders can be deleted.

## Problems

Search or open an [issue](https://github.com/modelware/sysight/issues), with the output of
`sysight --version`, your platform, and what the install printed.
