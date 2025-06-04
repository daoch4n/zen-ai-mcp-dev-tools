# mcp-devtools: multi-functional development tools mcp server

- This project provides a versatile MCP (Model Context Protocol) server running over the SSE (Server-Sent Events) protocol.
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

## Usage

### Linux/macOS

```bash
./server.sh -p 1337
```

### Windows

```powershell
.\server.ps1 -p 1337
```

## AI System Prompt

```
You have development tools at your disposal. Use relevant tools from devtools MCP server for git management, file operations, and terminal access. When using any tool from devtools, always provide the current repository full current working directory path as the 'repo_path' option, do not set it to any other folder. 'repo_path' must be explicitly asked from user in beginning of conversation.
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

## Known Issues and Workarounds

**Issue:**
### `write_to_file` and 💾 Direct Code Editing vs 🤖 Delegated Editing by Coding Agent

*    🔍 When using the `write_to_file` tool for direct code editing, especially with languages like JavaScript that utilize template literals (strings enclosed by backticks), you may encounter unexpected syntax errors. This issue stems from how the AI assistant generates the `content` string, where backticks and dollar signs within template literals might be incorrectly escaped with extra backslashes (`\`).

**Mitigation:** 

*    🔨 The `write_to_file` tool integrates with `tsc` (TypeScript compiler) for `.js`, `.mjs`, and `.ts` files. The output of `tsc --noEmit --allowJs` is provided as part of the tool's response. AI assistants should parse this output to detect any compiler errors and *should not proceed with further actions* if errors are reported, indicating a problem with the written code.

**Workarounds:**

*    🤖 (most reliable) Instruct your AI assistant to delegate editing files to MCP-compatible coding agent by adding it as another MCP server, as it is more suitable for direct code manipulation, and let AI assistant act as task orchestrator that will write down plans and docs with `write_to_file` and delegate coding to specialized agent, then use `git_read_file` to check up on agent work, and manage git operations ([Aider](https://github.com/Aider-AI/aider) via [its MCP bridge](https://github.com/daoch4n/zen-ai-mcp-aider) is a good candidate to explore).
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
- **Description:** Applies a diff to the working directory.
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
- **Description:** Searches for a string or regex pattern in a file and replaces it with another string. It first attempts a literal search and falls back to a regex search if no literal matches are found.
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
- **Description:** Executes a custom shell command within the specified repository path.
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
