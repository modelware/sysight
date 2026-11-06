# Sysight: SysML v2 Insight

[Sysight](https://www.modelware.io/sysight), by [Modelware](https://www.modelware.io), checks, evaluates and queries SysML v2 and
KerML models, from the command line, VS Code and AI agents (MCP). The standard libraries are
built in.

## Install

### VS Code extension

1. From the [latest release](https://github.com/modelware/sysight/releases/latest), download the
   `.vsix` for your platform: `darwin-arm64` (Apple silicon Mac), `darwin-x64` (Intel Mac),
   `linux-x64`, `linux-arm64` or `win32-x64`.
2. In VS Code, run **Extensions: Install from VSIX…** and pick the file.

The extension includes the `sysight` command it needs. Open a folder with `.sysml` or `.kerml`
files to start.

### Command line

macOS, Linux:

```sh
curl -fsSL https://github.com/modelware/sysight/releases/latest/download/install.sh | sh
```

Windows (PowerShell):

```powershell
powershell -c "irm https://github.com/modelware/sysight/releases/latest/download/install.ps1 | iex"
```

To update, run the install command again (or install the new `.vsix`). To pin a version, set
`SYSIGHT_VERSION=X.Y.Z`.

## Use

```sh
cd my-model
sysight check        # diagnostics
sysight help         # all commands
```

## Issues

[Report a bug or request a feature](https://github.com/modelware/sysight/issues).

## License

Free to use, including by companies and consultants. Reselling, redistributing or building
products on it needs a license from Modelware (info@modelware.io). See [LICENSE.md](LICENSE.md).
