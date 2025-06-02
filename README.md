# A simple Git MCP server based on [mcp-server-git](https://pypi.org/project/mcp-server-git) , adapted to run over SSE protocol

## Prerequisites
- `pip install uv`

## Usage
- Run it with `./server.sh` , it will handle installation and server startup, adjust port if needed

## Prompting 
- When using Git MCP over SSE, dont forget to include this in your AI assistant system prompt:
  - `Always use Git through MCP server, when using any tool from it, you need to always pass full cwd path as repo_path option`

## Available Tools

This MCP server provides the following Git-related tools:

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
- **Description:** Searches for a string or regex pattern in a file and replaces it with another string.
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
- **Description:** Writes content to a specified file, creating it if it doesn't exist or overwriting it if it does.
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
