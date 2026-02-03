# Part 3: Prompt Preparation

## Integrity Declaration
I declare that all written content in this assessment is my own work, created without the use of AI language models or automated writing tools. All technical analysis and documentation reflects my personal understanding and has been written in my own words.

---

## 3.1 Comprehensive Prompt Documentation

### 3.1.1 Repository Context

**MetaGPT** is a multi-agent framework designed to bridge the gap between Large Language Models (LLMs) and complex software engineering tasks. It assign different roles to GPTs to form a collaborative software entity for complex tasks. MetaGPT takes a one line requirement as input and outputs user stories / competitive analysis / requirements / data structures / APIs / documents, etc. Internally, MetaGPT includes product managers / architects / project managers / engineers. It provides the entire process of a software company along with carefully orchestrated SOPs.

Each role has a specific set of permissions and limits, sharing a common environment or memory. For example, an Architect agent reviews a Product Requirement Document (PRD) created by a Product Manager and produces a technical design. An Engineer then uses this design to write code. This organized approach helps reduce the confusion and mistakes often seen when a single LLM handles large projects.

The intended users of this repository are software developers, AI researchers, and automation engineers who want to quickly prototype applications or automate parts of their development workflow. It addresses the problem of context retention and process management in generative AI. By enforcing a strict workflow (SOP), MetaGPT makes sure that the output is not just a piece of code, but a complete, compilable software project.

### 3.1.2 Pull Request Description

**Selected PR:** [Feat: repo to markdown #1061](https://github.com/FoundationAgents/MetaGPT/pull/1061)

This pull request aims to change the entire repository into a single, structured markdown file. It also includes a test file to check whether the markdown generation and repository traversal function correctly. The main change involves a new Python module, `metagpt/utils/repo_to_markdown.py`, which contains the `RepoToMarkDown` class.

**Why these changes are needed:**
In real-world software engineering, developers rarely begin with a blank slate. They often need to read, debug, or refactor existing legacy code. LLM agents encounter a big challenge here. They cannot easily "browse" a file system like humans can. To reason well about a repository, the agent needs the whole file structure and content in its context window. This PR addresses that issue by turning a directory tree into a single, well-structured Markdown string.

**Previous vs. New Behavior:**
Previously, if an agent needed to analyze a repository, it had to request file reads one by one. This caused too many API calls, a lot of time, and scattered context. There was no built-in tool to snapshot a codebase. With the new behavior, a single function call to `get_repo_content` lets the agent take in the entire relevant part of a codebase, excluding unnecessary files like `.git` or compiled files, in a format that LLMs can easily understand (Markdown).

### 3.1.3 Acceptance Criteria

To define this implementation as successful, the system must satisfy the following criteria:

1.  ✓ **Recursive Traversal:** Given a root directory, the system must visit every subdirectory and identify all files. It should ensure no nested files are missed.  
2. ✓ **Noise Filtering:** The system must use a predefined "ignore list" that includes `.git`, `__pycache__`, `node_modules`, and `.png`. This will keep binary and metadata files out of the final output string.
3.  ✓ **Markdown Formatting:** The output string must format each file clearly. It should use Markdown headers for paths, like `## File: src/main.py`, and triple-backticks for code content. This helps the LLM tell the difference between filename and file content.  
4. ✓ **Language Detection:** The system must automatically detect the programming language based on the file extension. For example, it maps `.py` to python and `.js` to javascript. It applies the correct language tag to the Markdown code block.
5. ✓ **Robustness:** The function must return a valid string even when the repository is empty or contains only ignored files. It should not crash the main application thread.

### 3.1.4 Edge Cases

The prompt must clearly instruct the model to handle these situations:

1.  **Binary/Non-Text Files:** The directory may have non-text files that are not on the ignore list, such as a random `.exe` or complex binary data. The system must recognize `UnicodeDecodeError` and skip these files smoothly instead of crashing.
2.  **Encoding Conflicts:** Legacy files might be encoded in ISO-8859-1 or Windows-1252 instead of UTF-8. The implementation should try to decode common encodings or use a replacement strategy (`errors="replace"`) to avoid losing data.
3.  **Large Repository Size:** While the utility converts files to strings, processing a very large repository could cause memory spikes. The implementation should use generators, like `os.walk`, instead of loading the whole file tree into memory before processing.

### 3.1.5 Initial Prompt

**Context:**  
You are a Senior Python Engineer working on the **MetaGPT** framework, an open-source tool for software development. We are adding a new feature that allows our agents, including the Data Interpreter and Architect roles, to "read" an entire local code repository at once.

**Task:**  
Create a new utility module `metagpt/utils/repo_to_markdown.py` with a class `RepoToMarkDown`. This class will go through a local directory and change all valid code files into a single Markdown string.

**Technical Requirements:**

1. **Class Definition:** Define `RepoToMarkDown`. It should ideally be initialized without arguments, but feel free to include an `__init__` method if you need to set up default ignore patterns.
2. **Ignore Logic:**
   * Create a method to filter out unwanted files and folders.
   * **Must Ignore:** The following directories: `.git`, `.idea`, `.vscode`, `__pycache__`, `node_modules`.
   * **Must Ignore:** The following file extensions: `.png`, `.jpg`, `.jpeg`, `.gif`, `.svg`, `.pyc`, `.class`.
3. **Core Functionality:** Implement `get_repo_content(self, file_path: str) -> str`.
   * This method takes a path, either absolute or relative.
   * It will use `os.walk` or `pathlib` to go through the directory.
   * For each valid file found, read its content.
4. **Output Format:**
   * The output must be one single string.
   * For each file, add a header: `## File: {relative_path_from_root}`.
   * Follow the header with the file content in a Markdown code block.
   * **Important:** Identify the file language by its extension. For example, .py becomes python, and include it in the opening backticks (for example, \`\`\`python).
5. **Error Handling:**
   * Wrap file reading in a `try/except` block.
   * Specifically catch `UnicodeDecodeError` to deal with binary files that might bypass the extension filter. If a file cannot be read as text, skip it.

**Testing Requirements:**  
Create a companion test file `tests/metagpt/utils/test_repo_to_markdown.py`.  
* Use `pytest`.  
* Set up a temporary directory structure in the test setup.  
* Include one valid Python file, one ignored directory (like `.git`), and one ignored file type (like `.png`).  
* Check that the returned string includes the Python code.  
* Check that the returned string does not include the git data or the image data.
