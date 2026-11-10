# Sysight: SysML v2 Insight

[Sysight](https://www.modelware.io/sysight), by [Modelware](https://www.modelware.io), checks, evaluates and queries SysML v2 and
KerML models, from the command line, VS Code and AI agents (MCP). The standard libraries are
built in.

## Highlights

- **The whole model checked as you type**: syntax, names, the specification's rules, and the
  constraints and requirements that fail, across every file.
- **Values and verdicts where they are written**: attribute values, roll-ups and units
  computed, and each requirement satisfied, violated or undetermined, with why.
- **Ask anything in SPARQL**: in the editor's panel, as tables, charts and values, with
  completion of the model's classes and properties, and functions that compute values and
  verdicts inside a query.
- **The model as a graph**: elements and relationships drawn in an editor tab, to explore,
  filter and save with the model.
- **Agents that work on the live model**: MCP tools to read, compute, change and show graphs,
  with every change answered by its diff and the problems it added or removed.
- **One model for everyone**: the editor, agents, the command line, CI and your own programs
  (over HTTP) share one server per model, unsaved edits included, answering in milliseconds.
- **No configuration, no lock-in**: the folder is the model, the files stay the model, and it
  converts to and from RDF.

## Install

### VS Code extension

Install **Sysight: SysML v2 Insight** (`modelware.sysight`) from the Extensions view, from the
[VS Code Marketplace](https://marketplace.visualstudio.com/items?itemName=modelware.sysight), or,
in VSCodium, Cursor and other editors that use it, from
[Open VSX](https://open-vsx.org/extension/modelware/sysight). Updates install automatically.

The extension includes the `sysight` command it needs. Open a folder with `.sysml` or `.kerml`
files to start.

Without access to a marketplace, download the `.vsix` for your platform from the
[latest release](https://github.com/modelware/sysight/releases/latest) (`darwin-arm64`,
`darwin-x64`, `linux-x64`, `linux-arm64` or `win32-x64`) and run **Extensions: Install from
VSIX…**.

### Command line

macOS, Linux:

```sh
curl -fsSL https://github.com/modelware/sysight/releases/latest/download/install.sh | sh
```

Windows (PowerShell):

```powershell
powershell -c "irm https://github.com/modelware/sysight/releases/latest/download/install.ps1 | iex"
```

### AI agents (MCP)

Install the command line (above), then register `sysight mcp` with your agent, for example in
Claude Code:

```sh
claude mcp add sysight -- sysight mcp --workspace /path/to/model
```

Other clients take the same command in their MCP configuration:
`{"command": "sysight", "args": ["mcp", "--workspace", "/path/to/model"]}`.

### CI

In a GitHub Actions workflow (or any CI on macOS, Linux or Windows):

```yaml
- run: |
    curl -fsSL https://github.com/modelware/sysight/releases/latest/download/install.sh | sh
    echo "$HOME/.local/bin" >> "$GITHUB_PATH"
- run: sysight check
```

To update the command, run the install command again. To pin a version, set
`SYSIGHT_VERSION=X.Y.Z`.

## Get started

```sh
cd my-model
sysight check                                      # problems in the whole model
sysight evaluate 'Vehicle::mass' --unit kg         # a value
sysight query 'SELECT ?p WHERE { ?p a sysml:Part }' # anything, in SPARQL
```

The [manual](MANUAL.md) covers using Sysight in VS Code, with agents, on the command line, in
CI and from programs (its HTTP API): queries, values and verdicts, graphs, changing the model,
and settings. Agents read it too, through the MCP tool `manual`, and `sysight manual` prints it.

## Example

The [Apollo 11 dashboard](https://www.modelware.io/apollo-11-sysml-v2/) is built with Sysight
from Airbus's [SysML v2 model of Apollo 11](https://github.com/airbus/apollo-11-sysml-v2): the
timeline, functions, budgets, requirements, cost and model quality. Each page was made with a
Jupyter notebook that sends SPARQL queries to the model's Sysight server and charts the answers
(see the manual's [Programs and notebooks](MANUAL.md#programs-and-notebooks)).

## Performance

On [Airbus's Apollo 11 model](https://github.com/airbus/apollo-11-sysml-v2) (28 files, about
7,200 lines, with the standard libraries: about 143,000 facts), on an Apple M4, end to end from
the `sysight` command:

| Operation | Cold (server starting) | Warm (server running) |
|---|---:|---:|
| Check the whole model | 121 ms | 24 ms |
| Check after an edit | | 27 ms |
| SPARQL query | 195 ms | 5 ms |

The first run after installing or updating Sysight also prepares the standard libraries, once:
about 1 s.

The model stays loaded and current as files change, so the editor, agents and commands answer
in milliseconds.

## Support

Sysight is free to use, and its standard support is through [issues](https://github.com/modelware/sysight/issues):
report problems, request features and ask questions there. For organizations that need committed
response times, priority fixes or help adopting it, Modelware offers enterprise support
packages: info@modelware.io.

## License

Free to use, including by companies and consultants. Reselling, redistributing or building
products on it needs a license from Modelware (info@modelware.io). See [LICENSE.md](LICENSE.md).
