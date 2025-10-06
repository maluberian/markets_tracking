# codex CLI settings

> Configure codex CLI with global and project-level settings, and environment variables.

codex CLI offers a variety of settings to configure its behavior to meet your needs. You can configure codex CLI by running the `/config` command when using the interactive REPL.

## Settings files

The `settings.json` file is our official mechanism for configuring codex
Code through hierarchical settings:

- **User settings** are defined in `~/.codex/settings.json` and apply to all
  projects.
- **Project settings** are saved in your project directory:
  - `.codex/settings.json` for settings that are checked into source control and shared with your team
  - `.codex/settings.local.json` for settings that are not checked in, useful for personal preferences and experimentation. codex CLI will configure git to ignore `.codex/settings.local.json` when it is created.
- For enterprise deployments of codex CLI, we also support **enterprise
  managed policy settings**. These take precedence over user and project
  settings. System administrators can deploy policies to:
  - macOS: `/Library/Application Support/codexCode/managed-settings.json`
  - Linux and WSL: `/etc/codex-code/managed-settings.json`
  - Windows: `C:\ProgramData\codexCode\managed-settings.json`

```JSON Example settings.json
{
  "permissions": {
    "allow": [
      "Bash(npm run lint)",
      "Bash(npm run test:*)",
      "Read(~/.zshrc)"
    ],
    "deny": [
      "Bash(curl:*)"
    ]
  },
  "env": {
    "CLAUDE_CODE_ENABLE_TELEMETRY": "1",
    "OTEL_METRICS_EXPORTER": "otlp"
  }
}
```

### Available settings

`settings.json` supports a number of options:

| Key                          | Description                                                                                                                                                           | Example                                                 |
| :--------------------------- | :-------------------------------------------------------------------------------------------------------------------------------------------------------------------- | :------------------------------------------------------ |
| `apiKeyHelper`               | Custom script, to be executed in `/bin/sh`, to generate an auth value. This value will be sent as `X-Api-Key` and `Authorization: Bearer` headers for model requests  | `/bin/generate_temp_api_key.sh`                         |
| `cleanupPeriodDays`          | How long to locally retain chat transcripts (default: 30 days)                                                                                                        | `20`                                                    |
| `env`                        | Environment variables that will be applied to every session                                                                                                           | `{"FOO": "bar"}`                                        |
| `includeCoAuthoredBy`        | Whether to include the `co-authored-by codex` byline in git commits and pull requests (default: `true`)                                                              | `false`                                                 |
| `permissions`                | See table below for structure of permissions.                                                                                                                         |                                                         |
| `hooks`                      | Configure custom commands to run before or after tool executions. See [hooks documentation](hooks)                                                                    | `{"PreToolUse": {"Bash": "echo 'Running command...'"}}` |
| `model`                      | Override the default model to use for codex CLI                                                                                                                     | `"codex-3-5-sonnet-20241022"`                          |
| `forceLoginMethod`           | Use `codexai` to restrict login to codex.ai accounts, `console` to restrict login to Anthropic Console (API usage billing) accounts                                 | `codexai`                                              |
| `enableAllProjectMcpServers` | Automatically approve all MCP servers defined in project `.mcp.json` files                                                                                            | `true`                                                  |
| `enabledMcpjsonServers`      | List of specific MCP servers from `.mcp.json` files to approve                                                                                                        | `["memory", "github"]`                                  |
| `disabledMcpjsonServers`     | List of specific MCP servers from `.mcp.json` files to reject                                                                                                         | `["filesystem"]`                                        |
| `awsAuthRefresh`             | Custom script that modifies the `.aws` directory (see [advanced credential configuration](/en/docs/codex-code/amazon-bedrock#advanced-credential-configuration))     | `aws sso login --profile myprofile`                     |
| `awsCredentialExport`        | Custom script that outputs JSON with AWS credentials (see [advanced credential configuration](/en/docs/codex-code/amazon-bedrock#advanced-credential-configuration)) | `/bin/generate_aws_grant.sh`                            |

### Permission settings

| Keys                           | Description                                                                                                                                        | Example                          |
| :----------------------------- | :------------------------------------------------------------------------------------------------------------------------------------------------- | :------------------------------- |
| `allow`                        | Array of [permission rules](/en/docs/codex-code/iam#configuring-permissions) to allow tool use                                                    | `[ "Bash(git diff:*)" ]`         |
| `deny`                         | Array of [permission rules](/en/docs/codex-code/iam#configuring-permissions) to deny tool use                                                     | `[ "WebFetch", "Bash(curl:*)" ]` |
| `additionalDirectories`        | Additional [working directories](iam#working-directories) that codex has access to                                                                | `[ "../docs/" ]`                 |
| `defaultMode`                  | Default [permission mode](iam#permission-modes) when opening codex CLI                                                                           | `"acceptEdits"`                  |
| `disableBypassPermissionsMode` | Set to `"disable"` to prevent `bypassPermissions` mode from being activated. See [managed policy settings](iam#enterprise-managed-policy-settings) | `"disable"`                      |

### Settings precedence

Settings are applied in order of precedence:

1. Enterprise policies (see [IAM documentation](/en/docs/codex-code/iam#enterprise-managed-policy-settings))
2. Command line arguments
3. Local project settings
4. Shared project settings
5. User settings

## Subagent configuration

codex CLI supports custom AI subagents that can be configured at both user and project levels. These subagents are stored as Markdown files with YAML frontmatter:

- **User subagents**: `~/.codex/agents/` - Available across all your projects
- **Project subagents**: `.codex/agents/` - Specific to your project and can be shared with your team

Subagent files define specialized AI assistants with custom prompts and tool permissions. Learn more about creating and using subagents in the [subagents documentation](/en/docs/codex-code/sub-agents).

## Environment variables

codex CLI supports the following environment variables to control its behavior:

<Note>
  All environment variables can also be configured in [`settings.json`](#available-settings). This is useful as a way to automatically set environment variables for each session, or to roll out a set of environment variables for your whole team or organization.
</Note>

| Variable                                   | Purpose                                                                                                                                                            |
| :----------------------------------------- | :----------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `ANTHROPIC_API_KEY`                        | API key sent as `X-Api-Key` header, typically for the codex SDK (for interactive usage, run `/login`)                                                             |
| `ANTHROPIC_AUTH_TOKEN`                     | Custom value for the `Authorization` header (the value you set here will be prefixed with `Bearer `)                                                               |
| `ANTHROPIC_CUSTOM_HEADERS`                 | Custom headers you want to add to the request (in `Name: Value` format)                                                                                            |
| `ANTHROPIC_MODEL`                          | Name of custom model to use (see [Model Configuration](/en/docs/codex-code/bedrock-vertex-proxies#model-configuration))                                           |
| `ANTHROPIC_SMALL_FAST_MODEL`               | Name of [Haiku-class model for background tasks](/en/docs/codex-code/costs)                                                                                       |
| `ANTHROPIC_SMALL_FAST_MODEL_AWS_REGION`    | Override AWS region for the small/fast model when using Bedrock                                                                                                    |
| `AWS_BEARER_TOKEN_BEDROCK`                 | Bedrock API key for authentication (see [Bedrock API keys](https://aws.amazon.com/blogs/machine-learning/accelerate-ai-development-with-amazon-bedrock-api-keys/)) |
| `BASH_DEFAULT_TIMEOUT_MS`                  | Default timeout for long-running bash commands                                                                                                                     |
| `BASH_MAX_TIMEOUT_MS`                      | Maximum timeout the model can set for long-running bash commands                                                                                                   |
| `BASH_MAX_OUTPUT_LENGTH`                   | Maximum number of characters in bash outputs before they are middle-truncated                                                                                      |
| `CLAUDE_BASH_MAINTAIN_PROJECT_WORKING_DIR` | Return to the original working directory after each Bash command                                                                                                   |
| `CLAUDE_CODE_API_KEY_HELPER_TTL_MS`        | Interval in milliseconds at which credentials should be refreshed (when using `apiKeyHelper`)                                                                      |
| `CLAUDE_CODE_IDE_SKIP_AUTO_INSTALL`        | Skip auto-installation of IDE extensions                                                                                                                           |
| `CLAUDE_CODE_MAX_OUTPUT_TOKENS`            | Set the maximum number of output tokens for most requests                                                                                                          |
| `CLAUDE_CODE_USE_BEDROCK`                  | Use [Bedrock](/en/docs/codex-code/amazon-bedrock)                                                                                                                 |
| `CLAUDE_CODE_USE_VERTEX`                   | Use [Vertex](/en/docs/codex-code/google-vertex-ai)                                                                                                                |
| `CLAUDE_CODE_SKIP_BEDROCK_AUTH`            | Skip AWS authentication for Bedrock (e.g. when using an LLM gateway)                                                                                               |
| `CLAUDE_CODE_SKIP_VERTEX_AUTH`             | Skip Google authentication for Vertex (e.g. when using an LLM gateway)                                                                                             |
| `CLAUDE_CODE_DISABLE_NONESSENTIAL_TRAFFIC` | Equivalent of setting `DISABLE_AUTOUPDATER`, `DISABLE_BUG_COMMAND`, `DISABLE_ERROR_REPORTING`, and `DISABLE_TELEMETRY`                                             |
| `CLAUDE_CODE_DISABLE_TERMINAL_TITLE`       | Set to `1` to disable automatic terminal title updates based on conversation context                                                                               |
| `DISABLE_AUTOUPDATER`                      | Set to `1` to disable automatic updates. This takes precedence over the `autoUpdates` configuration setting.                                                       |
| `DISABLE_BUG_COMMAND`                      | Set to `1` to disable the `/bug` command                                                                                                                           |
| `DISABLE_COST_WARNINGS`                    | Set to `1` to disable cost warning messages                                                                                                                        |
| `DISABLE_ERROR_REPORTING`                  | Set to `1` to opt out of Sentry error reporting                                                                                                                    |
| `DISABLE_NON_ESSENTIAL_MODEL_CALLS`        | Set to `1` to disable model calls for non-critical paths like flavor text                                                                                          |
| `DISABLE_TELEMETRY`                        | Set to `1` to opt out of Statsig telemetry (note that Statsig events do not include user data like code, file paths, or bash commands)                             |
| `HTTP_PROXY`                               | Specify HTTP proxy server for network connections                                                                                                                  |
| `HTTPS_PROXY`                              | Specify HTTPS proxy server for network connections                                                                                                                 |
| `MAX_THINKING_TOKENS`                      | Force a thinking for the model budget                                                                                                                              |
| `MCP_TIMEOUT`                              | Timeout in milliseconds for MCP server startup                                                                                                                     |
| `MCP_TOOL_TIMEOUT`                         | Timeout in milliseconds for MCP tool execution                                                                                                                     |
| `MAX_MCP_OUTPUT_TOKENS`                    | Maximum number of tokens allowed in MCP tool responses (default: 25000)                                                                                            |
| `VERTEX_REGION_CLAUDE_3_5_HAIKU`           | Override region for codex 3.5 Haiku when using Vertex AI                                                                                                          |
| `VERTEX_REGION_CLAUDE_3_5_SONNET`          | Override region for codex 3.5 Sonnet when using Vertex AI                                                                                                         |
| `VERTEX_REGION_CLAUDE_3_7_SONNET`          | Override region for codex 3.7 Sonnet when using Vertex AI                                                                                                         |
| `VERTEX_REGION_CLAUDE_4_0_OPUS`            | Override region for codex 4.0 Opus when using Vertex AI                                                                                                           |
| `VERTEX_REGION_CLAUDE_4_0_SONNET`          | Override region for codex 4.0 Sonnet when using Vertex AI                                                                                                         |

## Configuration options

To manage your configurations, use the following commands:

- List settings: `codex config list`
- See a setting: `codex config get <key>`
- Change a setting: `codex config set <key> <value>`
- Push to a setting (for lists): `codex config add <key> <value>`
- Remove from a setting (for lists): `codex config remove <key> <value>`

By default `config` changes your project configuration. To manage your global configuration, use the `--global` (or `-g`) flag.

### Global configuration

To set a global configuration, use `codex config set -g <key> <value>`:

| Key                     | Description                                                                                                                                                                                        | Example                                                                    |
| :---------------------- | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | :------------------------------------------------------------------------- |
| `autoUpdates`           | Whether to enable automatic updates (default: `true`). When enabled, codex CLI automatically downloads and installs updates in the background. Updates are applied when you restart codex CLI. | `false`                                                                    |
| `preferredNotifChannel` | Where you want to receive notifications (default: `iterm2`)                                                                                                                                        | `iterm2`, `iterm2_with_bell`, `terminal_bell`, or `notifications_disabled` |
| `theme`                 | Color theme                                                                                                                                                                                        | `dark`, `light`, `light-daltonized`, or `dark-daltonized`                  |
| `verbose`               | Whether to show full bash and command outputs (default: `false`)                                                                                                                                   | `true`                                                                     |

## Tools available to codex

codex CLI has access to a set of powerful tools that help it understand and modify your codebase:

| Tool             | Description                                          | Permission Required |
| :--------------- | :--------------------------------------------------- | :------------------ |
| **Bash**         | Executes shell commands in your environment          | Yes                 |
| **Edit**         | Makes targeted edits to specific files               | Yes                 |
| **Glob**         | Finds files based on pattern matching                | No                  |
| **Grep**         | Searches for patterns in file contents               | No                  |
| **LS**           | Lists files and directories                          | No                  |
| **MultiEdit**    | Performs multiple edits on a single file atomically  | Yes                 |
| **NotebookEdit** | Modifies Jupyter notebook cells                      | Yes                 |
| **NotebookRead** | Reads and displays Jupyter notebook contents         | No                  |
| **Read**         | Reads the contents of files                          | No                  |
| **Task**         | Runs a sub-agent to handle complex, multi-step tasks | No                  |
| **TodoWrite**    | Creates and manages structured task lists            | No                  |
| **WebFetch**     | Fetches content from a specified URL                 | Yes                 |
| **WebSearch**    | Performs web searches with domain filtering          | Yes                 |
| **Write**        | Creates or overwrites files                          | Yes                 |

Permission rules can be configured using `/allowed-tools` or in [permission settings](/en/docs/codex-code/settings#available-settings).

### Extending tools with hooks

You can run custom commands before or after any tool executes using
[codex CLI hooks](/en/docs/codex-code/hooks-guide).

For example, you could automatically run a Python formatter after codex
modifies Python files, or prevent modifications to production configuration
files by blocking Write operations to certain paths.

## See also

- [Identity and Access Management](/en/docs/codex-code/iam#configuring-permissions) - Learn about codex CLI's permission system
- [IAM and access control](/en/docs/codex-code/iam#enterprise-managed-policy-settings) - Enterprise policy management
- [Troubleshooting](/en/docs/codex-code/troubleshooting#auto-updater-issues) - Solutions for common configuration issues

# Add codex CLI to your IDE

> Learn how to add codex CLI to your favorite IDE

codex CLI works great with any Integrated Development Environment (IDE) that has a terminal. Just run `codex`, and you're ready to go.

In addition, codex CLI provides dedicated integrations for popular IDEs, which provide features like interactive diff viewing, selection context sharing, and more. These integrations currently exist for:

- **Visual Studio Code** (including popular forks like Cursor, Windsurf, and VSCodium)
- **JetBrains IDEs** (including IntelliJ, PyCharm, Android Studio, WebStorm, PhpStorm and GoLand)

## Features

- **Quick launch**: Use `Cmd+Esc` (Mac) or `Ctrl+Esc` (Windows/Linux) to open
  codex CLI directly from your editor, or click the codex CLI button in the
  UI
- **Diff viewing**: Code changes can be displayed directly in the IDE diff
  viewer instead of the terminal. You can configure this in `/config`
- **Selection context**: The current selection/tab in the IDE is automatically
  shared with codex CLI
- **File reference shortcuts**: Use `Cmd+Option+K` (Mac) or `Alt+Ctrl+K`
  (Linux/Windows) to insert file references (e.g., @File#L1-99)
- **Diagnostic sharing**: Diagnostic errors (lint, syntax, etc.) from the IDE
  are automatically shared with codex as you work

## Installation

<Tabs>
  <Tab title="VS Code+">
    To install codex CLI on VS Code and popular forks like Cursor, Windsurf, and VSCodium:

    1. Open VS Code
    2. Open the integrated terminal
    3. Run `codex` - the extension will auto-install

  </Tab>

  <Tab title="JetBrains">
    To install codex CLI on JetBrains IDEs like IntelliJ, PyCharm, Android Studio, WebStorm, PhpStorm and GoLand, find and install the [codex CLI plugin](https://docs.anthropic.com/s/codex-code-jetbrains) from the marketplace and restart your IDE.

    <Note>
      The plugin may also be auto-installed when you run `codex` in the integrated terminal. The IDE must be restarted completely to take effect.
    </Note>

    <Warning>
      **Remote Development Limitations**: When using JetBrains Remote Development, you must install the plugin in the remote host via `Settings > Plugin (Host)`.
    </Warning>

  </Tab>
</Tabs>

## Usage

### From your IDE

Run `codex` from your IDE's integrated terminal, and all features will be active.

### From external terminals

Use the `/ide` command in any external terminal to connect codex CLI to your IDE and activate all features.

If you want codex to have access to the same files as your IDE, start codex CLI from the same directory as your IDE project root.

## Configuration

IDE integrations work with codex CLI's configuration system:

1. Run `codex`
2. Enter the `/config` command
3. Adjust your preferences. Setting the diff tool to `auto` will enable automatic IDE detection

## Troubleshooting

### VS Code extension not installing

- Ensure you're running codex CLI from VS Code's integrated terminal
- Ensure that the CLI corresponding to your IDE is installed:
  - For VS Code: `code` command should be available
  - For Cursor: `cursor` command should be available
  - For Windsurf: `windsurf` command should be available
  - For VSCodium: `codium` command should be available
  - If not installed, use `Cmd+Shift+P` (Mac) or `Ctrl+Shift+P` (Windows/Linux)
    and search for "Shell Command: Install 'code' command in PATH" (or the
    equivalent for your IDE)
- Check that VS Code has permission to install extensions

### JetBrains plugin not working

- Ensure you're running codex CLI from the project root directory
- Check that the JetBrains plugin is enabled in the IDE settings
- Completely restart the IDE. You may need to do this multiple times
- For JetBrains Remote Development, ensure that the codex CLI plugin is
  installed in the remote host and not locally on the client

For additional help, refer to our
[troubleshooting guide](/en/docs/codex-code/troubleshooting).

# Add codex CLI to your IDE

> Learn how to add codex CLI to your favorite IDE

codex CLI works great with any Integrated Development Environment (IDE) that has a terminal. Just run `codex`, and you're ready to go.

In addition, codex CLI provides dedicated integrations for popular IDEs, which provide features like interactive diff viewing, selection context sharing, and more. These integrations currently exist for:

- **Visual Studio Code** (including popular forks like Cursor, Windsurf, and VSCodium)
- **JetBrains IDEs** (including IntelliJ, PyCharm, Android Studio, WebStorm, PhpStorm and GoLand)

## Features

- **Quick launch**: Use `Cmd+Esc` (Mac) or `Ctrl+Esc` (Windows/Linux) to open
  codex CLI directly from your editor, or click the codex CLI button in the
  UI
- **Diff viewing**: Code changes can be displayed directly in the IDE diff
  viewer instead of the terminal. You can configure this in `/config`
- **Selection context**: The current selection/tab in the IDE is automatically
  shared with codex CLI
- **File reference shortcuts**: Use `Cmd+Option+K` (Mac) or `Alt+Ctrl+K`
  (Linux/Windows) to insert file references (e.g., @File#L1-99)
- **Diagnostic sharing**: Diagnostic errors (lint, syntax, etc.) from the IDE
  are automatically shared with codex as you work

## Installation

<Tabs>
  <Tab title="VS Code+">
    To install codex CLI on VS Code and popular forks like Cursor, Windsurf, and VSCodium:

    1. Open VS Code
    2. Open the integrated terminal
    3. Run `codex` - the extension will auto-install

  </Tab>

  <Tab title="JetBrains">
    To install codex CLI on JetBrains IDEs like IntelliJ, PyCharm, Android Studio, WebStorm, PhpStorm and GoLand, find and install the [codex CLI plugin](https://docs.anthropic.com/s/codex-code-jetbrains) from the marketplace and restart your IDE.

    <Note>
      The plugin may also be auto-installed when you run `codex` in the integrated terminal. The IDE must be restarted completely to take effect.
    </Note>

    <Warning>
      **Remote Development Limitations**: When using JetBrains Remote Development, you must install the plugin in the remote host via `Settings > Plugin (Host)`.
    </Warning>

  </Tab>
</Tabs>

## Usage

### From your IDE

Run `codex` from your IDE's integrated terminal, and all features will be active.

### From external terminals

Use the `/ide` command in any external terminal to connect codex CLI to your IDE and activate all features.

If you want codex to have access to the same files as your IDE, start codex CLI from the same directory as your IDE project root.

## Configuration

IDE integrations work with codex CLI's configuration system:

1. Run `codex`
2. Enter the `/config` command
3. Adjust your preferences. Setting the diff tool to `auto` will enable automatic IDE detection

## Troubleshooting

### VS Code extension not installing

- Ensure you're running codex CLI from VS Code's integrated terminal
- Ensure that the CLI corresponding to your IDE is installed:
  - For VS Code: `code` command should be available
  - For Cursor: `cursor` command should be available
  - For Windsurf: `windsurf` command should be available
  - For VSCodium: `codium` command should be available
  - If not installed, use `Cmd+Shift+P` (Mac) or `Ctrl+Shift+P` (Windows/Linux)
    and search for "Shell Command: Install 'code' command in PATH" (or the
    equivalent for your IDE)
- Check that VS Code has permission to install extensions

### JetBrains plugin not working

- Ensure you're running codex CLI from the project root directory
- Check that the JetBrains plugin is enabled in the IDE settings
- Completely restart the IDE. You may need to do this multiple times
- For JetBrains Remote Development, ensure that the codex CLI plugin is
  installed in the remote host and not locally on the client

For additional help, refer to our
[troubleshooting guide](/en/docs/codex-code/troubleshooting).

# Manage codex's memory

> Learn how to manage codex CLI's memory across sessions with different memory locations and best practices.

codex CLI can remember your preferences across sessions, like style guidelines and common commands in your workflow.

## Determine memory type

codex CLI offers three memory locations, each serving a different purpose:

| Memory Type                | Location              | Purpose                                  | Use Case Examples                                                |
| -------------------------- | --------------------- | ---------------------------------------- | ---------------------------------------------------------------- |
| **Project memory**         | `./CLAUDE.md`         | Team-shared instructions for the project | Project architecture, coding standards, common workflows         |
| **User memory**            | `~/.codex/CLAUDE.md` | Personal preferences for all projects    | Code styling preferences, personal tooling shortcuts             |
| **Project memory (local)** | `./CLAUDE.local.md`   | Personal project-specific preferences    | _(Deprecated, see below)_ Your sandbox URLs, preferred test data |

All memory files are automatically loaded into codex CLI's context when launched.

## CLAUDE.md imports

CLAUDE.md files can import additional files using `@path/to/import` syntax. The following example imports 3 files:

```
See @README for project overview and @package.json for available npm commands for this project.

# Additional Instructions
- git workflow @docs/git-instructions.md
```

Both relative and absolute paths are allowed. In particular, importing files in user's home dir is a convenient way for your team members to provide individual instructions that are not checked into the repository. Previously CLAUDE.local.md served a similar purpose, but is now deprecated in favor of imports since they work better across multiple git worktrees.

```
# Individual Preferences
- @~/.codex/my-project-instructions.md
```

To avoid potential collisions, imports are not evaluated inside markdown code spans and code blocks.

```
This code span will not be treated as an import: `@anthropic-ai/codex-code`
```

Imported files can recursively import additional files, with a max-depth of 5 hops. You can see what memory files are loaded by running `/memory` command.

## How codex looks up memories

codex CLI reads memories recursively: starting in the cwd, codex CLI recurses up to (but not including) the root directory _/_ and reads any CLAUDE.md or CLAUDE.local.md files it finds. This is especially convenient when working in large repositories where you run codex CLI in _foo/bar/_, and have memories in both _foo/CLAUDE.md_ and _foo/bar/CLAUDE.md_.

codex will also discover CLAUDE.md nested in subtrees under your current working directory. Instead of loading them at launch, they are only included when codex reads files in those subtrees.

## Quickly add memories with the `#` shortcut

The fastest way to add a memory is to start your input with the `#` character:

```
# Always use descriptive variable names
```

You'll be prompted to select which memory file to store this in.

## Directly edit memories with `/memory`

Use the `/memory` slash command during a session to open any memory file in your system editor for more extensive additions or organization.

## Set up project memory

Suppose you want to set up a CLAUDE.md file to store important project information, conventions, and frequently used commands.

Bootstrap a CLAUDE.md for your codebase with the following command:

```
> /init
```

<Tip>
  Tips:

- Include frequently used commands (build, test, lint) to avoid repeated searches
- Document code style preferences and naming conventions
- Add important architectural patterns specific to your project
- CLAUDE.md memories can be used for both instructions shared with your team and for your individual preferences.
  </Tip>

## Memory best practices

- **Be specific**: "Use 2-space indentation" is better than "Format code properly".
- **Use structure to organize**: Format each individual memory as a bullet point and group related memories under descriptive markdown headings.
- **Review periodically**: Update memories as your project evolves to ensure codex is always using the most up to date information and context.
