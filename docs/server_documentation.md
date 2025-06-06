# server.py Module Documentation

## Module Overview

"""
MCP Git Server

This module implements a server for the MCP (Multi-Agent Collaboration Platform)
that provides a set of Git-related tools and AI-powered code editing capabilities
using Aider. It allows clients to interact with Git repositories, perform file
operations, execute commands, and initiate AI-driven code modifications.

Key Components:
- Git Operations: Functions for common Git commands like status, diff, commit,
  reset, log, branch creation, checkout, and applying diffs.
- File Operations: Tools for reading, writing, and searching/replacing content
  within files.
- Command Execution: A general-purpose tool to execute arbitrary shell commands.
- AI-Powered Editing (Aider): Integration with the Aider tool for advanced
  code modifications based on natural language instructions.
- Configuration Loading: Utilities to load Aider-specific configurations and
  environment variables from various locations (.aider.conf.yml, .env).
- MCP Server Integration: Exposes these functionalities as MCP tools, allowing
  them to be called by agents.
- Starlette Application: Sets up an HTTP server with SSE (Server-Sent Events)
  for communication with MCP clients.
"""

## Data Models

### GitStatus
Represents the input schema for the `git_status` tool.

| Field      | Type | Description                                            |
|------------|------|--------------------------------------------------------|
| `repo_path`| `str`| The absolute path to the Git repository's working directory. |

### GitDiffAll
Represents the input schema for the `git_diff_all` tool.

| Field      | Type | Description                                            |
|------------|------|--------------------------------------------------------|
| `repo_path`| `str`| The absolute path to the Git repository's working directory. |

### GitDiff
Represents the input schema for the `git_diff` tool.

| Field      | Type | Description                                            |
|------------|------|--------------------------------------------------------|
| `repo_path`| `str`| The absolute path to the Git repository's working directory. |
| `target`   | `str`| The target (e.g., branch name, commit hash, tag) to diff against. For example, 'main', 'HEAD~1', or a full commit SHA. |

### GitCommit
Represents the input schema for the `git_stage_and_commit` tool.

| Field      | Type | Description                                            |
|------------|------|--------------------------------------------------------|
| `repo_path`| `str`| The absolute path to the Git repository's working directory. |
| `message`  | `str`| The commit message for the changes.                    |
| `files`    | `Optional[List[str]]`| An optional list of specific file paths (relative to the repository root) to stage before committing. If not provided, all changes will be staged. |

### GitReset
Represents the input schema for the `git_reset` tool.

| Field      | Type | Description                                            |
|------------|------|--------------------------------------------------------|
| `repo_path`| `str`| The absolute path to the Git repository's working directory. |

### GitLog
Represents the input schema for the `git_log` tool.

| Field      | Type | Description                                            |
|------------|------|--------------------------------------------------------|
| `repo_path`| `str`| The absolute path to the Git repository's working directory. |
| `max_count`| `int`| The maximum number of commit entries to retrieve. Defaults to 10. |

### GitCreateBranch
Represents the input schema for the `git_create_branch` tool.

| Field        | Type | Description                                            |
|--------------|------|--------------------------------------------------------|
| `repo_path`  | `str`| The absolute path to the Git repository's working directory. |
| `branch_name`| `str`| The name of the new branch to create.                  |
| `base_branch`| `Optional[str]`| Optional. The name of the branch or commit hash to base the new branch on. If not provided, the new branch will be based on the current active branch. |

### GitCheckout
Represents the input schema for the `git_checkout` tool.

| Field        | Type | Description                                            |
|--------------|------|--------------------------------------------------------|
| `repo_path`  | `str`| The absolute path to the Git repository's working directory. |
| `branch_name`| `str`| The name of the branch to checkout.                    |

### GitShow
Represents the input schema for the `git_show` tool.

| Field      | Type | Description                                            |
|------------|------|--------------------------------------------------------|
| `repo_path`| `str`| The absolute path to the Git repository's working directory. |
| `revision` | `str`| The commit hash or reference (e.g., 'HEAD', 'main', 'abc1234') to show details for. |

### GitApplyDiff
Represents the input schema for the `git_apply_diff` tool.

| Field        | Type | Description                                            |
|--------------|------|--------------------------------------------------------|
| `repo_path`  | `str`| The absolute path to the Git repository's working directory. |
| `diff_content`| `str`| The diff content string to apply to the repository. This should be in a unified diff format. |

### GitReadFile
Represents the input schema for the `git_read_file` tool.

| Field      | Type | Description                                            |
|------------|------|--------------------------------------------------------|
| `repo_path`| `str`| The absolute path to the Git repository's working directory. |
| `file_path`| `str`| The path to the file to read, relative to the repository's working directory. |

### SearchAndReplace
Represents the input schema for the `search_and_replace` tool.

| Field          | Type | Description                                            |
|----------------|------|--------------------------------------------------------|
| `repo_path`    | `str`| The absolute path to the Git repository's working directory. |
| `file_path`    | `str`| The path to the file to modify, relative to the repository's working directory. |
| `search_string`| `str`| The string or regex pattern to search for within the file. |
| `replace_string`| `str`| The string to replace all matches of the search string with. |
| `ignore_case`  | `bool`| If true, the search will be case-insensitive. Defaults to false. |
| `start_line`   | `Optional[int]`| Optional. The 1-based starting line number for the search and replace operation (inclusive). If not provided, search starts from the beginning of the file. |
| `end_line`     | `Optional[int]`| Optional. The 1-based ending line number for the search and replace operation (inclusive). If not provided, search continues to the end of the file. |

### WriteToFile
Represents the input schema for the `write_to_file` tool.

| Field      | Type | Description                                            |
|------------|------|--------------------------------------------------------|
| `repo_path`| `str`| The absolute path to the Git repository's working directory. |
| `file_path`| `str`| The path to the file to write to, relative to the repository's working directory. The file will be created if it doesn't exist, or overwritten if it does. |
| `content`  | `str`| The string content to write to the specified file.     |

### ExecuteCommand
Represents the input schema for the `execute_command` tool.

| Field      | Type | Description                                            |
|------------|------|--------------------------------------------------------|
| `repo_path`| `str`| The absolute path to the directory where the command should be executed. |
| `command`  | `str`| The shell command string to execute (e.g., 'ls -l', 'npm install'). |

### AiEdit
Represents the input schema for the `ai_edit` tool.

| Field        | Type | Description                                            |
|--------------|------|--------------------------------------------------------|
| `repo_path`  | `str`| The absolute path to the Git repository's working directory where the AI edit should be performed. |
| `message`    | `str`| A detailed natural language message describing the code changes to make. Be specific about files, desired behavior, and any constraints. |
| `files`      | `List[str]`| A list of file paths (relative to the repository root) that Aider should operate on. This argument is mandatory. |
| `options`    | `Optional[List[str]]`| Optional. A list of additional command-line options to pass directly to Aider (e.g., ['--model=gpt-4o', '--dirty-diff']). Each option should be a string. |
| `edit_format`| `EditFormat`| Optional. The format Aider should use for edits. If not explicitly provided, the default is selected based on the model name: if the model includes 'gemini', defaults to 'diff-fenced'; if the model includes 'gpt', defaults to 'udiff'; otherwise defaults to 'diff'. Options: 'diff', 'diff-fenced', 'udiff', 'whole'. |

### AiderStatus
Represents the input schema for the `aider_status` tool.

| Field            | Type | Description                                            |
|------------------|------|--------------------------------------------------------|
| `repo_path`      | `str`| The absolute path to the Git repository or working directory to check Aider's status within. |
| `check_environment`| `bool`| If true, the tool will also check Aider's configuration, environment variables, and Git repository details. Defaults to true. |

## Enumerations

### EditFormat
An enumeration of supported Aider edit formats.

| Member        | Value      |
|---------------|------------|
| `DIFF`        | `"diff"`   |
| `DIFF_FENCED` | `"diff-fenced"`|
| `UDIFF`       | `"udiff"`  |
| `WHOLE`       | `"whole"`  |

### GitTools
An enumeration of all available Git and related tools.

| Member             | Value                  |
|--------------------|------------------------|
| `STATUS`           | `"git_status"`         |
| `DIFF_ALL`         | `"git_diff_all"`       |
| `DIFF`             | `"git_diff"`           |
| `STAGE_AND_COMMIT` | `"git_stage_and_commit"`|
| `RESET`            | `"git_reset"`          |
| `LOG`              | `"git_log"`            |
| `CREATE_BRANCH`    | `"git_create_branch"`  |
| `CHECKOUT`         | `"git_checkout"`       |
| `SHOW`             | `"git_show"`           |
| `APPLY_DIFF`       | `"git_apply_diff"`     |
| `READ_FILE`        | `"git_read_file"`      |
| `SEARCH_AND_REPLACE`| `"search_and_replace"` |
| `WRITE_TO_FILE`    | `"write_to_file"`      |
| `EXECUTE_COMMAND`  | `"execute_command"`    |
| `AI_EDIT`          | `"ai_edit"`            |
| `AIDER_STATUS`     | `"aider_status"`       |

## Functions

### find_git_root
Finds the root directory of a Git repository by traversing up from the given path.

**Arguments:**
- `path` (`str`): The starting path to search from.

**Returns:**
- `Optional[str]`: The absolute path to the Git repository root, or None if not found.

### load_aider_config
Loads Aider configuration from various possible locations, merging them
in a specific order of precedence (home dir < git root < working dir < specified file).

**Arguments:**
- `repo_path` (`Optional[str]`): The path to the repository or working directory. Defaults to current working directory.
- `config_file` (`Optional[str]`): An optional specific path to an Aider configuration file to load.

**Returns:**
- `Dict[str, Any]`: A dictionary containing the merged Aider configuration.

### load_dotenv_file
Loads environment variables from .env files found in various locations,
merging them in a specific order of precedence (home dir < git root < working dir < specified file).

**Arguments:**
- `repo_path` (`Optional[str]`): The path to the repository or working directory. Defaults to current working directory.
- `env_file` (`Optional[str]`): An optional specific path to a .env file to load.

**Returns:**
- `Dict[str, str]`: A dictionary containing the loaded environment variables.

### run_command
Executes a shell command asynchronously.

**Arguments:**
- `command` (`List[str]`): A list of strings representing the command and its arguments.
- `input_data` (`Optional[str]`): Optional string data to pass to the command's stdin.

**Returns:**
- `Tuple[str, str]`: A tuple containing the stdout and stderr of the command as strings.

### prepare_aider_command
Prepares the full Aider command by adding files and options to the base command.

**Arguments:**
- `base_command` (`List[str]`): The initial Aider command (e.g., ["aider"]).
- `files` (`Optional[List[str]]`): An optional list of file paths to include in the command.
- `options` (`Optional[Dict[str, Any]]`): An optional dictionary of Aider options (e.g., {"yes_always": True}).

**Returns:**
- `List[str]`: A list of strings representing the complete Aider command.

### git_status
Gets the status of the Git working tree.

**Arguments:**
- `repo` (`git.Repo`): The Git repository object.

**Returns:**
- `str`: A string representing the output of `git status`.

### git_diff_all
Shows all changes in the working directory (staged and unstaged, compared to HEAD).

**Arguments:**
- `repo` (`git.Repo`): The Git repository object.

**Returns:**
- `str`: A string representing the output of `git diff HEAD`.

### git_diff
Shows differences between branches or commits.

**Arguments:**
- `repo` (`git.Repo`): The Git repository object.
- `target` (`str`): The target (branch, commit hash, etc.) to diff against.

**Returns:**
- `str`: A string representing the output of `git diff <target>`.

### git_stage_and_commit
Stages changes and commits them to the repository.

**Arguments:**
- `repo` (`git.Repo`): The Git repository object.
- `message` (`str`): The commit message.
- `files` (`Optional[List[str]]`): An optional list of specific files to stage. If None, all changes are staged.

**Returns:**
- `str`: A string indicating the success of the staging and commit operation.

### git_reset
Unstages all staged changes in the repository.

**Arguments:**
- `repo` (`git.Repo`): The Git repository object.

**Returns:**
- `str`: A string indicating that all staged changes have been reset.

### git_log
Shows the commit logs for the repository.

**Arguments:**
- `repo` (`git.Repo`): The Git repository object.
- `max_count` (`int`): The maximum number of commits to retrieve. Defaults to 10.

**Returns:**
- `list[str]`: A list of strings, where each string represents a formatted commit entry.

### git_create_branch
Creates a new branch in the repository.

**Arguments:**
- `repo` (`git.Repo`): The Git repository object.
- `branch_name` (`str`): The name of the new branch.
- `base_branch` (`str | None`): Optional. The name of the branch to base the new branch on. If None, the new branch is based on the current active branch.

**Returns:**
- `str`: A string indicating the successful creation of the branch.

### git_checkout
Switches the current branch to the specified branch.

**Arguments:**
- `repo` (`git.Repo`): The Git repository object.
- `branch_name` (`str`): The name of the branch to checkout.

**Returns:**
- `str`: A string indicating the successful checkout of the branch.

### git_show
Shows the contents (metadata and diff) of a specific commit.

**Arguments:**
- `repo` (`git.Repo`): The Git repository object.
- `revision` (`str`): The commit hash or reference to show.

**Returns:**
- `str`: A string containing the commit details and its diff.

### git_apply_diff
Applies a given diff content to the working directory of the repository.
Includes a check for successful application and generates a new diff output.
Also runs TSC if applicable after applying the diff.

**Arguments:**
- `repo` (`git.Repo`): The Git repository object.
- `diff_content` (`str`): The diff string to apply.

**Returns:**
- `str`: A string indicating the result of the diff application, including
  any new diff generated and TSC output if applicable, or an error message.

### git_read_file
Reads the content of a specified file within the repository.

**Arguments:**
- `repo` (`git.Repo`): The Git repository object.
- `file_path` (`str`): The path to the file relative to the repository's working directory.

**Returns:**
- `str`: A string containing the file's content, or an error message if the file
  is not found or cannot be read.

### _generate_diff_output
Generates a unified diff string between two versions of file content.

**Arguments:**
- `original_content` (`str`): The original content of the file.
- `new_content` (`str`): The new content of the file.
- `file_path` (`str`): The path of the file, used for diff headers.

**Returns:**
- `str`: A string containing the unified diff, or a message indicating no changes
  or that the diff was too large.

### _run_tsc_if_applicable
Runs TypeScript compiler (tsc) with --noEmit if the file has a .ts, .js, or .mjs extension.

**Arguments:**
- `repo_path` (`str`): The path to the repository's working directory.
- `file_path` (`str`): The path to the file that was modified.

**Returns:**
- `str`: A string containing the TSC output, or an empty string if TSC is not applicable.

### _search_and_replace_python_logic
Performs search and replace in a file using Python's re module.
Attempts literal search first, then falls back to regex search if no changes are made.

**Arguments:**
- `repo_path` (`str`): The path to the repository's working directory.
- `search_string` (`str`): The string or regex pattern to search for.
- `replace_string` (`str`): The string to replace matches with.
- `file_path` (`str`): The path to the file to modify.
- `ignore_case` (`bool`): If True, the search is case-insensitive.
- `start_line` (`Optional[int]`): Optional. The 1-based starting line number for the search.
- `end_line` (`Optional[int]`): Optional. The 1-based ending line number for the search.

**Returns:**
- `str`: A string indicating the result of the operation, including diff and TSC output,
  or an error message.

### search_and_replace_in_file
Searches for a string or regex pattern in a file and replaces it with another string.
Attempts to use `sed` for efficiency, falling back to Python logic if `sed` fails or makes no changes.

**Arguments:**
- `repo_path` (`str`): The path to the repository's working directory.
- `search_string` (`str`): The string or regex pattern to search for.
- `replace_string` (`str`): The string to replace matches with.
- `file_path` (`str`): The path to the file to modify.
- `ignore_case` (`bool`): If True, the search is case-insensitive.
- `start_line` (`Optional[int]`): Optional. The 1-based starting line number for the search.
- `end_line` (`Optional[int]`): Optional. The 1-based ending line number for the search.

**Returns:**
- `str`: A string indicating the result of the operation, including diff and TSC output,
  or an error message.

### write_to_file_content
Writes content to a specified file, creating it if it doesn't exist or overwriting it if it does.
Includes a check to ensure the content was written correctly and generates a diff.

**Arguments:**
- `repo_path` (`str`): The path to the repository's working directory.
- `file_path` (`str`): The path to the file to write to, relative to the repository.
- `content` (`str`): The string content to write to the file.

**Returns:**
- `str`: A string indicating the success of the write operation, including diff and TSC output,
  or an error message.

### execute_custom_command
Executes a custom shell command within the specified repository path.

**Arguments:**
- `repo_path` (`str`): The path to the directory where the command should be executed.
- `command` (`str`): The shell command string to execute.

**Returns:**
- `str`: A string containing the stdout and stderr of the command, and an indication
  if the command failed.

### ai_edit_files
AI pair programming tool for making targeted code changes using Aider.
This function encapsulates the logic from aider_mcp/server.py's edit_files tool.

**Arguments:**
- `repo_path` (`str`): The absolute path to the Git repository's working directory where the AI edit should be performed.
- `message` (`str`): A detailed natural language message describing the code changes to make. Be specific about files, desired behavior, and any constraints.
- `session` (`ServerSession`): The server session object for sending progress notifications.
- `files` (`List[str]`): A list of file paths (relative to the repository root) that Aider should operate on. This argument is mandatory.
- `options` (`Optional[list[str]]`): Optional. A list of additional command-line options to pass directly to Aider (e.g., ['--model=gpt-4o', '--dirty-diff']). Each option should be a string.
- `edit_format` (`EditFormat`): Optional. The format Aider should use for edits. If not explicitly provided, the default is selected based on the model name: if the model includes 'gemini', defaults to 'diff-fenced'; if the model includes 'gpt', defaults to 'udiff'; otherwise defaults to 'diff'. Options: 'diff', 'diff-fenced', 'udiff', 'whole'.
- `aider_path` (`Optional[str]`): Optional. The path to the Aider executable. Defaults to "aider".
- `config_file` (`Optional[str]`): Optional. Path to a specific Aider configuration file.
- `env_file` (`Optional[str]`): Optional. Path to a specific .env file.

**Returns:**
- `str`: A string indicating the result of the AI edit operation, including diff and Aider output, or an error message.

### aider_status_tool
Checks the status of Aider and its environment, including installation,
configuration, and Git repository details.

**Arguments:**
- `repo_path` (`str`): The path to the repository or working directory to check.
- `check_environment` (`bool`): If True, also checks Aider configuration and Git details.
- `aider_path` (`Optional[str]`): Optional. The path to the Aider executable. Defaults to "aider".
- `config_file` (`Optional[str]`): Optional. Path to a specific Aider configuration file.

**Returns:**
- `str`: A JSON string containing the status information, or an error message.

### list_tools
Lists all available tools provided by this MCP Git server.

**Arguments:**
- None

**Returns:**
- `list[Tool]`: A list of Tool objects, each describing a callable tool with its name,
  description, and input schema.

### list_repos
Lists all Git repositories known to the MCP client.
This function leverages the client's `list_roots` capability.

**Arguments:**
- None

**Returns:**
- `Sequence[str]`: A sequence of strings, where each string is the absolute path to a Git repository.

### call_tool
Executes a requested tool based on its name and arguments.
This is the main entry point for clients to interact with the server's tools.

**Arguments:**
- `name` (`str`): The name of the tool to call (must be one of the `GitTools` enum values).
- `arguments` (`dict`): A dictionary of arguments specific to the tool being called.

**Returns:**
- `list[Content]`: A list of Content objects (typically TextContent) containing the result
  or an error message.

### handle_sse
Handles Server-Sent Events (SSE) connections from MCP clients.
Establishes a communication channel for the MCP server to send events.

**Arguments:**
- `request`: The Starlette Request object.

**Returns:**
- `Response`: A Starlette Response object for the SSE connection.

### handle_post_message
Handles incoming POST messages from MCP clients, typically used for client-to-server communication.

**Arguments:**
- `scope`: The ASGI scope dictionary.
- `receive`: The ASGI receive callable.
- `send`: The ASGI send callable.

**Returns:**
- None
