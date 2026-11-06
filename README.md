# Sysight: SysML v2 Insight

Sysight, by [Modelware](https://www.modelware.io), checks, evaluates and queries SysML v2 and
KerML models, from the command line, VS Code and AI agents (MCP). The standard libraries are
built in.

## Install

macOS, Linux:

```sh
curl -fsSL https://github.com/modelware/sysight/releases/latest/download/install.sh | sh
```

Windows (PowerShell):

```powershell
powershell -c "irm https://github.com/modelware/sysight/releases/latest/download/install.ps1 | iex"
```

VS Code: download the `.vsix` for your platform from the
[latest release](https://github.com/modelware/sysight/releases/latest) and run
*Extensions: Install from VSIX…*.

To update, run the install command again. To pin a version, set `SYSIGHT_VERSION=X.Y.Z`.

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
