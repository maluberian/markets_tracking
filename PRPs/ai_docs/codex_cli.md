# CLI reference

> Complete reference for codex CLI command-line interface, including commands and flags.

## CLI commands

| Command                            | Description                                    | Example                                                            |
| :--------------------------------- | :--------------------------------------------- | :----------------------------------------------------------------- |
| `codex`                           | Start interactive REPL                         | `codex`                                                           |
| `codex "query"`                   | Start REPL with initial prompt                 | `codex "explain this project"`                                    |
| `codex -p "query"`                | Query via SDK, then exit                       | `codex -p "explain this function"`                                |
| `cat file \| codex -p "query"`    | Process piped content                          | `cat logs.txt \| codex -p "explain"`                              |
| `codex -c`                        | Continue most recent conversation              | `codex -c`                                                        |
| `codex -c -p "query"`             | Continue via SDK                               | `codex -c -p "Check for type errors"`                             |
| `codex -r "<session-id>" "query"` | Resume session by ID                           | `codex -r "abc123" "Finish this PR"`                              |
| `codex update`                    | Update to latest version                       | `codex update`                                                    |
| `codex mcp`                       | Configure Model Context Protocol (MCP) servers | See the [codex CLI MCP documentation](/en/docs/codex-code/mcp). |

## CLI flags

Customize codex CLI's behavior with these command-line flags:

| Flag                             | Description                                                                                                                                              | Example                                                     |
| :------------------------------- | :------------------------------------------------------------------------------------------------------------------------------------------------------- | :---------------------------------------------------------- |
| `--add-dir`                      | Add additional working directories for codex to access (validates each path exists as a directory)                                                      | `codex --add-dir ../apps ../lib`                           |
| `--allowedTools`                 | A list of tools that should be allowed without prompting the user for permission, in addition to [settings.json files](/en/docs/codex-code/settings)    | `"Bash(git log:*)" "Bash(git diff:*)" "Read"`               |
| `--disallowedTools`              | A list of tools that should be disallowed without prompting the user for permission, in addition to [settings.json files](/en/docs/codex-code/settings) | `"Bash(git log:*)" "Bash(git diff:*)" "Edit"`               |
| `--print`, `-p`                  | Print response without interactive mode (see [SDK documentation](/en/docs/codex-code/sdk) for programmatic usage details)                               | `codex -p "query"`                                         |
| `--output-format`                | Specify output format for print mode (options: `text`, `json`, `stream-json`)                                                                            | `codex -p "query" --output-format json`                    |
| `--input-format`                 | Specify input format for print mode (options: `text`, `stream-json`)                                                                                     | `codex -p --output-format json --input-format stream-json` |
| `--verbose`                      | Enable verbose logging, shows full turn-by-turn output (helpful for debugging in both print and interactive modes)                                       | `codex --verbose`                                          |
| `--max-turns`                    | Limit the number of agentic turns in non-interactive mode                                                                                                | `codex -p --max-turns 3 "query"`                           |
| `--model`                        | Sets the model for the current session with an alias for the latest model (`sonnet` or `opus`) or a model's full name                                    | `codex --model codex-sonnet-4-20250514`                   |
| `--permission-mode`              | Begin in a specified [permission mode](iam#permission-modes)                                                                                             | `codex --permission-mode plan`                             |
| `--permission-prompt-tool`       | Specify an MCP tool to handle permission prompts in non-interactive mode                                                                                 | `codex -p --permission-prompt-tool mcp_auth_tool "query"`  |
| `--resume`                       | Resume a specific session by ID, or by choosing in interactive mode                                                                                      | `codex --resume abc123 "query"`                            |
| `--continue`                     | Load the most recent conversation in the current directory                                                                                               | `codex --continue`                                         |
| `--dangerously-skip-permissions` | Skip permission prompts (use with caution)                                                                                                               | `codex --dangerously-skip-permissions`                     |

<Tip>
  The `--output-format json` flag is particularly useful for scripting and
  automation, allowing you to parse codex's responses programmatically.
</Tip>

For detailed information about print mode (`-p`) including output formats,
streaming, verbose logging, and programmatic usage, see the
[SDK documentation](/en/docs/codex-code/sdk).

## See also

- [Interactive mode](/en/docs/codex-code/interactive-mode) - Shortcuts, input modes, and interactive features
- [Slash commands](/en/docs/codex-code/slash-commands) - Interactive session commands
- [Quickstart guide](/en/docs/codex-code/quickstart) - Getting started with codex CLI
- [Common workflows](/en/docs/codex-code/common-workflows) - Advanced workflows and patterns
- [Settings](/en/docs/codex-code/settings) - Configuration options
- [SDK documentation](/en/docs/codex-code/sdk) - Programmatic usage and integrations

# Interactive mode

> Complete reference for keyboard shortcuts, input modes, and interactive features in codex CLI sessions.

## Keyboard shortcuts

### General controls

| Shortcut         | Description                        | Context                    |
| :--------------- | :--------------------------------- | :------------------------- |
| `Ctrl+C`         | Cancel current input or generation | Standard interrupt         |
| `Ctrl+D`         | Exit codex CLI session           | EOF signal                 |
| `Ctrl+L`         | Clear terminal screen              | Keeps conversation history |
| `Up/Down arrows` | Navigate command history           | Recall previous inputs     |
| `Esc` + `Esc`    | Edit previous message              | Double-escape to modify    |

### Multiline input

| Method         | Shortcut       | Context                 |
| :------------- | :------------- | :---------------------- |
| Quick escape   | `\` + `Enter`  | Works in all terminals  |
| macOS default  | `Option+Enter` | Default on macOS        |
| Terminal setup | `Shift+Enter`  | After `/terminal-setup` |
| Paste mode     | Paste directly | For code blocks, logs   |

### Quick commands

| Shortcut     | Description                        | Notes                                                     |
| :----------- | :--------------------------------- | :-------------------------------------------------------- |
| `#` at start | Memory shortcut - add to CLAUDE.md | Prompts for file selection                                |
| `/` at start | Slash command                      | See [slash commands](/en/docs/codex-code/slash-commands) |

## Vim mode

Enable vim-style editing with `/vim` command or configure permanently via `/config`.

### Mode switching

| Command | Action                      | From mode |
| :------ | :-------------------------- | :-------- |
| `Esc`   | Enter NORMAL mode           | INSERT    |
| `i`     | Insert before cursor        | NORMAL    |
| `I`     | Insert at beginning of line | NORMAL    |
| `a`     | Insert after cursor         | NORMAL    |
| `A`     | Insert at end of line       | NORMAL    |
| `o`     | Open line below             | NORMAL    |
| `O`     | Open line above             | NORMAL    |

### Navigation (NORMAL mode)

| Command         | Action                    |
| :-------------- | :------------------------ |
| `h`/`j`/`k`/`l` | Move left/down/up/right   |
| `w`             | Next word                 |
| `e`             | End of word               |
| `b`             | Previous word             |
| `0`             | Beginning of line         |
| `$`             | End of line               |
| `^`             | First non-blank character |
| `gg`            | Beginning of input        |
| `G`             | End of input              |

### Editing (NORMAL mode)

| Command        | Action                  |
| :------------- | :---------------------- |
| `x`            | Delete character        |
| `dd`           | Delete line             |
| `D`            | Delete to end of line   |
| `dw`/`de`/`db` | Delete word/to end/back |
| `cc`           | Change line             |
| `C`            | Change to end of line   |
| `cw`/`ce`/`cb` | Change word/to end/back |
| `.`            | Repeat last change      |

<Tip>
  Configure your preferred line break behavior in terminal settings. Run `/terminal-setup` to install Shift+Enter binding for iTerm2 and VS Code terminals.
</Tip>

## Command history

codex CLI maintains command history for the current session:

- History is stored per working directory
- Cleared with `/clear` command
- Use Up/Down arrows to navigate (see keyboard shortcuts above)
- **Ctrl+R**: Reverse search through history (if supported by terminal)
- **Note**: History expansion (`!`) is disabled by default

## See also

- [Slash commands](/en/docs/codex-code/slash-commands) - Interactive session commands
- [CLI reference](/en/docs/codex-code/cli-reference) - Command-line flags and options
- [Settings](/en/docs/codex-code/settings) - Configuration options
- [Memory management](/en/docs/codex-code/memory) - Managing CLAUDE.md files
