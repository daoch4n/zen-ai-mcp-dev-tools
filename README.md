# ⚙️ mcp-devtools: multi-functional development tools MCP server over SSE

[![GitHub repository](https://img.shields.io/badge/GitHub-repo-blue?logo=github)](https://github.com/daoch4n/zen-ai-mcp-devtools)
[![GitHub Workflow Status](https://img.shields.io/github/actions/workflow/status/daoch4n/zen-ai-mcp-devtools/python-package.yml?branch=main)](https://github.com/daoch4n/zen-ai-mcp-devtools/actions/workflows/python-package.yml)
[![PyPI](https://img.shields.io/pypi/v/mcp-devtools)](https://pypi.org/project/mcp-devtools)

- 🔧 `mcp-devtools` offers a comprehensive suite of development tools:
- 📄 For detailed overview of all tools, their arguments, and descriptions, please refer to the [Server Module Documentation](docs/server_documentation.md) or [Available Tools section](#available-tools).
  -  🎋 Git management operations (`git_status`, `git_stage_and_commit`, `git_diff`, `git_diff_all`, `git_log`, `git_create_branch`, `git_reset` `git_checkout`, `git_show`)
  -  📁 Git file operations (`git_read_file`, `git_apply_diff`)
  -  📂 Direct file operations (`search_and_replace`, `write_to_file`)
  -  🖥️ Terminal commands execution (`execute_command`)
  -  🤖 AI-assisted file operations using [Aider](https://github.com/Aider-AI/aider) (`ai_edit` (`aider_status` for systems ready check ))
    <br> ℹ️ When using the `ai_edit` tool, please refer to the [Aider Configuration documentation](docs/aider_config.md) for detailed instructions.
- 🌐 All these functions are accessible via Server-Sent Events (SSE), making it a powerful and versatile server for various development needs.
- 🛡️ Filesystem access boundaries are maintained via passing `repo_path` to every file command, so AI assistant only has read/write access to files in the current workspace (or whatever it decides to pass as `repo_path` , make sure system prompt is solid on that part).
  <br> ⚠️ Execise extreme caution with auto-allowing `execute_command` tool or at least don't leave AI assistant unattended when doing so. MCP server won't stop assistant from `execute_command` rm -rf ~/*

## Use Cases

- 🌐 Use it to extend online chat-based assistants such as ChatGPT, Google Gemini or AI Studio, Perplexity, Grok, OpenRouter Chat, DeepSeek, Kagi, T3 Chat with direct access to local files, git, terminal commands execution and AI-assisted file editing capabilities via [MCP-SuperAssistant](https://github.com/srbhptl39/MCP-SuperAssistant/) or similar projects.
- 🦘 Use it to boost code editors like Cursor, Windsurf or VSCode extensions like Roo Code, Cline, Copilot or Augment with intuitive Git management and AI-assisted file editing capabilities and say goodbye to those pesky diff application failures wasting your tool calls or `Roo having trouble...` breaking your carefully engineered automation workflows. Aider seems to get diffs right!
  - For [Roo Code](https://github.com/RooCodeInc/Roo-Code), place [.roomodes](https://github.com/daoch4n/zen-ai-mcp-devtools/blob/main/.roomodes) into your repo root and Roo will pick it up as `🤖 AI Code` mode that `🪃 Orchestrator` mode can call.


## Prerequisites

```bash
pip install uv
```

## Running from pypi

```bash
uvx mcp-devtools@latest -p 1337
```
## Running from git

### Linux/macOS

```bash
git clone "https://github.com/daoch4n/zen-ai-mcp-devtools/"
cd zen-ai-mcp-devtools
./server.sh -p 1337
```

### Windows

```powershell
git clone "https://github.com/daoch4n/zen-ai-mcp-devtools/"
cd zen-ai-mcp-devtools
.\server.ps1 -p 1337
```

## AI System Prompt

```
You have development tools at your disposal. Use relevant tools from devtools MCP server for git management, file operations, and terminal access. When using any tool from devtools, always provide the current repository full current working directory path as the 'repo_path' option, do not set it to any other folder. 'repo_path' must be explicitly asked from user in beginning of conversation. When using execute_command tool, the current working directory will be set to repo_path provided. When using it for file manipulations, make sure to pass full path in the terminal command including repo_path prefix as manipulated file path.
```

## MCP Server Configuration

To integrate `mcp-devtools` with your AI assistant, add the following configuration to your MCP settings file:

```json
{
  "mcpServers": {
    "devtools": {
      "url": "http://127.0.0.1:1337/sse",
      "disabled": false,
      "alwaysAllow": [],
      "timeout": 999
    }
  }
}
```

## Known Issues and Workarounds

**Issue:**
### `write_to_file` and 💾 Direct Code Editing vs 🤖 Delegated Editing by Coding Agent

*    🔍 When using the `write_to_file` tool for direct code editing, especially with languages like JavaScript that utilize template literals (strings enclosed by backticks), you may encounter unexpected syntax errors. This issue stems from how the AI assistant generates the `content` string, where backticks and dollar signs within template literals might be incorrectly escaped with extra backslashes (`\`).

**Mitigation:** 

*    🔨 The `write_to_file` tool integrates with `tsc` (TypeScript compiler) for `.js`, `.mjs`, and `.ts` files. The output of `tsc --noEmit --allowJs` is provided as part of the tool's response. AI assistants should parse this output to detect any compiler errors and *should not proceed with further actions* if errors are reported, indicating a problem with the written code.

**Workarounds:**

*    🤖 (most reliable) Instruct your AI assistant to delegate editing files to MCP-compatible coding agent by using `ai_edit` tool instead, as it is more suitable for direct code manipulation, automatically commits changes and produces resulting diff as tool output, and let AI assistant act as task orchestrator that will write down plans and docs with `write_to_file` tool then delegate actual coding to specialized agent, get its report (diff) as tool call result, use `git_read_file` tool to double check agent's work, and manage commits and branches (`ai_edit` tool basically integrates `Aider` via some logic ported from [its MCP bridge](https://github.com/sengokudaikon/aider-mcp-server)).
*    🖥️ (if you're feeling lucky) Instruct your AI assistant to craft a terminal command to edit problematic file via `execute_command` tool.

## Available Tools


### `git_status`
- **Description:** Shows the current status of the Git working tree, including untracked, modified, and staged files.
- **Input Schema:**
  ```json
  {
    "type": "object",
    "properties": {
      "repo_path": {
        "type": "string",
        "description": "The absolute path to the Git repository's working directory."
      }
    },
    "required": [
      "repo_path"
    ]
  }
  ```


### `git_diff_all`
- **Description:** Shows all changes in the working directory, including both staged and unstaged modifications, compared to the HEAD commit. This provides a comprehensive view of all local changes.
- **Input Schema:**
  ```json
  {
    "type": "object",
    "properties": {
      "repo_path": {
        "type": "string",
        "description": "The absolute path to the Git repository's working directory."
      }
    },
    "required": [
      "repo_path"
    ]
  }
  ```


### `git_diff`
- **Description:** Shows differences between the current working directory and a specified Git target (e.g., another branch, a specific commit hash, or a tag).
- **Input Schema:**
  ```json
  {
    "type": "object",
    "properties": {
      "repo_path": {
        "type": "string",
        "description": "The absolute path to the Git repository's working directory."
      },
      "target": {
        "type": "string",
        "description": "The target (e.g., branch name, commit hash, tag) to diff against. For example, 'main', 'HEAD~1', or a full commit SHA."
      }
    },
    "required": [
      "repo_path",
      "target"
    ]
  }
  ```

### `git_stage_and_commit`
- **Description:** Stages specified files (or all changes if no files are specified) and then commits them to the repository with a given message. This creates a new commit in the Git history.
- **Input Schema:**
  ```json
  {
    "type": "object",
    "properties": {
      "repo_path": {
        "type": "string",
        "description": "The absolute path to the Git repository's working directory."
      },
      "message": {
        "type": "string",
        "description": "The commit message for the changes."
      },
      "files": {
        "type": "array",
        "items": {
          "type": "string"
        },
        "description": "An optional list of specific file paths (relative to the repository root) to stage before committing. If not provided, all changes will be staged."
      }
    },
    "required": [
      "repo_path",
      "message"
    ]
  }
  ```


### `git_reset`
- **Description:** Unstages all currently staged changes in the repository, moving them back to the working directory without discarding modifications. This is equivalent to `git reset` without arguments.
- **Input Schema:**
  ```json
  {
    "type": "object",
    "properties": {
      "repo_path": {
        "type": "string",
        "description": "The absolute path to the Git repository's working directory."
      }
    },
    "required": [
      "repo_path"
    ]
  }
  ```

### `git_log`
- **Description:** Shows the commit history for the repository, listing recent commits with their hash, author, date, and message. The number of commits can be limited.
- **Input Schema:**
  ```json
  {
    "type": "object",
    "properties": {
      "repo_path": {
        "type": "string",
        "description": "The absolute path to the Git repository's working directory."
      },
      "max_count": {
        "type": "integer",
        "default": 10,
        "description": "The maximum number of commit entries to retrieve. Defaults to 10."
      }
    },
    "required": [
      "repo_path"
    ]
  }
  ```

### `git_create_branch`
- **Description:** Creates a new Git branch with the specified name. Optionally, you can base the new branch on an existing branch or commit, otherwise it defaults to the current active branch.
- **Input Schema:**
  ```json
  {
    "type": "object",
    "properties": {
      "repo_path": {
        "type": "string",
        "description": "The absolute path to the Git repository's working directory."
      },
      "branch_name": {
        "type": "string",
        "description": "The name of the new branch to create."
      },
      "base_branch": {
        "type": "string",
        "nullable": true,
        "description": "Optional. The name of the branch or commit hash to base the new branch on. If not provided, the new branch will be based on the current active branch."
      }
    },
    "required": [
      "repo_path",
      "branch_name"
    ]
  }
  ```

### `git_checkout`
- **Description:** Switches the current active branch to the specified branch name. This updates the working directory to reflect the state of the target branch.
- **Input Schema:**
  ```json
  {
    "type": "object",
    "properties": {
      "repo_path": {
        "type": "string",
        "description": "The absolute path to the Git repository's working directory."
      },
      "branch_name": {
        "type": "string",
        "description": "The name of the branch to checkout."
      }
    },
    "required": [
      "repo_path",
      "branch_name"
    ]
  }
  ```

### `git_show`
- **Description:** Shows the metadata (author, date, message) and the diff of a specific commit. This allows inspection of changes introduced by a particular commit.
- **Input Schema:**
  ```json
  {
    "type": "object",
    "properties": {
      "repo_path": {
        "type": "string",
        "description": "The absolute path to the Git repository's working directory."
      },
      "revision": {
        "type": "string",
        "description": "The commit hash or reference (e.g., 'HEAD', 'main', 'abc1234') to show details for."
      }
    },
    "required": [
      "repo_path",
      "revision"
    ]
  }
  ```

### `git_apply_diff`
- **Description:** Applies a given diff content (in unified diff format) to the working directory of the repository. This can be used to programmatically apply patches or changes.
- **Input Schema:**
  ```json
  {
    "type": "object",
    "properties": {
      "repo_path": {
        "type": "string",
        "description": "The absolute path to the Git repository's working directory."
      },
      "diff_content": {
        "type": "string",
        "description": "The diff content string to apply to the repository. This should be in a unified diff format."
      }
    },
    "required": [
      "repo_path",
      "diff_content"
    ]
  }
  ```

### `git_read_file`
- **Description:** Reads and returns the entire content of a specified file within the Git repository's working directory. The file path must be relative to the repository root.
- **Input Schema:**
  ```json
  {
    "type": "object",
    "properties": {
      "repo_path": {
        "type": "string",
        "description": "The absolute path to the Git repository's working directory."
      },
      "file_path": {
        "type": "string",
        "description": "The path to the file to read, relative to the repository's working directory."
      }
    },
    "required": [
      "repo_path",
      "file_path"
    ]
  }
  ```


### `search_and_replace`
- **Description:** Searches for a specified string or regex pattern within a file and replaces all occurrences with a new string. Supports case-insensitive search and line-range restrictions. It attempts to use `sed` for efficiency, falling back to Python logic if `sed` fails or makes no changes.
- **Input Schema:**
  ```json
  {
    "type": "object",
    "properties": {
      "repo_path": {
        "type": "string",
        "description": "The absolute path to the Git repository's working directory."
      },
      "file_path": {
        "type": "string",
        "description": "The path to the file to modify, relative to the repository's working directory."
      },
      "search_string": {
        "type": "string",
        "description": "The string or regex pattern to search for within the file."
      },
      "replace_string": {
        "type": "string",
        "description": "The string to replace all matches of the search string with."
      },
      "ignore_case": {
        "type": "boolean",
        "default": false,
        "description": "If true, the search will be case-insensitive. Defaults to false."
      },
      "start_line": {
        "type": "integer",
        "nullable": true,
        "description": "Optional. The 1-based starting line number for the search and replace operation (inclusive). If not provided, search starts from the beginning of the file."
      },
      "end_line": {
        "type": "integer",
        "nullable": true,
        "description": "Optional. The 1-based ending line number for the search and replace operation (inclusive). If not provided, search continues to the end of the file."
      }
    },
    "required": [
      "repo_path",
      "file_path",
      "search_string",
      "replace_string"
    ]
  }
  ```

### `write_to_file`
- **Description:** Writes the provided content to a specified file within the repository. If the file does not exist, it will be created. If it exists, its content will be completely overwritten. Includes a check to ensure content was written correctly and generates a diff.
- **Input Schema:**
  ```json
  {
    "type": "object",
    "properties": {
      "repo_path": {
        "type": "string",
        "description": "The absolute path to the Git repository's working directory."
      },
      "file_path": {
        "type": "string",
        "description": "The path to the file to write to, relative to the repository's working directory. The file will be created if it doesn't exist, or overwritten if it does."
      },
      "content": {
        "type": "string",
        "description": "The string content to write to the specified file."
      }
    },
    "required": [
      "repo_path",
      "file_path",
      "content"
    ]
  }
  ```

### `execute_command`
- **Description:** Executes an arbitrary shell command within the context of the specified repository's working directory. This tool can be used for tasks not covered by other specific Git tools, such as running build scripts, linters, or other system commands.
- **Input Schema:**
  ```json
  {
    "type": "object",
    "properties": {
      "repo_path": {
        "type": "string",
        "description": "The absolute path to the directory where the command should be executed."
      },
      "command": {
        "type": "string",
        "description": "The shell command string to execute (e.g., 'ls -l', 'npm install')."
      }
    },
    "required": [
      "repo_path",
      "command"
    ]
  }
  ```

### `ai_edit`
- **Description:** AI pair programming tool for making targeted code changes using Aider. Use this tool to:
  1. Implement new features or functionality in existing code
  2. Add tests to an existing codebase
  3. Fix bugs in code
  4. Refactor or improve existing code
  5. Make structural changes across multiple files

  The tool requires:
  - A repository path where the code exists
  - A detailed message describing what changes to make. Please only describe one change per message. If you need to make multiple changes, please submit multiple requests.

  **Edit Format Selection:**
  If the `edit_format` option is not explicitly provided, the default is selected based on the model name:
  - If the model includes `gemini`, defaults to `diff-fenced`
  - If the model includes `gpt`, defaults to `udiff`
  - Otherwise, defaults to `diff`

  Best practices for messages:
  - Be specific about what files or components to modify
  - Describe the desired behavior or functionality clearly
  - Provide context about the existing codebase structure
  - Include any constraints or requirements to follow

  Examples of good messages:
  - "Add unit tests for the Customer class in src/models/customer.rb testing the validation logic"
  - "Implement pagination for the user listing API in the controllers/users_controller.js file"
  - "Fix the bug in utils/date_formatter.py where dates before 1970 aren't handled correctly"
  - "Refactor the authentication middleware in middleware/auth.js to use async/await instead of callbacks"
- **Input Schema:**
  ```json
  {
    "type": "object",
    "properties": {
      "repo_path": {
        "type": "string",
        "description": "The absolute path to the Git repository's working directory where the AI edit should be performed."
      },
      "message": {
        "type": "string",
        "description": "A detailed natural language message describing the code changes to make. Be specific about files, desired behavior, and any constraints."
      },
      "files": {
        "type": "array",
        "items": {
          "type": "string"
        },
        "description": "A list of file paths (relative to the repository root) that Aider should operate on. This argument is mandatory."
      },
      "options": {
        "type": "array",
        "items": {
          "type": "string"
        },
        "description": "Optional. A list of additional command-line options to pass directly to Aider (e.g., ['--model=gpt-4o', '--dirty-diff']). Each option should be a string."
      },
      "edit_format": {
        "type": "string",
        "enum": [
          "diff",
          "diff-fenced",
          "udiff",
          "whole"
        ],
        "default": "diff",
        "description": "Optional. The format Aider should use for edits. Defaults to 'diff'. Options: 'diff', 'diff-fenced', 'udiff', 'whole'."
      }
    },
    "required": [
      "repo_path",
      "message",
      "files"
    ]
  }
  ```

### `aider_status`
- **Description:** Check the status of Aider and its environment. Use this to:
  1. Verify Aider is correctly installed
  2. Check API keys for OpenAI/Anthropic are set up
  3. View the current configuration
  4. Diagnose connection or setup issues
- **Input Schema:**
  ```json
  {
    "type": "object",
    "properties": {
      "repo_path": {
        "type": "string",
        "description": "The absolute path to the Git repository or working directory to check Aider's status within."
      },
      "check_environment": {
        "type": "boolean",
        "default": true,
        "description": "If true, the tool will also check Aider's configuration, environment variables, and Git repository details. Defaults to true."
      }
    },
    "required": [
      "repo_path"
    ]
  }
