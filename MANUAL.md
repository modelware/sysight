# Sysight manual

Welcome! This manual shows you how to use Sysight on your SysML v2 and KerML models. To install
it, see the [README](README.md).

## Contents

- [Getting started](#getting-started)
  - [Your model](#your-model)
  - [In VS Code](#in-vs-code)
  - [With AI agents](#with-ai-agents)
  - [On the command line](#on-the-command-line)
- [Using Sysight](#using-sysight)
  - [Check your model](#check-your-model)
  - [Compute values and verdicts](#compute-values-and-verdicts)
  - [Ask questions](#ask-questions)
  - [See it as a graph](#see-it-as-a-graph)
  - [Change the model](#change-the-model)
  - [Export and import](#export-and-import)
  - [Use it from programs](#use-it-from-programs)
  - [Check it in CI](#check-it-in-ci)
- [Reference](#reference)
  - [Commands](#commands)
  - [MCP tools](#mcp-tools)
  - [VS Code commands and settings](#vs-code-commands-and-settings)
  - [Query vocabulary](#query-vocabulary)
  - [Settings](#settings)

## Getting started

Open your model and use it from wherever you work.

### Your model

Your model is a folder of `.sysml` and `.kerml` files, and Sysight reads it with no setup.

The standard libraries are built in. One Sysight server per model keeps it loaded and current,
so VS Code, agents and commands all see the same model, unsaved edits included, and answer in
milliseconds.

- **The model's folder** is the nearest one upward with a `sysight.toml` (it can be empty), or
  else the folder you are in.
- **The server starts and stops on its own:** it starts when needed and stops after 15 idle
  minutes. `sysight start` and `sysight stop` control it yourself.
- **Staying up to date:** when a newer Sysight is out, a command in your terminal says so in one
  line, with how to update. It checks GitHub at most once a day and sends nothing about your
  model; `SYSIGHT_NO_UPDATE_CHECK=1` turns it off. The VS Code extension updates itself.

### In VS Code

Open your model's folder, and Sysight checks it as you type.

You get problems in the Problems view, values and verdicts on hover and inline, navigation,
rename, completion and formatting, the Sysight panel for questions, and the graph view. This
manual opens with *Sysight: Open Manual*, or the book icon in the panel.

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

Then ask in plain words: "Which requirements fail, and why?"

### On the command line

The `sysight` command works in any terminal, in your model's folder.

```sh
sysight check      # problems in the whole model
sysight help       # every command
```

Other editors with Language Server Protocol support can use Sysight too, by running
`sysight attach`. For example, Neovim:

```lua
vim.lsp.config('sysight', { cmd = { 'sysight', 'attach' }, filetypes = { 'sysml', 'kerml' }, root_markers = { 'sysight.toml', '.git' } })
vim.lsp.enable('sysight')
```

## Using Sysight

The things you can do with your model, wherever you work.

### Check your model

Sysight checks your whole model for problems: syntax, names that do not resolve, the language's
rules, and constraints or requirements that fail.

- **In VS Code**, problems appear in the Problems view as you type, for every file, open or not.
- **On the command line:** `sysight check`, or `sysight check <files>` for some files only.
- **Agents** use the `check` tool, and check again after changing the model.

### Compute values and verdicts

Sysight computes your model's values, with units, and decides whether constraints and
requirements are met.

- **In VS Code**, hover over an element for its value or verdict, and see values inline
  (`= 2214000 kg`).
- **On the command line:**

  ```sh
  sysight evaluate 'Vehicle::mass' --unit kg
  sysight verify 'Vehicle::massLimit'
  ```

- **Agents** use the `evaluate` and `verify` tools.

A verdict is **satisfied**, **violated** or **undetermined**, with the reason. Units are checked:
a mass asked for in metres is an error, not a wrong number. To evaluate an expression inside an
element: `sysight evaluate 'e1.thrust * 2' --context 'M::Rocket'`.

### Ask questions

Ask your model anything in SPARQL, with values and verdicts in the answers.

```sparql
SELECT ?requirement (eval:verdict(?requirement) AS ?verdict) (eval:why(?requirement) AS ?why)
WHERE { ?requirement a sysml:Requirement }
```

- **In VS Code**, write it at the bottom of the Sysight panel and see the answer as a table, a
  chart or a value. Your queries and sessions are kept.
- **On the command line:** `sysight query '<query>'`, or `sysight query @queries/<name>.rq`.
- **Agents** use the `query` tool, or just ask in plain words.

Queries are easy to write:

- **Familiar names:** classes follow the keywords (`sysml:Part`, `sysml:Requirement`), relations
  the notation (`kerml:typedBy`, `kerml:ownedFeature`).
- **No prefixes to declare:** `kerml:`, `sysml:` and `eval:` are ready.
- **Kinds are understood:** asking for parts finds every kind of part.
- **Shortcuts and functions** such as `sysml:satisfies` and `eval:value` (see
  [Query vocabulary](#query-vocabulary)).

Save queries as `.rq` files in the model's `queries` folder, to keep them with the model.

### See it as a graph

See your model as a graph in VS Code, and explore how its elements relate.

- *Sysight: Open Model Graph* draws the whole model, or a file or folder (right-click it in the
  Explorer).
- *Sysight: Show in Graph* (right-click in the editor) draws an element and its neighbors.
- *Open as graph* draws a CONSTRUCT or DESCRIBE answer from the panel.
- **Agents** can show you one with the `graph` tool: "show me how the requirements trace to the
  parts".

Hover to highlight neighbors, double-click to open an element, drag to pin. *Save* keeps the
view, layout included, as a query file to commit with the model.

### Change the model

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

- **create** adds members to an element, next to another (`after`, `before`), or to a file (a
  new file is created).
- **set** changes one property: `value`, `multiplicity`, `typedBy`, `specializes`, `doc`,
  `isAbstract`, and more. `null` removes it.
- **rename** renames an element and every reference to it.
- **replace** rewrites an element from new text.
- **move** moves an element, with its notes, and keeps references working.
- **delete** removes an element and its notes. If something refers to it, Sysight stops,
  unless you add `"references": "detach"` (remove them) or `"leave"` (keep them).

Changes apply all or nothing, and the answer shows each file's diff and the problems added or
fixed. `--dry-run` previews them; `--if-version <n>` refuses them if the model changed since.

### Export and import

Save your model as standard notation or RDF, and bring models in from RDF or a `.kpar` archive.

```sh
sysight export --format text      # each file in the standard style
sysight export --format ttl       # the model as RDF (also trig, nq)
sysight import model.trig         # add files from RDF or a .kpar archive
```

`//` notes are not part of the model, so exports leave them out. `import` never overwrites your
files unless you add `--force`.

### Use it from programs

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

**Changes** go to `/update`, as the JSON of [Change the model](#change-the-model). Pass the
version you read from `/inspect`, so a change made meanwhile by someone else is refused:

```python
version = requests.get(f"{url}/inspect").json()["version"]
change = {"ifVersion": version, "operations": [
    {"op": "set", "element": "Vehicle::mass", "property": "value", "value": "1450 [kg]"}]}
requests.post(f"{url}/update", json=change).raise_for_status()
```

Answers come as JSON, CSV, TSV or XML. `/check`, `/evaluate`, `/verify`, `/export` and
`/inspect` work like the commands.

### Check it in CI

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
