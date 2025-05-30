# Claude MCP Manager

A command-line tool for macOS to define, manage, and switch between different sets of `mcpServers` configurations (profiles) for the Claude Desktop application.

## Goal

The Claude desktop app stores its configuration, including the `mcpServers` endpoints, in a JSON file at `~/Library/Application Support/Claude/claude_desktop_config.json`. This tool allows you to define individual server configurations and group them into named profiles (e.g., 'default', 'work', 'experimental'). Activating a profile updates the `mcpServers` block in the main Claude config file with the servers defined in that profile, automatically restarting Claude to apply the changes. By separating server definitions and utilizing environment variables for secrets (see "How it Works"), you can easily share server configurations or commit profiles to version control without exposing sensitive credentials.

## Quick Install (Recommended)

Run the following command in your macOS terminal:

```bash
curl -fsSL https://raw.githubusercontent.com/BLTGV/claude-mcp-manager/main/install.sh | sh
```

This downloads and executes the installer script, which handles dependency checks, copying files, and setting up the command-line tool.

## Requirements

*   **macOS:** The script relies on macOS-specific paths and commands (`pgrep`, `kill`, `open`). **It will not work on Windows or Linux.**
*   **jq:** The script uses `jq` for JSON processing. Install via Homebrew: `brew install jq`.
*   **bash:** Requires a reasonably modern version of bash (>= 3.2) for features like regex matching (`=~`). Standard macOS bash is sufficient.

## Installation

### Manual Installation

1.  Clone the repository or download the source code.
2.  Navigate to the project directory in your terminal.
3.  Make the installer executable: `chmod +x install.sh`
4.  Run the installer:
    ```bash
    ./install.sh
    ```
    *   The script will attempt to install to `/usr/local/lib/claude-mcp-manager` and create a symlink in `/usr/local/bin`. This might require `sudo` if you don't have write permissions.
    *   If `/usr/local` is not writable, it will fall back to `$HOME/.local/lib` and `$HOME/.local/bin`.
    *   The installer will notify you if the chosen `bin` directory is not in your `$PATH` and provide instructions to add it.

## How it Works

*   **Configuration Directory:** Stores all data in `~/.config/claude/`.
*   **Servers Directory:** Individual server definitions are stored as `.json` files in `~/.config/claude/servers/`.
*   **Profiles Directory:** Profiles, which contain lists of server names, are stored as `.json` files in `~/.config/claude/profiles/`.
    *   Each profile file looks like: `{\"servers\": [\"server1_name\", \"server2_name\"]}`
*   **Target File:** The script modifies the `mcpServers` key within the official Claude config file: `~/Library/Application Support/Claude/claude_desktop_config.json`.
*   **Secrets (`.env` file):** You can store sensitive values (like API keys) in `~/.config/claude/.env`. Server definition files can reference these using placeholders like `{{ENV:YOUR_API_KEY}}` or `{{DOTENV:YOUR_API_KEY}}`. These are resolved when a profile is activated.
*   **Tracking (`loaded` file):** The script keeps track of the *name* of the currently active profile in `~/.config/claude/loaded`.
*   **Backup (`last.json`):** Before applying a new profile, the script saves the *current* `mcpServers` block from the target file into `~/.config/claude/last.json`, allowing easy rollback using the `last` command.
*   **First Run:** The installer runs `claude-mcp-manager server list` to ensure the config directories are created upon successful installation.

## Usage

```
claude-mcp-manager <command | profile_name | option>
```

### Primary Commands

*   `<profile_name>`: Activate the specified profile. Reads server definitions listed in the profile, resolves secrets, constructs the `mcpServers` block, backs up the current block in the main config, writes the new block, and restarts Claude.
*   `last`: Restore the `mcpServers` block that was active before the last profile activation (from `last.json`) and restart Claude.
*   `server <subcommand>`: Manage server definitions. Run `claude-mcp-manager server help` for details.
*   `profile <subcommand>`: Manage profiles. Run `claude-mcp-manager profile help` for details.

### Server Subcommands (`claude-mcp-manager server ...`)

*   `list`: List available server definition files found in `~/.config/claude/servers/`.
*   `show <name>`: Display the JSON content of the specified server definition file.
*   `add <name> --file <path>`: Add a new server definition by copying the content from `<path>`.
*   `add <name> --json '{\"key\":\"value\"}'`: Add a new server definition using the provided JSON string.
*   `edit <name>`: Open the specified server definition file in your `$EDITOR` (or default like vim/nano).
*   `remove <name>`: Delete the specified server definition file (prompts for confirmation).
*   `extract [--profile <name>]`: Read the `mcpServers` block from the main Claude config, save each server as a separate file in `~/.config/claude/servers/`. Optionally, create or update the specified profile `<name>` to list all extracted servers.

### Profile Subcommands (`claude-mcp-manager profile ...`)

*   `list`: List available profile files found in `~/.config/claude/profiles/`.
*   `show <name>`: List the server names contained within the specified profile file and indicate if the corresponding server definition file exists.
*   `create <name>`: Create a new, empty profile file.
*   `copy <src> <dest>`: Copy an existing profile file to a new name.
*   `edit <name>`: Open the specified profile file in your `$EDITOR` (or default like vim/nano) to manually edit the server list.
*   `add <profile> <server...>`: Add one or more server names to the specified profile's `"servers"` array.
*   `remove <profile> <server...>`: Remove one or more server names from the specified profile's `"servers"` array.
*   `delete <name>`: Delete the specified profile file (prompts for confirmation).

### Options

*   `help, -h, --help`: Show the main help message.
*   `version, -v`: Show the script version.

## Acknowledgements

This project was built utilizing [claude-task-master](https://github.com/eyaltoledano/claude-task-master), an AI-driven task management system by Eyal Toledano ([@eyaltoledano](https://github.com/eyaltoledano)) and Ralph Ecom ([@RalphEcom](https://github.com/RalphEcom)), to define, manage, and implement the features described above.

## Shell Completion (Optional)

Tab completion is available for Bash and Zsh shells.

First, ensure `claude-mcp-manager` is installed and accessible in your PATH.

### Bash

1.  **Install `bash-completion@2`:** If you haven't already, install the latest bash completion system using Homebrew:
    ```bash
    brew install bash-completion@2
    ```
2.  **Add to `.bash_profile`:** Follow the instructions printed by `brew info bash-completion@2` to add the necessary line to your `~/.bash_profile` (or `~/.bashrc`). It usually looks like this:
    ```bash
    # Add the following line (or similar) to ~/.bash_profile
    export BASH_COMPLETION_COMPAT_DIR="$(brew --prefix)/etc/bash_completion.d"
    [[ -r "$(brew --prefix)/etc/profile.d/bash_completion.sh" ]] && . "$(brew --prefix)/etc/profile.d/bash_completion.sh"
    ```
3.  **Generate and Install Script:** Run the following command to generate the completion script and place it in the correct directory:
    ```bash
    # Determine the completion directory
    COMPLETION_DIR="$(brew --prefix)/etc/bash_completion.d"
    # Ensure the directory exists
    mkdir -p "$COMPLETION_DIR"
    # Generate the script
    claude-mcp-manager completion bash > "${COMPLETION_DIR}/claude-mcp-manager"
    ```
4.  **Reload Shell:** Open a new terminal window or run `source ~/.bash_profile`.

### Zsh

Zsh has built-in completion support.

1.  **Ensure `compinit`:** Make sure your `~/.zshrc` file includes `autoload -U compinit && compinit` somewhere (usually near the beginning or end).
2.  **Choose a Completion Directory:** Zsh completions are typically stored in a directory listed in your `$fpath` variable. A common user-specific location is `~/.zsh/completion`. Create it if it doesn't exist:
    ```bash
    mkdir -p ~/.zsh/completion
    ```
3.  **Add Directory to `fpath`:** Ensure this directory is added to your Zsh function path in your `~/.zshrc` *before* calling `compinit`:
    ```bash
    # Add near the top of ~/.zshrc
    fpath=(~/.zsh/completion $fpath)
    
    # ... other zsh config ...
    
    # Ensure compinit is loaded
    autoload -U compinit && compinit
    ```
4.  **Generate and Install Script:** Run the following command to generate the completion script. Note the filename starts with an underscore (`_`):
    ```bash
    claude-mcp-manager completion zsh > ~/.zsh/completion/_claude-mcp-manager
    ```
5.  **Reload Shell:** Open a new terminal window or run `source ~/.zshrc`.

After installation, you should be able to use tab completion for commands, subcommands, profile names, and server names.

## Uninstall

To uninstall, run the following command:
```bash
claude-mcp-manager uninstall
```
This will remove the script and the command symlink. It will ask for confirmation first.

To *also* remove the configuration directory (`~/.config/claude`), which contains all your profiles, server definitions, logs, and the `.env` file (potentially containing secrets), use:
```bash
claude-mcp-manager uninstall --remove-config
```
**Warning:** Removing the configuration directory is permanent and requires a separate confirmation. 