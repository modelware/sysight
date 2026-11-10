# Sysight manual

How to use Sysight on SysML v2 and KerML models: in VS Code, with AI agents, on the command
line, in CI and from programs. Installing it is in the [README](README.md).

Each `##` section below is one task. Agents read the same sections through the MCP tool
`manual`, and `sysight manual <section>` prints one in a terminal.

## The model and its server

A model is a folder of `.sysml` and `.kerml` files; Sysight needs no configuration, and the
standard KerML and SysML libraries are built in.

- **The model's folder:** the nearest folder upward with a `sysight.toml` (even an empty one);
  else the nearest one upward whose server is running; else the current one. The model is every
  `.sysml` and `.kerml` file under it, except hidden folders, `target`, `node_modules` and the
  paths `sysight.toml` excludes.
- **One server per model** holds it loaded and current. The editor, agents and commands all ask
  it, so they see the same model, unsaved editor text included, and answer in milliseconds.
- **Starting:** the first command, editor or agent that needs the server starts it. Its address
  is in the model's `.sysight/server.json`, its log in `.sysight/server.log`.
- **Staying current:** the server watches the files and updates the model as they change. Text
  an editor has open counts over the file on disk until the editor closes it (`check` marks such
  files "unsaved editor version").
- **Stopping:** a server started on demand stops after 15 minutes with no requests and no
  editor (`idle-minutes` in `sysight.toml`). `sysight start` keeps it running until
  `sysight stop`.
- **After updating Sysight:** a command of the new release replaces a server started on demand
  that no editor is attached to; otherwise it says so, and `sysight restart` switches it.
- `sysight inspect` shows the model's server (address, release, files, libraries, editors);
  `sysight status` lists every server of the machine; `sysight stop --all` stops them all.
  Servers listen on `127.0.0.1` only.

## In VS Code

The extension checks the whole model as you type and shows what it computes where it is written.

Open a folder with `.sysml` or `.kerml` files.

- **Diagnostics** for the whole model in the Problems view: syntax, names that do not resolve,
  the specification's rules, and asserted constraints and requirements that fail. Files you have
  not opened are checked too.
- **Navigation:** the outline, go to definition and find references, across files and into the
  libraries (their documents open read-only).
- **Completion** of the names in scope, for what the place asks: a type after `:`, a feature
  after `:>`, a package's members after `::`, a feature's after `.`; keywords and snippets at
  the start of a member.
- **Rename** (F2) a name declared in your files, everywhere it is spelled.
- **Formatting** in the notation's canonical style, keeping your notes and blank lines. Put
  `// sysight-format ignore` before a member to keep it as written.
- **Hover:** what an element is, what it specializes, its documentation, its computed value, and
  a constraint's or requirement's verdict (satisfied, violated or undetermined, and why).
- **Inlay hints:** computed values (`= 2214000 kg`), and what the language implies but the text
  does not say: the library elements a definition or usage specializes by its kind and place
  (`part def Engine` `:> Parts::Part`; a part in a part `:> Items::Item::subparts`), `private`
  before an import without a visibility (double-click to write it), and the name an unnamed
  element takes (`:>> engine.mass` is named `mass`).
- **The Sysight panel**, docked beside the chat: write a SPARQL query at the bottom and see the
  answer above it, as a table, chart, value, yes or no, or cards (see
  [Queries](#queries)). Completion offers query patterns, the properties that fit each
  element, and the `eval:` functions. Queries are saved by name in the model's `queries` folder,
  and each line of inquiry is kept as a session to rename, search and reopen.
- **Query files:** *Run in Sysight* above a `.rq` file runs it in the panel.
- **The graph view** (see [Graphs](#graphs)).

| Command | |
|---|---|
| Sysight: Show Query Panel | Open the Sysight panel (⌘Enter / Ctrl+Enter runs the query in it) |
| Sysight: New Session in an Editor Tab | A new session, in a tab of its own |
| Sysight: Run Query in Sysight | Run the selection, or the query file, in the panel |
| Sysight: Open Model Graph | The whole model as a graph |
| Sysight: Show in Graph | An element and its neighbors as a graph |
| Sysight: Show Model Server | The server's workspace, release, files, libraries and editors |
| Sysight: Restart Model Server | Restart the server with the current release and settings |
| Sysight: Show Output | The extension's log |

| Setting | |
|---|---|
| `sysight.path` | The `sysight` command to run (default: the one the extension carries) |
| `sysight.queriesFolder` | Where the panel saves queries, relative to the model's root (default `queries`) |
| `sysight.inlayHints.values` | Show computed values (default on) |
| `sysight.inlayHints.impliedSpecializations` | Show the library specializations the notation implies (default on) |
| `sysight.inlayHints.defaultVisibility` | Show `private` before imports without a visibility (default on) |
| `sysight.inlayHints.memberVisibility` | Show `public` before other members without a visibility (default off) |
| `sysight.inlayHints.effectiveNames` | Show the names unnamed elements take (default on) |

## With AI agents

Agents read, compute with and change the model through Sysight's MCP tools, on the same live
model as the editor.

VS Code's agents (such as Copilot in agent mode) get the tools from the extension, with nothing
to set up: *MCP: List Servers* shows `sysight`. In Claude Code, register it once per model:

```sh
claude mcp add sysight -- sysight mcp --workspace /path/to/model
```

Or add it to the model's `.mcp.json`, shared with everyone who opens the model:

```json
{ "mcpServers": { "sysight": { "command": "sysight", "args": ["mcp"] } } }
```

Other agents take the same command in their MCP configuration:
`{"command": "sysight", "args": ["mcp", "--workspace", "/path/to/model"]}`.

| Tool | Does |
|---|---|
| `describe` | An element as notation: its declaration and what it owns |
| `export` | A file as notation or RDF; without a file, the model's files |
| `check` | Diagnostics, optionally for some files |
| `query` | SPARQL (see [Queries](#queries)) |
| `evaluate`, `verify` | Values, and verdicts of constraints, requirements and `satisfy` |
| `update` | Change the model (see [Changing the model](#changing-the-model)) |
| `graph` | Show the user a graph in their editor (see [Graphs](#graphs)) |
| `manual` | This manual, by section |

The manual is also offered as MCP resources (`sysight://manual`, and one per section), for
clients that let you attach resources to a conversation.

Ask in plain words: "which requirements fail, and why?", "what is the total mass of the
vehicle?", "show me how the requirements trace to the parts". After changing the model, an
agent should `check` or `verify` what the change was meant to achieve.

## On the command line

The `sysight` command asks the model's server; `sysight help <command>` explains each command.

```sh
cd my-model
sysight check                                       # problems in the whole model
sysight evaluate 'Vehicle::mass' --unit kg          # a value
sysight verify 'Vehicle::massLimit'                 # a constraint or requirement: satisfied?
sysight query 'SELECT ?p WHERE { ?p a sysml:Part }' # anything, in SPARQL
sysight export --format ttl                         # the model as RDF
sysight manual queries                              # a section of this manual
```

| Command | Does |
|---|---|
| `check [files] [--syntax-only]` | Report problems. Exit 1 on errors. |
| `evaluate <target>... [--context <element>]... [--unit <unit>]` | Compute values (see [Values and verdicts](#values-and-verdicts)). |
| `verify <target>...` | Decide constraints, requirements and `satisfy` statements. |
| `query <sparql \| @file.rq \| -> [--format table\|csv\|tsv\|json\|xml\|ttl\|nt] [--no-entailment]` | Ask anything in SPARQL. |
| `export [files] [--format text\|trig\|nq\|ttl] [--element <name>] [--output <dir>]` | Write the model as notation or RDF (see [Text and RDF](#text-and-rdf)). |
| `update <operations.json \| -> [--dry-run] [--if-version <n>]` | Change the model (see [Changing the model](#changing-the-model)). |
| `import <file> [--format trig\|nq\|ttl\|kpar] [--into <dir>] [--force] [--graph <iri>]` | Add model files from RDF or a `.kpar` archive. |
| `start [--idle-minutes <n>] [--port <n>] [--foreground]` | Start the server, or keep the running one, until stopped. |
| `stop [--all]`, `restart`, `status`, `inspect` | Manage the servers (see [The model and its server](#the-model-and-its-server)). |
| `attach` | Connect an editor (see [Other editors](#other-editors)). |
| `mcp` | Serve the model to an agent (see [With AI agents](#with-ai-agents)). |
| `manual [section]` | This manual, or one of its sections. |
| `version`, `license [--third-party]` | The version; the license, or the third-party licenses. |

Every command takes `--workspace <dir>` (the model's folder), `--json` (print the server's
answer as JSON) and `--no-start` (fail instead of starting a server; also `SYSIGHT_NO_START=1`).

## Values and verdicts

Sysight computes attribute values, roll-ups and units, and decides constraints and
requirements.

- **Values:** `sysight evaluate <target>` takes an element (qualified name or IRI) or an
  expression. An expression is read in the last `--context` element; names it cannot see from
  there are looked up in the outer context elements, then as a unique function, unit or type
  anywhere. `--unit` gives the number in that unit, checked against its dimension:

  ```sh
  sysight evaluate 'Vehicle::mass' --unit kg
  sysight evaluate 'e1.thrust * 2' --context 'M::Rocket'
  ```

- **Verdicts:** `sysight verify <target>` decides a constraint, requirement or `satisfy`
  statement: satisfied (exit 0), violated (1) or undetermined (2), and why.
- **In queries:** `eval:value`, `eval:text`, `eval:unit`, `eval:verdict` and `eval:why` compute
  the same in SPARQL (see [Queries](#queries)).
- **As diagnostics:** `check`, the editor's Problems view and agents' `check` report asserted
  constraints and requirements that fail, or cannot be evaluated.

## Queries

Ask anything about the model in SPARQL 1.1, with entailment and functions that compute values
and verdicts.

```sparql
SELECT ?requirement (eval:verdict(?requirement) AS ?verdict) (eval:why(?requirement) AS ?why)
WHERE { ?requirement a sysml:Requirement }
```

- **Prefixes:** `kerml:`, `sysml:`, `eval:`, `rdf:`, `rdfs:`, `owl:` and `xsd:` need no
  declaring.
- **Classes** are named after the notation's keywords: `sysml:PartDef`, `sysml:Part`,
  `sysml:Attribute`, `sysml:Requirement`, `kerml:Feature`, ...
- **Relations** are named after the notation: `kerml:typedBy`, `kerml:specializes`,
  `kerml:subsets`, `kerml:redefines`, `kerml:ownedMember`, `kerml:ownedFeature`,
  `kerml:declaredName`, `kerml:value` (an expression's text), ...
- **Entailment:** queries see subclasses, sub-properties and inverses (`?x a kerml:Feature`
  finds parts too; `kerml:specializes+` follows typing, subsetting and redefinition).
  `--no-entailment` turns it off.
- **Graphs:** the default graph is the union of all files; `GRAPH ?g { … }` tells which file a
  fact comes from.

Relations Sysight infers, to keep queries short:

| Property | Holds when |
|---|---|
| `kerml:type` | a feature has a type: declared, or of what it subsets or redefines |
| `kerml:owns`, `kerml:within` | an element is declared inside another, at any depth |
| `kerml:conformsTo` | a type is another, at any depth, including the library types its kind and place imply (`kerml:impliedSpecializes`: a part is a `Parts::parts`) |
| `sysml:performs`, `sysml:exhibits`, `sysml:includesUseCase` | an element performs an action, exhibits a state, includes a use case |
| `sysml:satisfies`, `sysml:satisfiedBy` | `satisfy r by x` relates x and r |
| `sysml:verifies` | a verification case verifies a requirement |
| `kerml:dependsOn` | a dependency's client depends on its supplier |
| `kerml:connectedTo` | a connector (connection, interface, binding, flow, succession) joins two features |

The functions take what to evaluate (an element, a qualified name as text, or expression text),
then optionally context elements (outermost first) and a unit as text:

| Function | Gives |
|---|---|
| `eval:value` | the value: a number, Boolean or text |
| `eval:unit` | its unit |
| `eval:text` | the value as shown, or why it cannot be computed |
| `eval:verdict`, `eval:why` | a constraint's or requirement's verdict, and why |
| `eval:name(?x)`, `eval:qualifiedName(?x)` | an element's name (the one it takes, when unnamed) and qualified name |

```sparql
SELECT ?f (eval:text(?f) AS ?value) WHERE { ?a a sysml:Analysis ; kerml:ownedFeature ?f }
```

Save queries as `.rq` files in the model's `queries` folder, to commit with the model; the
editor's panel lists them, and `sysight query @queries/<name>.rq` runs one.

## Graphs

Draw the model's elements and relationships as a graph, in VS Code.

- *Sysight: Open Model Graph* draws the whole model, or a file or folder from the Explorer's
  context menu.
- *Sysight: Show in Graph*, in the editor's context menu, draws an element and its neighbors.
- *Open as graph* on any CONSTRUCT or DESCRIBE answer in the panel, or *Open as Graph* above
  such a query file, draws that answer.
- Agents show you a graph with the MCP tool `graph`, from a CONSTRUCT or DESCRIBE query.

Hover an element to bring out its neighbors, click it for what it is and how it relates,
double-click to open its declaration; search, expand an element's neighbors, and drag to pin.
Settings filter what shows (ownership, library elements, orphans, each relationship), color by
kind, package or file (with an optional legend), and set labels, arrows and forces.

*Save* keeps the view, layout included, as a query file whose first line holds its settings;
commit it with the model, and *Reload* reads it again:

```sparql
# graph: {"hide":["owns","library"],"color":"package","legend":true}
CONSTRUCT { ?part kerml:typedBy ?def } WHERE { ?part a sysml:Part ; kerml:typedBy ?def }
```

The settings: `hide` (a list of `"owns"`, `"library"`, `"orphans"` or relationship names),
`color` (`"kind"`, `"package"` or `"file"`), `legend` and `arrows` (true or false), `labels`
(`"always"`, `"zoom"` or `"never"`). Relationships worth drawing: `kerml:typedBy`,
`kerml:specializes`, `kerml:subsets`, `kerml:redefines`, `kerml:ownedMember`,
`sysml:satisfies`, `sysml:verifies`, `kerml:dependsOn`, `kerml:connectedTo`.

## Changing the model

Change the model in its own terms; the changes are edits to the files' text, so the files stay
the model.

`sysight update` (and the MCP tool `update`) applies a list of operations. Elements are named by
qualified name or IRI, properties as in queries. Only the text an operation is about changes;
everything else (notes, layout) stays as written.

```json
{ "operations": [
  { "op": "create",  "owner": "Vehicles::Vehicle", "text": "part battery : Battery[1];" },
  { "op": "create",  "owner": "parts/New.sysml", "text": "package New;" },
  { "op": "create",  "after": "Vehicles::Vehicle::engine", "text": "attribute color;" },
  { "op": "set",     "element": "Vehicles::Vehicle::mass", "property": "value", "value": "1450 [kg]" },
  { "op": "set",     "element": "Vehicles::Vehicle", "property": "specializes", "value": ["Base::Asset"] },
  { "op": "set",     "element": "Vehicles::Vehicle", "property": "doc", "value": "A road vehicle." },
  { "op": "rename",  "element": "Vehicles::Engine", "name": "PowerUnit" },
  { "op": "replace", "element": "Vehicles::Wheel", "text": "part def Wheel { attribute size; }" },
  { "op": "move",    "element": "Vehicles::Spare", "owner": "Vehicles::Storage" },
  { "op": "delete",  "element": "Vehicles::OldPart", "references": "fail" }
]}
```

- **create** adds members, written as text, at the end of an owner's body, next to a sibling
  (`after`, `before`), or to a file (a path in the model's folder; a new file is created).
- **set** changes one property and writes the element's declaration again, keeping its body:
  `value`, `multiplicity`, `typedBy`, `subsets`, `redefines`, `references`, `crosses`,
  `specializes` (names), `direction`, `visibility`, `shortName`, `doc`, and the flags
  (`isAbstract`, `isDerived`, `isConstant`, `isEnd`, `isOrdered`, `isUnique`, `isComposite`,
  ...); `null` removes it.
- **rename** changes the declared name and every reference that spells it, in every file.
- **replace** writes an element again from the text given.
- **move** moves an element to another owner, a sibling's side or a file, with the notes right
  above it, and keeps every reference reaching what it reached.
- **delete** removes an element, what it owns and the notes right above it. It refuses while
  something refers to it, unless `references` is `"leave"` (the references stay, and no longer
  resolve) or `"detach"` (the references go: `part p : Old;` becomes `part p;`, and a
  relationship that is nothing without the name, such as an import or a `satisfy`, is removed).

The operations apply in order, each seeing the ones before, and all or none: if one fails, or
leaves a file that does not parse, nothing changes. The answer has the model's new version,
each file's diff, the elements created, and the diagnostics the change added and removed.
`--dry-run` (`"dryRun": true`) shows it without changing anything; `--if-version <n>`
(`"ifVersion"`) refuses if the model changed since version `n`. A file open in an editor is
changed in the editor.

## Text and RDF

Write the model as canonical notation or as RDF, and add files from RDF or a `.kpar` archive.

- `sysight export --format text` writes each file in the notation's canonical style.
  `//` notes are not part of the model and are not written; `doc` and comments are.
- `sysight export --format ttl|trig|nq` writes the model as RDF, one named graph per file, in
  the vocabulary queries use. `--element <name>` exports one element and what it owns;
  `--output <dir>` writes one file per document.
- `sysight import <file>` adds model files: RDF is written back as notation, a `.kpar`
  archive's documents as they are. Files go under `--into` (default: the model's folder),
  keeping their layout; existing files are only overwritten with `--force`.

## Programs and notebooks

Programs and notebooks, in any language, use the model through an HTTP API: SPARQL queries to
read it, and the update operations to change it.

The server listens on `127.0.0.1`; its address is in the model's `.sysight/server.json`
(`sysight inspect` shows it), and `port` in `sysight.toml` (or `sysight start --port <n>`)
fixes it. Run `sysight start` first, so the server is there for the program.

**Queries.** `POST /query` (or `GET /query?query=…`) speaks the SPARQL 1.1 protocol: the query as
a form field (`query=…`) or as the body (`Content-Type: application/sparql-query`). The answer
comes in the format `Accept` asks for: `application/sparql-results+json`, `text/csv`,
`text/tab-separated-values` or `application/sparql-results+xml` (Turtle or N-Triples for
CONSTRUCT and DESCRIBE); `format=json|csv|tsv|xml|ttl|nt` overrides it. `entailment=none` turns
entailment off. A query that does not parse is answered with status 400 and where it fails.

Keep the queries a program uses as `.rq` files in the model's `queries` folder: they are its
API, versioned with the model, and the editor runs them too. Pass values to a query with a
`VALUES` clause after it, which is standard SPARQL, rather than by editing its text:

```sparql
# queries/mass.rq: the value of each attribute with the names given
SELECT ?attribute (eval:text(?attribute) AS ?value)
WHERE { ?attribute a sysml:Attribute ; kerml:declaredName ?name }
```

Python:

```python
import json, pathlib, requests

url = json.load(open(".sysight/server.json"))["url"]
query = pathlib.Path("queries/mass.rq").read_text() + ' VALUES ?name { "dryMass" "totalMass" }'
answer = requests.post(f"{url}/query", data={"query": query},
                       headers={"Accept": "application/sparql-results+json"}).json()
for row in answer["results"]["bindings"]:
    print(row["attribute"]["value"], row["value"]["value"])
```

With pandas, ask for CSV: `pd.read_csv(io.StringIO(requests.post(..., headers={"Accept":
"text/csv"}).text))`.

JavaScript (Node 18 or later):

```js
import fs from "node:fs";

const { url } = JSON.parse(fs.readFileSync(".sysight/server.json", "utf8"));
const res = await fetch(`${url}/query`, {
  method: "POST",
  headers: { "Content-Type": "application/sparql-query", Accept: "application/sparql-results+json" },
  body: "SELECT ?r (eval:verdict(?r) AS ?verdict) WHERE { ?r a sysml:Requirement }",
});
const { results } = await res.json();
```

Java (`java.net.http`):

```java
var request = HttpRequest.newBuilder(URI.create(url + "/query"))
    .header("Content-Type", "application/sparql-query")
    .header("Accept", "text/csv")
    .POST(HttpRequest.BodyPublishers.ofString("SELECT ?p WHERE { ?p a sysml:Part }"))
    .build();
String csv = HttpClient.newHttpClient().send(request, HttpResponse.BodyHandlers.ofString()).body();
```

The shell:

```sh
curl -s "$URL/query" -H 'Accept: text/csv' --data-urlencode 'query=SELECT ?p WHERE { ?p a sysml:Part }'
```

The prefixes `kerml:`, `sysml:` and `eval:` need no declaring here either (see
[Queries](#queries)).

**Updates.** `POST /update` takes the operations of [Changing the model](#changing-the-model)
as a JSON body, `{"operations": [...], "dryRun": false, "ifVersion": 42}`, and answers with
JSON: the new version, each file's diff, the elements created, and the diagnostics added and
removed. A refused update is answered with status 422 and the operation that failed; one whose
`ifVersion` is out of date with 409. Read the version from `GET /inspect` before the change, so
an update made while someone else changed the model is refused, not merged:

```python
version = requests.get(f"{url}/inspect").json()["version"]
change = {"ifVersion": version, "operations": [
    {"op": "set", "element": "Vehicle::mass", "property": "value", "value": "1450 [kg]"}]}
answer = requests.post(f"{url}/update", json=change)
answer.raise_for_status()
print(answer.json()["diagnostics"]["added"])
```

**The other endpoints** have the commands' names and parameters, and answer JSON: `GET /check`
(`file`, `syntax-only`), `GET /evaluate` (`target`, `context`, `unit`), `GET /verify`
(`target`), `GET /export` (`file`, `format`, `element`), `GET /inspect`, and `POST /import`.

## Settings

A `sysight.toml` at the model's root holds its settings, all optional.

```toml
libraries = ["../sysml.library"]   # .kpar archives, .bundle files, or folders of either
exclude = ["build"]                # paths that are not part of the model
port = 7878                        # a fixed HTTP port (default: any free one)
idle-minutes = 15                  # a server started on demand stops after this long with
                                   # no requests and no editor (0: never)

[format]                           # formatting, for everyone on the project
indent = 4                         # spaces per level, or "tab" (default: the editor's)
break-conditions = 3               # and/or chains this long break one condition per line (0: never)
```

Without `libraries`, the standard KerML and SysML libraries are used; `libraries = []` means
none. `SYSIGHT_LIBRARIES` (paths separated by `:`) adds libraries without a settings file. In a
terminal, each command ends with a line naming Sysight; `SYSIGHT_NO_BANNER=1` turns it off.

## Other editors

Any editor with LSP support can use Sysight: configure it to run `sysight attach`.

`sysight attach` connects to the model's server, starting it if needed, and speaks LSP on
stdin/stdout. Several editors can attach at once; each sees the others' unsaved edits. For
example, Neovim:

```lua
vim.lsp.config('sysight', { cmd = { 'sysight', 'attach' }, filetypes = { 'sysml', 'kerml' }, root_markers = { 'sysight.toml', '.git' } })
vim.lsp.enable('sysight')
```

## In CI

Check the model, and decide its requirements, on every change.

Install Sysight in a step (see the [README](README.md)), then run the commands; their exit codes
fail the build:

```yaml
- run: sysight check                          # exit 1 on errors
- run: sysight verify 'Vehicle::massLimit'    # exit 1 if violated, 2 if undetermined
```

The first command starts the model's server, and the later steps use it.
