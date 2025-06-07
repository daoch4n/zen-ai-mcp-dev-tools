# Pydantic Usage in MCP Server

## Introduction
Pydantic plays a crucial role in the MCP server by providing robust input validation, data modeling, and automatic schema generation. It ensures that the data passed to the server's tools is valid and correctly typed, which enhances reliability and reduces errors. Additionally, Pydantic models are used to generate OpenAPI-like schemas for the tools, which are then used by the frontend to validate inputs and provide documentation.

## Core Implementation
The core implementation of Pydantic models is located in `server.py`. The key imports are:
```python
from pydantic import BaseModel, Field
```

## Key Components
### Model Definitions
The MCP server defines several models as subclasses of `BaseModel`. These models represent the input parameters for the various tools provided by the server. Examples include:
- `GitStatus`: Represents the input for the git_status tool
- `GitDiff`: Represents the input for the git_diff tool
- `AiEdit`: Represents the input for the ai_edit tool

Each model defines attributes with type hints and uses `Field` to provide metadata such as descriptions and examples.

### Field Metadata
The `Field` function is used to provide metadata for each attribute. This includes a description, which is used in the generated schema to document the field. For example:
```python
repo_path: str = Field(..., description="The absolute path to the Git repository's working directory")
```

### Enum Integration
For fields that have a restricted set of values, we use Python's `enum.Enum` to define the allowed values. The enum is then used as the type for the field. For example:
```python
class EditFormat(Enum):
    diff = 'diff'
    diff_fenced = 'diff-fenced'
    udiff = 'udiff'
    whole = 'whole'

edit_format: EditFormat = Field(default=EditFormat.diff, description="The format Aider should use for edits")
```

## Schema Generation
The server uses the `model_json_schema()` method of the Pydantic models to generate JSON schemas for the tools. These schemas are then exposed by the `list_tools` function (lines 1333-1435 in `server.py`) and used by the frontend to validate tool inputs and generate documentation.

## Validation Workflow
The validation workflow in the MCP server is as follows:
1. The server receives a tool call request with a JSON payload
2. The payload is parsed and validated against the corresponding Pydantic model
3. If validation fails, the server returns a 422 response with the validation errors
4. If validation succeeds, the validated data is passed to the tool implementation function

This workflow is implemented in the `call_tool` function (lines 1503-1644 in `server.py`).

## Best Practices & Conventions
When defining Pydantic models in the MCP server, we follow these best practices:
- **Descriptive Field Metadata**: Always provide a `description` in the `Field` function to document the field
- **Strict Typing**: Use Python type hints (e.g., `List[str]`, `Optional[str]`) to ensure type safety
- **Default Values**: Specify sensible default values for fields where applicable
- **Validation Logic**: Use Pydantic validators for complex rules
- **Organization**: Group related models logically within `server.py` (or separate file if growing)

## Example Snippet
```python
class AiEdit(BaseModel):
    repo_path: str = Field(..., description="Absolute path to the Git repository's working directory")
    message: str = Field(..., description="Detailed natural language message describing the code changes")
    files: List[str] = Field(..., description="List of file paths to edit, relative to repo root")
    options: Optional[List[str]] = Field(default=None, description="Additional Aider command-line options")
    edit_format: EditFormat = Field(default=EditFormat.diff, description="Edit format for Aider")
```

## Mermaid Diagram: Validation Flow
```mermaid
graph TD
    A[Receive Tool Call Request] --> B[Parse JSON Payload]
    B --> C{Valid?}
    C -->|Yes| D[Call Tool Implementation]
    C -->|No| E[Return 422 with Errors]
    D --> F[Return 200 with Result]