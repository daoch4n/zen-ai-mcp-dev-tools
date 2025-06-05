# mcp-devtools: multi-functional development tools MCP server over SSE

[![GitHub repository](https://img.shields.io/badge/GitHub-repo-blue?logo=github)](https://github.com/daoch4n/zen-ai-mcp-devtools)
[![GitHub Workflow Status](https://img.shields.io/github/actions/workflow/status/daoch4n/zen-ai-mcp-devtools/python-package.yml?branch=main)](https://github.com/daoch4n/zen-ai-mcp-devtools/actions/workflows/python-package.yml)
[![PyPI](https://img.shields.io/pypi/v/mcp-devtools)](https://pypi.org/project/mcp-devtools)

- `mcp-devtools` offers a comprehensive suite of development tools, including extensive Git operations
  -  (`git_status`, `git_diff`, `git_commit`, `git_add`, `git_reset`, `git_log`, branch management, `git_checkout`, `git_show`, `git_apply_diff`, `git_read_file`, `git_stage_all`)
  -  general file manipulation (`search_and_replace`, `write_to_file`)
  -  ability to execute shell commands (`execute_command`)
- All these functionalities are accessible via Server-Sent Events (SSE), making it a powerful and versatile server for various development needs.
- Filesystem access boundaries are maintained via passing `repo_path` to every file command, so AI assistant only has read/write access to files in the current workspace (or whatever it decides to pass as `repo_path` , make sure system prompt is solid on that part).
- It also won't stop assistant from `execute_command` rm -rf ~/* , so execise extreme caution with auto-allowing command execution tool or at least don't leave assistant unattended when doing so.

## Prerequisites

```bash
pip install uv
```

## Running with uvx (Recommended)

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

## Integration

`mcp-devtools` is designed to be used in conjunction with [MCP-SuperAssistant](https://github.com/srbhptl39/MCP-SuperAssistant/) or similar projects to extend online chat-based assistants such as ChatGPT, Google Gemini, Perplexity, Grok, Google AI Studio, OpenRouter Chat, DeepSeek, Kagi, T3 Chat with direct access to local files, git and cli tools.

## MCP Server Configuration Example

To integrate `mcp-devtools` with your AI assistant, add the following configuration to your MCP settings file:

```json
{
  "mcpServers": {
    "devtools": {
      "url": "http://127.0.0.1:1337/sse",
      "disabled": false,
      "alwaysAllow": []
    }
  }
}
```

## Aider Integration Environment Variables

When using the `ai_edit` tool (which leverages `aider`), the following environment variables can be set to configure `aider`'s behavior. These are optional; if not set, `aider` will use its default behavior or the defaults configured within `mcp-devtools`.

For a detailed explanation of Aider's configuration hierarchy and how settings are applied, please refer to the [Aider Configuration documentation](docs/aider_config.md).

- `OPENAI_API_KEY`: Your OpenAI API key. Required for accessing OpenAI models.
- `OPENAI_API_BASE`: (Optional) The base URL for the OpenAI API, useful for custom endpoints or proxying.
- `AIDER_MODEL`: (Optional) Specifies the model to use with `aider`. If unset, `mcp-devtools` defaults to `gemini-2.5-flash-preview-05-20-refined-high`.

## Known Issues and Workarounds

**Issue:**
### `write_to_file` and 💾 Direct Code Editing vs 🤖 Delegated Editing by Coding Agent

*    🔍 When using the `write_to_file` tool for direct code editing, especially with languages like JavaScript that utilize template literals (strings enclosed by backticks), you may encounter unexpected syntax errors. This issue stems from how the AI assistant generates the `content` string, where backticks and dollar signs within template literals might be incorrectly escaped with extra backslashes (`\`).

**Mitigation:** 

*    🔨 The `write_to_file` tool integrates with `tsc` (TypeScript compiler) for `.js`, `.mjs`, and `.ts` files. The output of `tsc --noEmit --allowJs` is provided as part of the tool's response. AI assistants should parse this output to detect any compiler errors and *should not proceed with further actions* if errors are reported, indicating a problem with the written code.

**Workarounds:**

*    🤖 (most reliable) Instruct your AI assistant to delegate editing files to MCP-compatible coding agent by adding it as another MCP server, as it is more suitable for direct code manipulation, and let AI assistant act as task orchestrator that will write down plans and docs with `write_to_file` and delegate coding to specialized agent, then use `git_read_file` or `git_diff` to check up on agent's work, and manage commits and branches ([Aider](https://github.com/Aider-AI/aider) via [its MCP bridge](https://github.com/sengokudaikon/aider-mcp-server) is already integrated as `ai_edit_files` tool).
*    🖥️ (if you're feeling lucky) Instruct your AI assistant to craft a terminal command to edit problematic file via `execute_command` tool.

## Available Tools

### `git_status`
- **Description:** Shows the working tree status.
- **Input Schema:**
  ```json
  {
    "type": "object",
    "properties": {
      "repo_path": {
        "type": "string"
      }
    },
    "required": [
      "repo_path"
    ]
  }
  ```

### `git_diff_unstaged`
- **Description:** Shows changes in the working directory that are not yet staged.
- **Input Schema:**
  ```json
  {
    "type": "object",
    "properties": {
      "repo_path": {
        "type": "string"
      }
    },
    "required": [
      "repo_path"
    ]
  }
  ```

### `git_diff_staged`
- **Description:** Shows changes that are staged for commit.
- **Input Schema:**
  ```json
  {
    "type": "object",
    "properties": {
      "repo_path": {
        "type": "string"
      }
    },
    "required": [
      "repo_path"
    ]
  }
  ```

### `git_diff`
- **Description:** Shows differences between branches or commits.
- **Input Schema:**
  ```json
  {
    "type": "object",
    "properties": {
      "repo_path": {
        "type": "string"
      },
      "target": {
        "type": "string"
      }
    },
    "required": [
      "repo_path",
      "target"
    ]
  }
  ```

### `git_commit`
- **Description:** Records changes to the repository.
- **Input Schema:**
  ```json
  {
    "type": "object",
    "properties": {
      "repo_path": {
        "type": "string"
      },
      "message": {
        "type": "string"
      }
    },
    "required": [
      "repo_path",
      "message"
    ]
  }
  ```

### `git_add`
- **Description:** Adds file contents to the staging area.
- **Input Schema:**
  ```json
  {
    "type": "object",
    "properties": {
      "repo_path": {
        "type": "string"
      },
      "files": {
        "type": "array",
        "items": {
          "type": "string"
        }
      }
    },
    "required": [
      "repo_path",
      "files"
    ]
  }
  ```

### `git_reset`
- **Description:** Unstages all staged changes.
- **Input Schema:**
  ```json
  {
    "type": "object",
    "properties": {
      "repo_path": {
        "type": "string"
      }
    },
    "required": [
      "repo_path"
    ]
  }
  ```

### `git_log`
- **Description:** Shows the commit logs.
- **Input Schema:**
  ```json
  {
    "type": "object",
    "properties": {
      "repo_path": {
        "type": "string"
      },
      "max_count": {
        "type": "integer",
        "default": 10
      }
    },
    "required": [
      "repo_path"
    ]
  }
  ```

### `git_create_branch`
- **Description:** Creates a new branch from an optional base branch.
- **Input Schema:**
  ```json
  {
    "type": "object",
    "properties": {
      "repo_path": {
        "type": "string"
      },
      "branch_name": {
        "type": "string"
      },
      "base_branch": {
        "type": "string",
        "nullable": true
      }
    },
    "required": [
      "repo_path",
      "branch_name"
    ]
  }
  ```

### `git_checkout`
- **Description:** Switches branches.
- **Input Schema:**
  ```json
  {
    "type": "object",
    "properties": {
      "repo_path": {
        "type": "string"
      },
      "branch_name": {
        "type": "string"
      }
    },
    "required": [
      "repo_path",
      "branch_name"
    ]
  }
  ```

### `git_show`
- **Description:** Shows the contents of a commit.
- **Input Schema:**
  ```json
  {
    "type": "object",
    "properties": {
      "repo_path": {
        "type": "string"
      },
      "revision": {
        "type": "string"
      }
    },
    "required": [
      "repo_path",
      "revision"
    ]
  }
  ```

### `git_apply_diff`
- **Description:** Applies a diff to the working directory. Also outputs a diff of the changes made after successful application and `tsc --noEmit --allowJs` output for `.js`, `.mjs`, and `.ts` files to facilitate clean edits.
- **Input Schema:**
  ```json
  {
    "type": "object",
    "properties": {
      "repo_path": {
        "type": "string"
      },
      "diff_content": {
        "type": "string"
      }
    },
    "required": [
      "repo_path",
      "diff_content"
    ]
  }
  ```

### `git_read_file`
- **Description:** Reads the content of a file in the repository.
- **Input Schema:**
  ```json
  {
    "type": "object",
    "properties": {
      "repo_path": {
        "type": "string"
      },
      "file_path": {
        "type": "string"
      }
    },
    "required": [
      "repo_path",
      "file_path"
    ]
  }
  ```

### `git_stage_all`
- **Description:** Stages all changes in the working directory.
- **Input Schema:**
  ```json
  {
    "type": "object",
    "properties": {
      "repo_path": {
        "type": "string"
      }
    },
    "required": [
      "repo_path"
    ]
  }
  ```

### `search_and_replace`
- **Description:** Searches for a string or regex pattern in a file and replaces it with another string. It first attempts to use `sed` for the replacement. If `sed` fails or makes no changes, it falls back to a Python-based logic that first attempts a literal search and then a regex search if no literal matches are found. Also outputs a diff of the changes made after successful replacement and `tsc --noEmit --allowJs` output for `.js`, `.mjs`, and `.ts` files to facilitate clean edits.
- **Input Schema:**
  ```json
  {
    "type": "object",
    "properties": {
      "repo_path": {
        "type": "string"
      },
      "file_path": {
        "type": "string"
      },
      "search_string": {
        "type": "string"
      },
      "replace_string": {
        "type": "string"
      },
      "ignore_case": {
        "type": "boolean",
        "default": false
      },
      "start_line": {
        "type": "integer",
        "nullable": true
      },
      "end_line": {
        "type": "integer",
        "nullable": true
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
- **Description:** Writes content to a specified file, creating it if it doesn't exist or overwriting it if it does. Also outputs a diff of the changes made after successful write and `tsc --noemit --allowJs` output for `.js` `.mjs` `.ts` files to facilitate clean edits.
- **Input Schema:**
  ```json
  {
    "type": "object",
    "properties": {
      "repo_path": {
        "type": "string"
      },
      "file_path": {
        "type": "string"
      },
      "content": {
        "type": "string"
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
- **Description:** Executes a custom shell command. The `repo_path` parameter is used to set the current working directory (cwd) for the executed command.
- **Input Schema:**
  ```json
  {
    "type": "object",
    "properties": {
      "repo_path": {
        "type": "string"
      },
      "command": {
        "type": "string"
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
        "type": "string"
      },
      "message": {
        "type": "string"
      },
      "options": {
        "type": "array",
        "items": {
          "type": "string"
        },
        "nullable": true
      }
    },
    "required": [
      "repo_path",
      "message"
    ]
  }
  ```

### `aider_status`
- **Description:** Check the status of Aider and its environment. Use this to:
  1. Verify Aider is correctly installed
  2. Check that API keys are set up
  3. View the current configuration
  4. Diagnose connection or setup issues
- **Input Schema:**
  ```json
  {
    "type": "object",
    "properties": {
      "repo_path": {
        "type": "string"
      },
      "check_environment": {
        "type": "boolean",
        "default": true
      }
    },
    "required": [
      "repo_path"
    ]
  }
