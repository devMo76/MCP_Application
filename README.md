# MCP Application

MCP Application is an interactive command-line assistant that connects Anthropic's Claude models to local and external [Model Context Protocol (MCP)](https://modelcontextprotocol.io/) servers. It demonstrates how an AI chat client can discover tools, read resources, invoke reusable prompts, and execute tool calls across one or more MCP connections.

The included document server uses an in-memory collection of sample documents. From the terminal, users can mention those documents with `@`, ask Claude questions about their contents, and expose document editing operations to the model as MCP tools.

## Project goals

- Provide a small, understandable reference implementation of an MCP client and server in Python.
- Connect Claude to tools and resources exposed by MCP servers over standard input/output.
- Support document-aware conversations through `@document-name` mentions.
- Offer an extensible CLI that can connect to additional MCP server scripts.
- Demonstrate the full tool-use loop between a user, Claude, and MCP tools.

## Features

- Interactive terminal chat with history, completion, and command suggestions.
- Anthropic Claude integration with multi-turn conversation state.
- MCP server discovery and execution of tools from multiple connected servers.
- MCP document resources for listing and reading sample documents.
- Document tools for reading and replacing text.
- A reusable MCP prompt for formatting documents as Markdown.
- Optional startup of additional MCP server scripts passed on the command line.

## Architecture

```text
User terminal
    |
    v
CliApp / CliChat
    |
    +----> Anthropic Claude API
    |          |
    |          +---- tool-use requests
    |
    +----> MCPClient ---- stdio ----> DocumentMCP server
    |                               - resources
    |                               - tools
    |                               - prompts
    |
    +----> optional additional MCP servers
```

Key files:

| File | Purpose |
| --- | --- |
| `main.py` | Loads configuration, starts MCP clients, and launches the CLI. |
| `mcp_server.py` | Defines the sample document MCP server, resources, tools, and prompt. |
| `mcp_client.py` | Manages an MCP stdio session and exposes client operations. |
| `core/cli.py` | Implements the interactive prompt, completions, and suggestions. |
| `core/cli_chat.py` | Resolves document mentions and slash commands. |
| `core/chat.py` | Runs the Claude response and MCP tool-use loop. |
| `core/tools.py` | Converts MCP tools for Claude and routes tool calls to the correct server. |
| `core/claude.py` | Wraps the Anthropic Messages API. |

## Requirements

- Python 3.10 or newer
- An Anthropic API key
- An Anthropic model ID available to your account
- [`uv`](https://docs.astral.sh/uv/) (recommended), or `pip`

## Setup

### 1. Clone the repository

```bash
git clone https://github.com/devMo76/MCP_Application.git
cd MCP_Application
```

### 2. Configure the environment

Create a `.env` file in the project root:

```dotenv
ANTHROPIC_API_KEY=your_anthropic_api_key
CLAUDE_MODEL=your_model_id
USE_UV=1
```

`USE_UV=1` starts the bundled MCP server through `uv`. Set it to `0` if you want the application to launch the server with the active Python interpreter instead. The `.env` file is ignored by Git and must never be committed.

### 3. Install dependencies

With `uv`:

```bash
uv sync
```

Or with a standard virtual environment:

```bash
python -m venv .venv
```

Activate it on Windows:

```powershell
.venv\Scripts\Activate.ps1
```

Or on macOS/Linux:

```bash
source .venv/bin/activate
```

Then install the project dependencies:

```bash
pip install -e .
```

When using this option, set `USE_UV=0` in `.env`.

## Running the application

With `uv`:

```bash
uv run main.py
```

With the active Python environment:

```bash
python main.py
```

To connect one or more additional Python MCP servers, append their script paths:

```bash
uv run main.py path/to/server_one.py path/to/server_two.py
```

The application communicates with MCP servers over stdio, so server output intended for diagnostics should be written to standard error rather than standard output.

## Usage

### Chat with Claude

Enter a normal message at the prompt:

```text
> Explain what this application can do.
```

Press `Ctrl+C` to exit.

### Reference a document

Type `@` to see document completions, then mention a document in the question:

```text
> What condition is described in @report.pdf?
```

The client reads the selected MCP resource and supplies its content to Claude as context.

### Use MCP tools

Claude can call the server's `read_doc_contents` and `edit_document` tools when they are relevant to a request. Tool requests are routed to the MCP server that advertises the requested tool, and the results are returned to Claude before it produces the final response.

## Included MCP capabilities

The sample `DocumentMCP` server exposes:

| Type | Name or URI | Description |
| --- | --- | --- |
| Tool | `read_doc_contents` | Returns the contents of a document by ID. |
| Tool | `edit_document` | Replaces matching text in an in-memory document. |
| Resource | `docs://documents` | Lists all available document IDs. |
| Resource | `docs://documents/{doc_id}` | Returns one document's contents. |
| Prompt | `format` | Guides Claude to format a document as Markdown. |

The bundled documents are demonstration data stored in memory. Edits last only for the lifetime of the server process and do not modify files on disk.

## Current status

This repository is a work in progress. Tool discovery, resource reading, document mentions, and the core Claude tool-use loop are implemented. MCP prompt discovery and retrieval in `mcp_client.py` are currently placeholders, so slash-command prompt execution will not work until `list_prompts()` and `get_prompt()` are completed. A document summarization prompt is also noted as a future enhancement in `mcp_server.py`.

## Development

To add sample documents, update the `docs` dictionary in `mcp_server.py`. To add MCP functionality, decorate new functions with the relevant `FastMCP` tool, resource, or prompt decorator; connected clients discover tools automatically.

A quick syntax check can be run with:

```bash
python -m compileall main.py mcp_client.py mcp_server.py core
```

There is not yet an automated test, lint, or type-check configuration. Contributions that add tests or finish prompt support are welcome.

## Security notes

- Keep `ANTHROPIC_API_KEY` only in `.env` or another secure secret store.
- Review third-party MCP servers before connecting them; their tools can execute actions available to their process.
- Treat document and tool output as untrusted input when extending this project.

## License

No license has been added yet. Until one is provided, the repository's contents remain under the copyright holder's default rights.
