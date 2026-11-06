# Sysight: SysML v2 Insight

Sysight, by [Modelware](https://www.modelware.io), brings insight to SysML v2 and KerML models: a
model server per workspace that reports diagnostics, evaluates expressions and verifies
requirements, answers SPARQL queries, and serves your editor (VS Code), your pipeline (the
command line) and your AI agents (MCP). The KerML 1.0 and SysML 2.0 standard libraries are built
in.

This repository holds Sysight's releases, documentation and issue tracker.

## Install

```sh
# macOS, Linux
curl -fsSL https://github.com/modelware/sysight/releases/latest/download/install.sh | sh
```

```powershell
# Windows (PowerShell)
powershell -c "irm https://github.com/modelware/sysight/releases/latest/download/install.ps1 | iex"
```

Runs on macOS (Apple silicon and Intel), Linux (x86-64 and ARM) and Windows (x86-64).
`SYSIGHT_VERSION=X.Y.Z` installs a given version; downloads are checked against the release's
`SHA256SUMS`. Or download an archive from the [latest release](https://github.com/modelware/sysight/releases/latest).

**VS Code:** download the extension for your platform from the
[latest release](https://github.com/modelware/sysight/releases/latest) (for example
`sysight-darwin-arm64-<version>.vsix` on an Apple silicon Mac) and install it with
*Extensions: Install from VSIX…*. It carries the `sysight` command.

Options, updating, uninstalling and troubleshooting: [INSTALL.md](INSTALL.md).

## Get started

```sh
cd my-model
sysight check                                   # diagnostics for the whole workspace
sysight evaluate Vehicle::mass --unit kg        # evaluate an expression
sysight query 'SELECT ?part WHERE { ?part a sysml:Part }'
sysight help                                    # every command
```

## Issues and questions

Report bugs and ask for features in [Issues](https://github.com/modelware/sysight/issues). A
small model that shows the problem, and the output of `sysight --version`, help a lot.

## License

Sysight is proprietary software, free to use directly by individuals and organizations,
companies and consultants included. Reselling, redistributing, hosting it for others or building
products on it needs a license from Modelware (info@modelware.io). See [LICENSE.md](LICENSE.md).
