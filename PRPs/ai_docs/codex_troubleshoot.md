# Troubleshooting

> Discover solutions to common issues with codex CLI installation and usage.

## Common installation issues

### Windows installation issues: errors in WSL

You might encounter the following issues in WSL:

**OS/platform detection issues**: If you receive an error during installation, WSL may be using Windows `npm`. Try:

- Run `npm config set os linux` before installation
- Install with `npm install -g @anthropic-ai/codex-code --force --no-os-check` (Do NOT use `sudo`)

**Node not found errors**: If you see `exec: node: not found` when running `codex`, your WSL environment may be using a Windows installation of Node.js. You can confirm this with `which npm` and `which node`, which should point to Linux paths starting with `/usr/` rather than `/mnt/c/`. To fix this, try installing Node via your Linux distribution's package manager or via [`nvm`](https://github.com/nvm-sh/nvm).

### Linux and Mac installation issues: permission or command not found errors

When installing codex CLI with npm, `PATH` problems may prevent access to `codex`.
You may also encounter permission errors if your npm global prefix is not user writable (eg. `/usr`, or `/usr/local`).

#### Recommended solution: Native codex CLI installation

codex CLI has a native installation that doesn't depend on npm or Node.js.

<Note>
  The native codex CLI installer is currently in beta.
</Note>

Use the following command to run the native installer.

**macOS, Linux, WSL:**

```bash
# Install stable version (default)
curl -fsSL https://codex.ai/install.sh | bash

# Install latest version
curl -fsSL https://codex.ai/install.sh | bash -s latest

# Install specific version number
curl -fsSL https://codex.ai/install.sh | bash -s 1.0.58
```

**Windows PowerShell:**

```powershell
# Install stable version (default)
irm https://codex.ai/install.ps1 | iex

# Install latest version
& ([scriptblock]::Create((irm https://codex.ai/install.ps1))) latest

# Install specific version number
& ([scriptblock]::Create((irm https://codex.ai/install.ps1))) 1.0.58

```

This command installs the appropriate build of codex CLI for your operating system and architecture and adds a symlink to the installation at `~/.local/bin/codex`.

<Tip>
  Make sure that you have the installation directory in your system PATH.
</Tip>

#### Alternative solution: Migrate to local installation

Alternatively, if codex CLI will run, you can migrate to a local installation:

```bash
codex migrate-installer
```

This moves codex CLI to `~/.codex/local/` and sets up an alias in your shell configuration. No `sudo` is required for future updates.

After migration, restart your shell, and then verify your installation:

On macOS/Linux/WSL:

```bash
which codex  # Should show an alias to ~/.codex/local/codex
```

On Windows:

```powershell
where codex  # Should show path to codex executable
```

Verify installation:

```bash
codex doctor # Check installation health
```

## Permissions and authentication

### Repeated permission prompts

If you find yourself repeatedly approving the same commands, you can allow specific tools
to run without approval using the `/permissions` command. See [Permissions docs](/en/docs/codex-code/iam#configuring-permissions).

### Authentication issues

If you're experiencing authentication problems:

1. Run `/logout` to sign out completely
2. Close codex CLI
3. Restart with `codex` and complete the authentication process again

If problems persist, try:

```bash
rm -rf ~/.config/codex-code/auth.json
codex
```

This removes your stored authentication information and forces a clean login.

## Performance and stability

### High CPU or memory usage

codex CLI is designed to work with most development environments, but may consume significant resources when processing large codebases. If you're experiencing performance issues:

1. Use `/compact` regularly to reduce context size
2. Close and restart codex CLI between major tasks
3. Consider adding large build directories to your `.gitignore` file

### Command hangs or freezes

If codex CLI seems unresponsive:

1. Press Ctrl+C to attempt to cancel the current operation
2. If unresponsive, you may need to close the terminal and restart

### ESC key not working in JetBrains (IntelliJ, PyCharm, etc.) terminals

If you're using codex CLI in JetBrains terminals and the ESC key doesn't interrupt the agent as expected, this is likely due to a keybinding clash with JetBrains' default shortcuts.

To fix this issue:

1. Go to Settings -> Tools -> Terminal
2. Click the "Configure terminal keybindings" hyperlink next to "Override IDE Shortcuts"
3. Within the terminal keybindings, scroll down to "Switch focus to Editor" and delete that shortcut

This will allow the ESC key to properly function for canceling codex CLI operations instead of being captured by PyCharm's "Switch focus to Editor" action.

## Getting more help

If you're experiencing issues not covered here:

1. Use the `/bug` command within codex CLI to report problems directly to Anthropic
2. Check the [GitHub repository](https://github.com/anthropics/codex-code) for known issues
3. Run `/doctor` to check the health of your codex CLI installation
4. Ask codex directly about its capabilities and features - codex has built-in access to its documentation
