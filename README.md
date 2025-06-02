# mcp-dev-tools: A Multi-functional Development Tools Server

This project provides a versatile MCP (Model Context Protocol) server running over the SSE (Server-Sent Events) protocol. `mcp-dev-tools` offers a comprehensive suite of development tools, including extensive Git operations (status, diff, commit, add, reset, log, branch management, checkout, show, apply diff, read file, stage all), general file manipulation (`search_and_replace`, `write_to_file`), and the ability to execute custom shell commands (`execute_command`). All these functionalities are accessible via Server-Sent Events (SSE), making it a powerful and versatile server for various development needs.

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
You have development tools at your disposal. Use relevant tools from dev-tools MCP server for git and file operations. When using any tool from it, always provide the full current working directory path as the 'repo_path' option.
```

## Integration with MCP-SuperAssistant

`mcp-dev-tools` can be used in conjunction with [MCP-SuperAssistant](https://github.com/srbhptl39/MCP-SuperAssistant/) to extend online chat-based assistants such as ChatGPT, Google Gemini, Perplexity, Grok, Google AI Studio, OpenRouter Chat, DeepSeek, Kagi, and T3 Chat.

## MCP Server Configuration Example

To integrate `mcp-dev-tools` with your AI assistant, add the following configuration to your MCP settings file:

```json
{
  "mcpServers": {
    "dev-tools": {
      "url": "http://127.0.0.1:1337/sse",
      "disabled": false,
      "alwaysAllow": [],
      "timeout": 15
    }
  }
}
```

## Available Tools

This mcp-dev-tools server provides a suite of development tools, including Git-related functionalities and general file manipulation commands, and console commands execution:

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
- **Description:** Writes content to a specified file, creating it if it doesn't exist or overwriting it if it does. Also outputs a diff of the changes made.
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
