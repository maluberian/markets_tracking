# codex CLI SDK

> Learn about programmatically integrating codex CLI into your applications with the codex CLI SDK.

The codex CLI SDK enables running codex CLI as a subprocess, providing a way to build AI-powered coding assistants and tools that leverage codex's capabilities.

The SDK is available for command line, TypeScript, and Python usage.

## Authentication

The codex CLI SDK supports multiple authentication methods:

### Anthropic API key

To use the codex CLI SDK directly with Anthropic's API, we recommend creating a dedicated API key:

1. Create an Anthropic API key in the [Anthropic Console](https://console.anthropic.com/)
2. Then, set the `ANTHROPIC_API_KEY` environment variable. We recommend storing this key securely (e.g., using a Github [secret](https://docs.github.com/en/actions/security-for-github-actions/security-guides/using-secrets-in-github-actions))

### Third-Party API credentials

The SDK also supports third-party API providers:

- **Amazon Bedrock**: Set `CLAUDE_CODE_USE_BEDROCK=1` environment variable and configure AWS credentials
- **Google Vertex AI**: Set `CLAUDE_CODE_USE_VERTEX=1` environment variable and configure Google Cloud credentials

For detailed configuration instructions for third-party providers, see the [Amazon Bedrock](/en/docs/codex-code/amazon-bedrock) and [Google Vertex AI](/en/docs/codex-code/google-vertex-ai) documentation.

## Basic SDK usage

The codex CLI SDK allows you to use codex CLI in non-interactive mode from your applications.

### Command line

Here are a few basic examples for the command line SDK:

```bash
# Run a single prompt and exit (print mode)
$ codex -p "Write a function to calculate Fibonacci numbers"

# Using a pipe to provide stdin
$ echo "Explain this code" | codex -p

# Output in JSON format with metadata
$ codex -p "Generate a hello world function" --output-format json

# Stream JSON output as it arrives
$ codex -p "Build a React component" --output-format stream-json
```

### TypeScript

The TypeScript SDK is included in the main [`@anthropic-ai/codex-code`](https://www.npmjs.com/package/@anthropic-ai/codex-code) package on NPM:

```ts
import { query, type SDKMessage } from "@anthropic-ai/codex-code";

const messages: SDKMessage[] = [];

for await (const message of query({
  prompt: "Write a haiku about foo.py",
  abortController: new AbortController(),
  options: {
    maxTurns: 3,
  },
})) {
  messages.push(message);
}

console.log(messages);
```

The TypeScript SDK accepts all arguments supported by the command line SDK, as well as:

| Argument                     | Description                         | Default                                                       |
| :--------------------------- | :---------------------------------- | :------------------------------------------------------------ |
| `abortController`            | Abort controller                    | `new AbortController()`                                       |
| `cwd`                        | Current working directory           | `process.cwd()`                                               |
| `executable`                 | Which JavaScript runtime to use     | `node` when running with Node.js, `bun` when running with Bun |
| `executableArgs`             | Arguments to pass to the executable | `[]`                                                          |
| `pathTocodexCodeExecutable` | Path to the codex CLI executable  | Executable that ships with `@anthropic-ai/codex-code`        |

### Python

The Python SDK is available as [`codex-code-sdk`](https://github.com/anthropics/codex-code-sdk-python) on PyPI:

```bash
pip install codex-code-sdk
```

**Prerequisites:**

- Python 3.10+
- Node.js
- codex CLI CLI: `npm install -g @anthropic-ai/codex-code`

Basic usage:

```python
import anyio
from codex_code_sdk import query, codexCodeOptions, Message

async def main():
    messages: list[Message] = []

    async for message in query(
        prompt="Write a haiku about foo.py",
        options=codexCodeOptions(max_turns=3)
    ):
        messages.append(message)

    print(messages)

anyio.run(main)
```

The Python SDK accepts all arguments supported by the command line SDK through the `codexCodeOptions` class:

```python
from codex_code_sdk import query, codexCodeOptions
from pathlib import Path

options = codexCodeOptions(
    max_turns=3,
    system_prompt="You are a helpful assistant",
    cwd=Path("/path/to/project"),  # Can be string or Path
    allowed_tools=["Read", "Write", "Bash"],
    permission_mode="acceptEdits"
)

async for message in query(prompt="Hello", options=options):
    print(message)
```

## Advanced usage

The documentation below uses the command line SDK as an example, but can also be used with the TypeScript and Python SDKs.

### Multi-turn conversations

For multi-turn conversations, you can resume conversations or continue from the most recent session:

```bash
# Continue the most recent conversation
$ codex --continue

# Continue and provide a new prompt
$ codex --continue "Now refactor this for better performance"

# Resume a specific conversation by session ID
$ codex --resume 550e8400-e29b-41d4-a716-446655440000

# Resume in print mode (non-interactive)
$ codex -p --resume 550e8400-e29b-41d4-a716-446655440000 "Update the tests"

# Continue in print mode (non-interactive)
$ codex -p --continue "Add error handling"
```

### Custom system prompts

You can provide custom system prompts to guide codex's behavior:

```bash
# Override system prompt (only works with --print)
$ codex -p "Build a REST API" --system-prompt "You are a senior backend engineer. Focus on security, performance, and maintainability."

# System prompt with specific requirements
$ codex -p "Create a database schema" --system-prompt "You are a database architect. Use PostgreSQL best practices and include proper indexing."
```

You can also append instructions to the default system prompt:

```bash
# Append system prompt (only works with --print)
$ codex -p "Build a REST API" --append-system-prompt "After writing code, be sure to code review yourself."
```

### MCP Configuration

The Model Context Protocol (MCP) allows you to extend codex CLI with additional tools and resources from external servers. Using the `--mcp-config` flag, you can load MCP servers that provide specialized capabilities like database access, API integrations, or custom tooling.

Create a JSON configuration file with your MCP servers:

```json
{
  "mcpServers": {
    "filesystem": {
      "command": "npx",
      "args": [
        "-y",
        "@modelcontextprotocol/server-filesystem",
        "/path/to/allowed/files"
      ]
    },
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_TOKEN": "your-github-token"
      }
    }
  }
}
```

Then use it with codex CLI:

```bash
# Load MCP servers from configuration
$ codex -p "List all files in the project" --mcp-config mcp-servers.json

# Important: MCP tools must be explicitly allowed using --allowedTools
# MCP tools follow the format: mcp__$serverName__$toolName
$ codex -p "Search for TODO comments" \
  --mcp-config mcp-servers.json \
  --allowedTools "mcp__filesystem__read_file,mcp__filesystem__list_directory"

# Use an MCP tool for handling permission prompts in non-interactive mode
$ codex -p "Deploy the application" \
  --mcp-config mcp-servers.json \
  --allowedTools "mcp__permissions__approve" \
  --permission-prompt-tool mcp__permissions__approve
```

<Note>
  When using MCP tools, you must explicitly allow them using the `--allowedTools` flag. MCP tool names follow the pattern `mcp__<serverName>__<toolName>` where:

- `serverName` is the key from your MCP configuration file
- `toolName` is the specific tool provided by that server

This security measure ensures that MCP tools are only used when explicitly permitted.

If you specify just the server name (i.e., `mcp__<serverName>`), all tools from that server will be allowed.

Glob patterns (e.g., `mcp__go*`) are not supported.
</Note>

### Custom permission prompt tool

Optionally, use `--permission-prompt-tool` to pass in an MCP tool that we will use to check whether or not the user grants the model permissions to invoke a given tool. When the model invokes a tool the following happens:

1. We first check permission settings: all [settings.json files](/en/docs/codex-code/settings), as well as `--allowedTools` and `--disallowedTools` passed into the SDK; if one of these allows or denies the tool call, we proceed with the tool call
2. Otherwise, we invoke the MCP tool you provided in `--permission-prompt-tool`

The `--permission-prompt-tool` MCP tool is passed the tool name and input, and must return a JSON-stringified payload with the result. The payload must be one of:

```ts
// tool call is allowed
{
  "behavior": "allow",
  "updatedInput": {...}, // updated input, or just return back the original input
}

// tool call is denied
{
  "behavior": "deny",
  "message": "..." // human-readable string explaining why the permission was denied
}
```

For example, a TypeScript MCP permission prompt tool implementation might look like this:

```ts
const server = new McpServer({
  name: "Test permission prompt MCP Server",
  version: "0.0.1",
});

server.tool(
  "approval_prompt",
  'Simulate a permission check - approve if the input contains "allow", otherwise deny',
  {
    tool_name: z
      .string()
      .describe("The name of the tool requesting permission"),
    input: z.object({}).passthrough().describe("The input for the tool"),
    tool_use_id: z
      .string()
      .optional()
      .describe("The unique tool use request ID"),
  },
  async ({ tool_name, input }) => {
    return {
      content: [
        {
          type: "text",
          text: JSON.stringify(
            JSON.stringify(input).includes("allow")
              ? {
                  behavior: "allow",
                  updatedInput: input,
                }
              : {
                  behavior: "deny",
                  message: "Permission denied by test approval_prompt tool",
                },
          ),
        },
      ],
    };
  },
);
```

To use this tool, add your MCP server (eg. with `--mcp-config`), then invoke the SDK like so:

```sh
codex -p "..." \
  --permission-prompt-tool mcp__test-server__approval_prompt \
  --mcp-config my-config.json
```

Usage notes:

- Use `updatedInput` to tell the model that the permission prompt mutated its input; otherwise, set `updatedInput` to the original input, as in the example above. For example, if the tool shows a file edit diff to the user and lets them edit the diff manually, the permission prompt tool should return that updated edit.
- The payload must be JSON-stringified

## Available CLI options

The SDK leverages all the CLI options available in codex CLI. Here are the key ones for SDK usage:

| Flag                       | Description                                                                                            | Example                                                                                                                   |
| :------------------------- | :----------------------------------------------------------------------------------------------------- | :------------------------------------------------------------------------------------------------------------------------ |
| `--print`, `-p`            | Run in non-interactive mode                                                                            | `codex -p "query"`                                                                                                       |
| `--output-format`          | Specify output format (`text`, `json`, `stream-json`)                                                  | `codex -p --output-format json`                                                                                          |
| `--resume`, `-r`           | Resume a conversation by session ID                                                                    | `codex --resume abc123`                                                                                                  |
| `--continue`, `-c`         | Continue the most recent conversation                                                                  | `codex --continue`                                                                                                       |
| `--verbose`                | Enable verbose logging                                                                                 | `codex --verbose`                                                                                                        |
| `--max-turns`              | Limit agentic turns in non-interactive mode                                                            | `codex --max-turns 3`                                                                                                    |
| `--system-prompt`          | Override system prompt (only with `--print`)                                                           | `codex --system-prompt "Custom instruction"`                                                                             |
| `--append-system-prompt`   | Append to system prompt (only with `--print`)                                                          | `codex --append-system-prompt "Custom instruction"`                                                                      |
| `--allowedTools`           | Space-separated list of allowed tools, or <br /><br /> string of comma-separated list of allowed tools | `codex --allowedTools mcp__slack mcp__filesystem`<br /><br />`codex --allowedTools "Bash(npm install),mcp__filesystem"` |
| `--disallowedTools`        | Space-separated list of denied tools, or <br /><br /> string of comma-separated list of denied tools   | `codex --disallowedTools mcp__splunk mcp__github`<br /><br />`codex --disallowedTools "Bash(git commit),mcp__github"`   |
| `--mcp-config`             | Load MCP servers from a JSON file                                                                      | `codex --mcp-config servers.json`                                                                                        |
| `--permission-prompt-tool` | MCP tool for handling permission prompts (only with `--print`)                                         | `codex --permission-prompt-tool mcp__auth__prompt`                                                                       |

For a complete list of CLI options and features, see the [CLI reference](/en/docs/codex-code/cli-reference) documentation.

## Output formats

The SDK supports multiple output formats:

### Text output (default)

Returns just the response text:

```bash
$ codex -p "Explain file src/components/Header.tsx"
# Output: This is a React component showing...
```

### JSON output

Returns structured data including metadata:

```bash
$ codex -p "How does the data layer work?" --output-format json
```

Response format:

```json
{
  "type": "result",
  "subtype": "success",
  "total_cost_usd": 0.003,
  "is_error": false,
  "duration_ms": 1234,
  "duration_api_ms": 800,
  "num_turns": 6,
  "result": "The response text here...",
  "session_id": "abc123"
}
```

### Streaming JSON output

Streams each message as it is received:

```bash
$ codex -p "Build an application" --output-format stream-json
```

Each conversation begins with an initial `init` system message, followed by a list of user and assistant messages, followed by a final `result` system message with stats. Each message is emitted as a separate JSON object.

## Message schema

Messages returned from the JSON API are strictly typed according to the following schema:

```ts
type SDKMessage =
  // An assistant message
  | {
      type: "assistant";
      message: Message; // from Anthropic SDK
      session_id: string;
    }

  // A user message
  | {
      type: "user";
      message: MessageParam; // from Anthropic SDK
      session_id: string;
    }

  // Emitted as the last message
  | {
      type: "result";
      subtype: "success";
      duration_ms: float;
      duration_api_ms: float;
      is_error: boolean;
      num_turns: int;
      result: string;
      session_id: string;
      total_cost_usd: float;
    }

  // Emitted as the last message, when we've reached the maximum number of turns
  | {
      type: "result";
      subtype: "error_max_turns" | "error_during_execution";
      duration_ms: float;
      duration_api_ms: float;
      is_error: boolean;
      num_turns: int;
      session_id: string;
      total_cost_usd: float;
    }

  // Emitted as the first message at the start of a conversation
  | {
      type: "system";
      subtype: "init";
      apiKeySource: string;
      cwd: string;
      session_id: string;
      tools: string[];
      mcp_servers: {
        name: string;
        status: string;
      }[];
      model: string;
      permissionMode: "default" | "acceptEdits" | "bypassPermissions" | "plan";
    };
```

We will soon publish these types in a JSONSchema-compatible format. We use semantic versioning for the main codex CLI package to communicate breaking changes to this format.

`Message` and `MessageParam` types are available in Anthropic SDKs. For example, see the Anthropic [TypeScript](https://github.com/anthropics/anthropic-sdk-typescript) and [Python](https://github.com/anthropics/anthropic-sdk-python/) SDKs.

## Input formats

The SDK supports multiple input formats:

### Text input (default)

Input text can be provided as an argument:

```bash
$ codex -p "Explain this code"
```

Or input text can be piped via stdin:

```bash
$ echo "Explain this code" | codex -p
```

### Streaming JSON input

A stream of messages provided via `stdin` where each message represents a user turn. This allows multiple turns of a conversation without re-launching the `codex` binary and allows providing guidance to the model while it is processing a request.

Each message is a JSON 'User message' object, following the same format as the output message schema. Messages are formatted using the [jsonl](https://jsonlines.org/) format where each line of input is a complete JSON object. Streaming JSON input requires `-p` and `--output-format stream-json`.

Currently this is limited to text-only user messages.

```bash
$ echo '{"type":"user","message":{"role":"user","content":[{"type":"text","text":"Explain this code"}]}}' | codex -p --output-format=stream-json --input-format=stream-json --verbose
```

## Examples

### Simple script integration

```bash
#!/bin/bash

# Simple function to run codex and check exit code
run_codex() {
    local prompt="$1"
    local output_format="${2:-text}"

    if codex -p "$prompt" --output-format "$output_format"; then
        echo "Success!"
    else
        echo "Error: codex failed with exit code $?" >&2
        return 1
    fi
}

# Usage examples
run_codex "Write a Python function to read CSV files"
run_codex "Optimize this database query" "json"
```

### Processing files with codex

```bash
# Process a file through codex
$ cat mycode.py | codex -p "Review this code for bugs"

# Process multiple files
$ for file in *.js; do
    echo "Processing $file..."
    codex -p "Add JSDoc comments to this file:" < "$file" > "${file}.documented"
done

# Use codex in a pipeline
$ grep -l "TODO" *.py | while read file; do
    codex -p "Fix all TODO items in this file" < "$file"
done
```

### Session management

```bash
# Start a session and capture the session ID
$ codex -p "Initialize a new project" --output-format json | jq -r '.session_id' > session.txt

# Continue with the same session
$ codex -p --resume "$(cat session.txt)" "Add unit tests"
```

## Best practices

1. **Use JSON output format** for programmatic parsing of responses:

   ```bash
   # Parse JSON response with jq
   result=$(codex -p "Generate code" --output-format json)
   code=$(echo "$result" | jq -r '.result')
   cost=$(echo "$result" | jq -r '.cost_usd')
   ```

2. **Handle errors gracefully** - check exit codes and stderr:

   ```bash
   if ! codex -p "$prompt" 2>error.log; then
       echo "Error occurred:" >&2
       cat error.log >&2
       exit 1
   fi
   ```

3. **Use session management** for maintaining context in multi-turn conversations

4. **Consider timeouts** for long-running operations:

   ```bash
   timeout 300 codex -p "$complex_prompt" || echo "Timed out after 5 minutes"
   ```

5. **Respect rate limits** when making multiple requests by adding delays between calls

## Real-world applications

The codex CLI SDK enables powerful integrations with your development workflow. One notable example is the [codex CLI GitHub Actions](/en/docs/codex-code/github-actions), which uses the SDK to provide automated code review, PR creation, and issue triage capabilities directly in your GitHub workflow.

## Related resources

- [CLI usage and controls](/en/docs/codex-code/cli-reference) - Complete CLI documentation
- [GitHub Actions integration](/en/docs/codex-code/github-actions) - Automate your GitHub workflow with codex
- [Common workflows](/en/docs/codex-code/common-workflows) - Step-by-step guides for common use cases
