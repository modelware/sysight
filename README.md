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

To update, run the install command again (or install the new `.vsix`). To pin a version, set
`SYSIGHT_VERSION=X.Y.Z`.

## Use

### In VS Code

Open a folder with `.sysml` or `.kerml` files. As you type, Sysight checks the whole model
(the Problems view), and gives navigation, completion, rename, formatting and highlighting.
Hover an element for its computed value or a requirement's verdict; inlay hints show values and
what the language implies.

The **Sysight view**, docked beside the chat, is where you ask the model: write a SPARQL query
at the bottom (with completion of the model's classes and properties) and see the answer as a
table, chart or value. Queries are saved in the model's `queries/` folder, and conversations are
kept as sessions. A `.rq` file opens in the view with *Run in Sysight*.

### With AI agents

VS Code's agents (such as Copilot in agent mode) get Sysight's tools from the extension, with
nothing to set up: check *MCP: List Servers* for `sysight`.

In Claude Code, register it once per model, then ask in plain words ("which requirements fail,
and why?"):

```sh
claude mcp add sysight -- sysight mcp --workspace /path/to/model
```

Or add it to the model's `.mcp.json`, shared with everyone who opens it (other agents take the
same command in their MCP configuration):

```json
{ "mcpServers": { "sysight": { "command": "sysight", "args": ["mcp"] } } }
```

The tools: `check`, `query` (SPARQL), `evaluate`, `verify`, `describe`, `export` and `update`.

### On the command line

```sh
cd my-model
sysight check                                      # problems in the whole model
sysight evaluate 'Vehicle::mass' --unit kg         # a value
sysight verify 'Vehicle::massLimit'                # a constraint or requirement: satisfied?
sysight query 'SELECT ?p WHERE { ?p a sysml:Part }' # anything, in SPARQL
sysight export --format ttl                        # the model as RDF
sysight help                                       # every command
```

No configuration is needed: the folder is the model, and the standard libraries are built in.
The editor, agents and commands share one server per model, so they all see the same model,
unsaved edits included.

## Example

The [Apollo 11 dashboard](https://www.modelware.io/apollo-11-sysml-v2/) is built with Sysight
from Airbus's [SysML v2 model of Apollo 11](https://github.com/airbus/apollo-11-sysml-v2): the
timeline, functions, budgets, requirements, cost and model quality. Each page was made with a
Jupyter notebook that sends SPARQL queries to the model's Sysight server and charts the answers.

The server speaks the SPARQL 1.1 protocol, so any notebook or program can send it queries over
HTTP. Its address is in the model's `.sysight/server.json` (`sysight inspect` shows it):

```python
import json, requests

url = json.load(open(".sysight/server.json"))["url"] + "/query"
query = """
SELECT ?requirement (eval:verdict(?requirement) AS ?verdict) (eval:why(?requirement) AS ?why)
WHERE { ?requirement a sysml:Requirement }
"""
answer = requests.post(url, data={"query": query},
                       headers={"Accept": "application/sparql-results+json"}).json()
for row in answer["results"]["bindings"]:
    print(row["requirement"]["value"], row["verdict"]["value"])
```

The `kerml:`, `sysml:` and `eval:` prefixes need no declaring; `eval:` functions compute values
and verdicts in the query. Answers also come as CSV (`Accept: text/csv`), ready for pandas.

## Performance

On the Apollo 11 model (28 files, about 7,200 lines, with the standard libraries: about 143,000
facts), on an Apple M4, end to end from the `sysight` command:

| | Time |
|---|---|
| Check the whole model after an edit | 70 ms |
| A SPARQL query | 20 ms |
| Starting the model's server (once; it then stays running) | 0.3 s, 1 s the very first time |

The server keeps the model loaded and updates it incrementally as files change, so the editor,
agents and commands answer in milliseconds.

## Issues

[Report a bug or request a feature](https://github.com/modelware/sysight/issues).

## License

Free to use, including by companies and consultants. Reselling, redistributing or building
products on it needs a license from Modelware (info@modelware.io). See [LICENSE.md](LICENSE.md).
