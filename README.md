# A simple Git MCP server based on [mcp-server-git](https://pypi.org/project/mcp-server-git) , adapted to run over SSE protocol

## Usage
- Run it with `./server.sh` , it will handle installation and such, adjust port if needed

## Prompting 
- When using Git MCP over SSE, dont forget to include this in your AI assistant system prompt:
  - `Always use Git through MCP server, when using any tool from it, you need to always pass full cwd path as repo_path option`
