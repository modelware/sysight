# Sysight manual

Welcome! This manual shows you how to use Sysight on your SysML v2 and KerML models. To install
it, see the [README](README.md).

New here? Read [The model and its server](#the-model-and-its-server), then jump to where you
work: [VS Code](#in-vs-code), [AI agents](#with-ai-agents) or
[the command line](#on-the-command-line).

## Contents

- **[Getting started](#getting-started):** [The model and its server](#the-model-and-its-server)
- **[Where you use it](#where-you-use-it):** [In VS Code](#in-vs-code) ·
  [With AI agents](#with-ai-agents) · [On the command line](#on-the-command-line) ·
  [Other editors](#other-editors)
- **[Working with a model](#working-with-a-model):** [Values and verdicts](#values-and-verdicts) ·
  [Queries](#queries) · [Graphs](#graphs) · [Changing the model](#changing-the-model) ·
  [Text and RDF](#text-and-rdf)
- **[Automating](#automating):** [Programs and notebooks](#programs-and-notebooks) ·
  [In CI](#in-ci)
- **[Reference](#reference):** [Commands](#commands) · [MCP tools](#mcp-tools) ·
  [VS Code commands and settings](#vs-code-commands-and-settings) ·
  [Query vocabulary](#query-vocabulary) · [Settings](#settings)

## Getting started

What Sysight needs from you: very little.

### The model and its server

Your model is a folder of `.sysml` and `.kerml` files, and Sysight reads it with no setup.

Open the folder in VS Code, or run a `sysight` command in it. The standard libraries are built
in.

One Sysight server per model keeps it loaded and current. VS Code, agents and commands all use
it, so they see the same model, unsaved edits included, and answer in milliseconds.

- **The model's folder** is the nearest one upward with a `sysight.toml` (it can be empty), or
  else the folder you are in.
- **The server starts and stops on its own:** it starts when needed and stops after 15 idle
  minutes. `sysight start` and `sysight stop` control it yourself.
- **After updating Sysight,** run `sysight restart` if an editor is still connected.
- **To see what's running:** `sysight inspect` for this model, `sysight status` for all.
- **Staying up to date:** when a newer Sysight is out, a command in your terminal says so in
  one line, with how to update. It checks GitHub at most once a day and sends nothing about your
  model; `SYSIGHT_NO_UPDATE_CHECK=1` turns it off. The VS Code extension updates itself.

## Where you use it

The same model in VS Code, with agents, in a terminal, or in another editor.

### In VS Code

The Sysight extension checks your model as you type and shows values and verdicts where they are
written.

- **Problems** for the whole model, in the Problems view.
- **Hover** for what an element is, its value, or a requirement's verdict and why.
- **Inline hints** with computed values (`= 2214000 kg`) and what the language implies.
- **Navigation, rename, completion and formatting**, across files and into the libraries.
- **The Sysight panel**, next to the chat: ask in SPARQL and see tables, charts and values (see
  [Queries](#queries)).
- **The graph view** (see [Graphs](#graphs)).
- **This manual:** *Sysight: Open Manual*, or the book icon in the panel.

Commands and settings: [VS Code commands and settings](#vs-code-commands-and-settings).

### With AI agents

Agents can read, compute with and change your model through Sysight, on the same live model you
see.

In **VS Code**, agents such as Copilot get Sysight's tools with nothing to set up. In **Claude
Code**, add it once per model:

```sh
claude mcp add sysight -- sysight mcp --workspace /path/to/model
```

To share it with your team, put this in the model's `.mcp.json`. Other agents take the same
command:

```json
{ "mcpServers": { "sysight": { "command": "sysight", "args": ["mcp"] } } }
```

Then ask in plain words: "Which requirements fail, and why?" or "Show me how the requirements
trace to the parts." The tools are listed in [MCP tools](#mcp-tools).

### On the command line

The `sysight` command checks, computes and queries your model from a terminal.

```sh
sysight check                                       # problems in the whole model
sysight evaluate 'Vehicle::mass' --unit kg          # a value
sysight verify 'Vehicle::massLimit'                 # is this requirement met?
sysight query 'SELECT ?p WHERE { ?p a sysml:Part }' # anything, in SPARQL
```

`sysight help` lists the commands; they are also in [Commands](#commands).

### Other editors

Any editor with Language Server Protocol support can use Sysight by running `sysight attach`.

For example, Neovim:

```lua
vim.lsp.config('sysight', { cmd = { 'sysight', 'attach' }, filetypes = { 'sysml', 'kerml' }, root_markers = { 'sysight.toml', '.git' } })
vim.lsp.enable('sysight')
```

## Working with a model

Computing, asking, drawing and changing.

### Values and verdicts

Sysight computes your model's values, with units, and decides whether constraints and
requirements are met.

```sh
sysight evaluate 'Vehicle::mass' --unit kg
sysight verify 'Vehicle::massLimit'
```

A verdict is **satisfied**, **violated** or **undetermined**, with the reason.

- Evaluate an expression inside an element: `sysight evaluate 'e1.thrust * 2' --context 'M::Rocket'`.
- Units are checked: a mass asked for in metres is an error, not a wrong number.
- The same values and verdicts appear in hover, in the Problems view, in queries and for agents.

### Queries

Ask your model anything in SPARQL, with values and verdicts in the answers.

```sparql
SELECT ?requirement (eval:verdict(?requirement) AS ?verdict) (eval:why(?requirement) AS ?why)
WHERE { ?requirement a sysml:Requirement }
```

- **Familiar names:** classes follow the keywords (`sysml:Part`, `sysml:Requirement`), relations
  the notation (`kerml:typedBy`, `kerml:ownedFeature`).
- **No prefixes to declare:** `kerml:`, `sysml:` and `eval:` are ready.
- **Kinds are understood:** asking for parts finds every kind of part.
- **Shortcuts and functions** such as `sysml:satisfies` and `eval:value` (see
  [Query vocabulary](#query-vocabulary)).

Save queries as `.rq` files in the model's `queries` folder. The panel lists them, and
`sysight query @queries/<name>.rq` runs one.

### Graphs

See your model as a graph in VS Code, and explore how its elements relate.

- *Sysight: Open Model Graph* draws the whole model, or a file or folder (right-click it in the
  Explorer).
- *Sysight: Show in Graph* (right-click in the editor) draws an element and its neighbors.
- *Open as graph* draws a CONSTRUCT or DESCRIBE answer from the panel.
- Agents can show you one too.

Hover to highlight neighbors, double-click to open an element, drag to pin. *Save* keeps the
view, layout included, as a query file to commit with the model.

### Changing the model

Name what to change, and Sysight edits the files for you, touching only that.

`sysight update`, and the agents' `update` tool, take a list of changes:

```json
{ "operations": [
  { "op": "create",  "owner": "Vehicles::Vehicle", "text": "part battery : Battery[1];" },
  { "op": "set",     "element": "Vehicles::Vehicle::mass", "property": "value", "value": "1450 [kg]" },
  { "op": "rename",  "element": "Vehicles::Engine", "name": "PowerUnit" },
  { "op": "replace", "element": "Vehicles::Wheel", "text": "part def Wheel { attribute size; }" },
  { "op": "move",    "element": "Vehicles::Spare", "owner": "Vehicles::Storage" },
  { "op": "delete",  "element": "Vehicles::OldPart" }
]}
```

- **create** adds members to an element, next to another (`after`, `before`), or to a file.
- **set** changes one property: `value`, `multiplicity`, `typedBy`, `specializes`, `doc`,
  `isAbstract`, and more. `null` removes it.
- **rename** renames an element and every reference to it.
- **replace** rewrites an element from new text.
- **move** moves an element, with its notes, and keeps references working.
- **delete** removes an element and its notes. If something refers to it, Sysight stops,
  unless you add `"references": "detach"` (remove them) or `"leave"` (keep them).

Changes apply all or nothing, and the answer shows each file's diff and the problems added or
fixed. `--dry-run` previews them; `--if-version <n>` refuses them if the model changed since.

### Text and RDF

Save your model as standard notation or RDF, and bring models in from RDF or a `.kpar` archive.

```sh
sysight export --format text      # each file in the standard style
sysight export --format ttl       # the model as RDF (also trig, nq)
sysight import model.trig         # add files from RDF or a .kpar archive
```

`//` notes are not part of the model, so exports leave them out. `import` never overwrites your
files unless you add `--force`.

## Automating

Your model in programs, notebooks and CI.

### Programs and notebooks

Any program or notebook can query and change your model over HTTP.

Run `sysight start` first. The server's address is in the model's `.sysight/server.json`.

**Queries** go to `/query`, as standard SPARQL. In Python:

```python
import json, requests

url = json.load(open(".sysight/server.json"))["url"]
query = "SELECT ?r (eval:verdict(?r) AS ?verdict) WHERE { ?r a sysml:Requirement }"
rows = requests.post(f"{url}/query", data={"query": query},
                     headers={"Accept": "application/sparql-results+json"}).json()
```

In JavaScript, or from the shell:

```js
const res = await fetch(`${url}/query`, {
  method: "POST",
  headers: { "Content-Type": "application/sparql-query", Accept: "text/csv" },
  body: "SELECT ?p WHERE { ?p a sysml:Part }",
});
```

```sh
curl -s "$URL/query" -H 'Accept: text/csv' --data-urlencode 'query=SELECT ?p WHERE { ?p a sysml:Part }'
```

**Keep your queries in the model,** as `.rq` files in `queries/`, and pass values with a
`VALUES` clause added at the end (`VALUES ?name { "dryMass" }`).

**Changes** go to `/update`, as the JSON of [Changing the model](#changing-the-model). Pass the
version you read from `/inspect`, so a change made meanwhile by someone else is refused:

```python
version = requests.get(f"{url}/inspect").json()["version"]
change = {"ifVersion": version, "operations": [
    {"op": "set", "element": "Vehicle::mass", "property": "value", "value": "1450 [kg]"}]}
requests.post(f"{url}/update", json=change).raise_for_status()
```

Answers come as JSON, CSV, TSV or XML. `/check`, `/evaluate`, `/verify`, `/export` and
`/inspect` work like the commands.

### In CI

Check your model, and its requirements, on every change.

Install Sysight in a step (see the [README](README.md)), then:

```yaml
- run: sysight check                          # fails on errors
- run: sysight verify 'Vehicle::massLimit'    # fails if not satisfied
```

## Reference

Commands, tools, settings and query vocabulary, in one place.

### Commands

Every `sysight` command; `sysight help <command>` gives the details.

| Command | What it does |
|---|---|
| `check [files]` | Reports problems |
| `evaluate <target> [--context <element>] [--unit <unit>]` | Computes values |
| `verify <target>` | Decides constraints and requirements |
| `query <sparql \| @file.rq> [--format csv]` | Answers a SPARQL query |
| `export [--format text\|ttl\|trig\|nq]` | Writes the model as notation or RDF |
| `update <changes.json> [--dry-run]` | Changes the model |
| `import <file> [--force]` | Adds files from RDF or a `.kpar` archive |
| `start`, `stop`, `restart` | Controls the model's server |
| `inspect`, `status` | Shows this model's server, or all servers |
| `attach`, `mcp` | Connects an editor, or an agent |
| `manual [section \| --all]` | Shows this manual |
| `version`, `license` | Shows the version or the license |

Every command also takes `--workspace <dir>` and `--json`.

### MCP tools

The tools Sysight gives AI agents.

| Tool | What it does |
|---|---|
| `describe` | Shows an element as notation |
| `export` | Shows a file, or lists the model's files |
| `check` | Reports problems |
| `query` | Answers a SPARQL query |
| `evaluate`, `verify` | Computes values and verdicts |
| `update` | Changes the model |
| `graph` | Shows you a graph in VS Code |
| `manual` | Gives a section of this manual |

The manual is also available as MCP resources: `sysight://manual`, and one per section.

### VS Code commands and settings

The extension's commands and settings.

| Command | What it does |
|---|---|
| Sysight: Show Query Panel | Opens the panel (⌘Enter or Ctrl+Enter runs the query) |
| Sysight: New Session in an Editor Tab | Starts a session in its own tab |
| Sysight: Run Query in Sysight | Runs the selection or query file |
| Sysight: Open Model Graph | Draws the model as a graph |
| Sysight: Show in Graph | Draws an element and its neighbors |
| Sysight: Open Manual | Opens this manual |
| Sysight: Show Model Server | Shows the model's server |
| Sysight: Restart Model Server | Restarts it |
| Sysight: Show Output | Shows the extension's log |

| Setting | What it does |
|---|---|
| `sysight.path` | The `sysight` command to use (default: the one included) |
| `sysight.queriesFolder` | Where the panel saves queries (default: `queries`) |
| `sysight.inlayHints.values` | Computed values inline (on) |
| `sysight.inlayHints.impliedSpecializations` | What elements specialize implicitly (on) |
| `sysight.inlayHints.defaultVisibility` | `private` before imports without one (on) |
| `sysight.inlayHints.memberVisibility` | `public` before other members (off) |
| `sysight.inlayHints.effectiveNames` | Names that unnamed elements take (on) |

### Query vocabulary

The shortcuts and functions you can use in queries.

| Shortcut | Holds when |
|---|---|
| `kerml:type` | a feature has a type, declared or inherited |
| `kerml:owns`, `kerml:within` | an element is inside another, at any depth |
| `kerml:conformsTo` | a type is a kind of another, at any depth |
| `sysml:performs`, `sysml:exhibits` | an element performs an action or exhibits a state |
| `sysml:satisfies`, `sysml:satisfiedBy` | an element satisfies a requirement |
| `sysml:verifies` | a verification case verifies a requirement |
| `kerml:dependsOn` | an element depends on another |
| `kerml:connectedTo` | two features are connected |

| Function | Gives |
|---|---|
| `eval:value` | the value: a number, Boolean or text |
| `eval:unit` | its unit |
| `eval:text` | the value as shown, or why it cannot be computed |
| `eval:verdict`, `eval:why` | a verdict, and why |
| `eval:name`, `eval:qualifiedName` | an element's name and qualified name |

Each function takes an element, a qualified name or an expression, and optionally the elements
to read it in and a unit.

### Settings

A `sysight.toml` at the model's root holds its settings, all optional.

```toml
libraries = ["../sysml.library"]   # other libraries: .kpar archives or folders
exclude = ["build"]                # folders that are not part of the model
port = 7878                        # a fixed address for programs (default: any free port)
idle-minutes = 15                  # how long an idle server keeps running (0: until stopped)

[format]
indent = 4                         # spaces per level, or "tab"
break-conditions = 3               # break and/or chains this long, one condition per line
```
