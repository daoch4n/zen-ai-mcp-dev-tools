# Plan to Achieve 100% Test Coverage for `server.py`

**Goal:** Increase test coverage of `server.py` to 100% by adding comprehensive unit tests for all uncovered functions, branches, and error handling paths.

**Current Coverage:** 58% (295 missed statements out of 702)

**Strategy:**
1.  **Modular Testing:** Create new test functions or expand existing ones in `test_server.py` to target specific functions and logical branches within `server.py`.
2.  **Mocking External Dependencies:** Utilize `unittest.mock.patch` and `MagicMock`/`AsyncMock` to isolate the functions under test from external systems (e.g., Git, file system, subprocess calls, MCP server interactions).
3.  **Edge Case and Error Handling:** Explicitly write tests that trigger `try-except` blocks, `if-else` conditions, and various input scenarios to ensure all code paths are executed.
4.  **Parameterization:** Use `pytest.mark.parametrize` where applicable to test multiple inputs and expected outputs efficiently.
5.  **Temporary Files:** For file system interactions (`load_aider_config`, `load_dotenv_file`, `git_apply_diff`, `write_to_file_content`, `git_read_file`, `_search_and_replace_python_logic`, `search_and_replace_in_file`), use `tempfile` module to create and clean up test files/directories, ensuring tests are isolated and don't leave artifacts.

---

### Detailed Breakdown of Test Cases per Function/Block:

**1. `find_git_root` (Lines 43-48 missed)**
    *   **Test Cases:**
        *   Path is the root of a Git repository.
        *   Path is a subdirectory within a Git repository.
        *   Path is not part of any Git repository.
        *   Test with an empty or invalid path.

**2. `load_aider_config` (Lines 51-91 missed)**
    *   **Test Cases:**
        *   Config file exists in the working directory.
        *   Config file exists in the Git root (different from working dir).
        *   Config file specified directly.
        *   Config file exists in the home directory.
        *   Test merging logic when multiple config files exist (e.g., home, git root, workdir).
        *   No config files found.
        *   Handle `Exception` during YAML loading (e.g., malformed YAML).
        *   Test with an empty config file.

**3. `load_dotenv_file` (Lines 94-139 missed)**
    *   **Test Cases:** (Similar to `load_aider_config`)
        *   `.env` in working directory.
        *   `.env` in Git root.
        *   `.env` specified directly.
        *   `.env` in home directory.
        *   Test merging logic.
        *   No `.env` files found.
        *   Handle `ValueError` for malformed lines (e.g., `KEY_NO_VALUE`).
        *   Test with an empty `.env` file.
        *   Test with lines starting with `#`.

**4. `run_command` (Lines 142-154 missed)**
    *   **Test Cases:**
        *   Successful command execution with and without input data.
        *   Command failure (non-zero exit code).
        *   Commands with stdout and stderr.

**5. `prepare_aider_command` (Lines 161-187 missed)**
    *   **Test Cases:**
        *   Test with `True`/`False` boolean options.
        *   Test with list options.
        *   Test with string/integer value options.
        *   Test with `None` options.
        *   Test with files argument.
        *   Test a combination of all types of options and files.
        *   Test with empty base command, no files, no options.

**6. `git_apply_diff` (Lines 362-409 missed)**
    *   **Test Cases:**
        *   Successful application of a valid diff.
        *   Simulate `GitCommandError` during `repo.git.apply`.
        *   Simulate other `Exception` types.
        *   Diff content does not contain `a/` or `b/` paths.
        *   `full_affected_path.exists()` is `False`.
        *   Ensure temporary file is unlinked in `finally` block.
        *   Test when `_run_tsc_if_applicable` is called (e.g., `.ts` file).
        *   Test when `_run_tsc_if_applicable` is not called (e.g., `.py` file).

**7. `git_read_file` (Lines 419-420 missed)**
    *   **Test Cases:**
        *   Simulate `FileNotFoundError`.
        *   Simulate other `Exception` during file reading.

**8. `git_stage_all` (Lines 426-427 missed)**
    *   **Test Cases:**
        *   Simulate `git.GitCommandError`.

**9. `_generate_diff_output` (Lines 438-439, 441 missed)**
    *   **Test Cases:**
        *   Test with diff lines > 1000.
        *   Test when `diff_output` is empty.

**10. `_run_tsc_if_applicable` (Lines 446-449 missed)**
    *   **Test Cases:**
        *   Files with `.ts`, `.js`, `.mjs` extensions.
        *   Files with other extensions (should return empty string).

**11. `_search_and_replace_python_logic` (Lines 515, 530, 533-534 missed)**
    *   **Test Cases:**
        *   Simulate `FileNotFoundError`.
        *   Simulate `re.error` for invalid regex.
        *   Simulate other `Exception`.
        *   Test the `else` branch where no changes are made by either literal or regex search.

**12. `search_and_replace_in_file` (Lines 554, 559, 561, 563, 582-585, 590-594 missed)**
    *   **Test Cases:**
        *   Test `sed` with both `start_line` and `end_line`.
        *   Test `sed` with `start_line` only.
        *   Test `sed` with `end_line` only.
        *   Simulate `sed` command failing and verify fallback to Python logic.
        *   Simulate `sed` making no changes and verify fallback to Python logic.
        *   Simulate `FileNotFoundError` for the file.
        *   Simulate other `Exception` during `sed` attempt and verify fallback.

**13. `write_to_file_content` (Lines 618, 629-630 missed)**
    *   **Test Cases:**
        *   Create a new file.
        *   Simulate `written_bytes != content.encode('utf-8')`.
        *   Simulate `Exception` during file writing.

**14. `execute_custom_command` (Lines 651-652 missed)**
    *   **Test Cases:**
        *   Simulate `Exception` during subprocess creation/communication.
        *   Command executes successfully with no stdout/stderr.

**15. `ai_edit_files` (Lines 667-788 missed)**
    *   **Test Cases:**
        *   Test when `repo_path` does not exist.
        *   Simulate Aider process exiting with non-zero code.
        *   Simulate Aider process completing successfully.
        *   Simulate `process.stdin` being `None`.
        *   Simulate `Exception` during stdin write/close.
        *   Test parsing of `--option` and `--no-option`.
        *   Test parsing of `--option=value`.
        *   Verify unsupported options are removed.
        *   Ensure temporary file is unlinked and directory restored in `finally`.

**16. `aider_status_tool` (Lines 800-869 missed)**
    *   **Test Cases:**
        *   Simulate `aider --version` command failure.
        *   Simulate `find_git_root` returning `None`.
        *   Simulate `Exception` when getting git remote/branch details.
        *   Test when `check_environment` is `False`.
        *   Simulate `load_aider_config` returning empty dict.
        *   Simulate a general `Exception` during status check.

**17. `list_repos` (Lines 999, 1008-1015 missed)**
    *   **Test Cases:**
        *   Simulate `mcp_server.request_context.session` not being `ServerSession`.
        *   Simulate client not having `RootsCapability`.
        *   Simulate `git.InvalidGitRepositoryError` for a root.
        *   Test with multiple valid Git repositories.

**18. `call_tool` (Lines 1028, 1195-1227, 1250 missed)**
    *   **Test Cases:**
        *   Test the `repo_path_arg == "."` condition.
        *   Test `AI_EDIT` case, specifically ensuring `openai_api_key` and `openai_api_base` are retrieved from `os.environ` (mock environment variables for the test).
        *   Test `AIDER_STATUS` case with `check_environment=False`.
        *   The `if __name__ == "__main__":` block (line 1250) is typically not covered by unit tests.

---

### Mermaid Diagram (High-Level Test Flow):

```mermaid
graph TD
    A[Start: User requests 100% coverage] --> B{Analyze Coverage Report};
    B --> C{Identify Uncovered Code Paths in server.py};
    C --> D[Develop Detailed Test Cases for Each Path];
    D --> E[Implement Unit Tests in test_server.py];
    E --> F{Run Pytest with Coverage};
    F -- Coverage < 100% --> C;
    F -- Coverage = 100% --> G[Commit Changes];
    G --> H[Attempt Completion];