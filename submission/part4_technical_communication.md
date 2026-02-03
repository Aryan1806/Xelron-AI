# Part 4: Technical Communication

## Integrity Declaration
I declare that all written content in this assessment is my own work, created without the use of AI language models or automated writing tools. All technical analysis and documentation reflects my personal understanding and has been written in my own words.

---

## 4.1 Scenario Response

**Reviewer Question:**
"Why did you choose this specific PR over the others? What made it comprehensible to you, and what challenges do you anticipate in implementing it?"

**Response:**

I chose **PR #1061 (Feat: repo to markdown)** because it showcases a clear and valuable feature that is different from the framework’s complicated internal processes. Other PRs, like #1049, focused on fixing specific problems with unit tests that needed a strong understanding of the CI/CD environment. In contrast, PR #1061 addresses a common issue in LLM development: how to effectively feed a codebase into a model’s context window.

This PR was particularly clear to me because it uses basic computer science concepts, such as recursive file traversal and text processing, instead of specific jargon tied to the framework. My experience with Python development has given me a good grasp of the `os` and `pathlib` libraries, which play a key role in this implementation. I quickly understood the "input-process-output" flow: traverse the directory, filter out unnecessary data, and convert the remaining text into Markdown.

However, I see two major challenges in implementing this effectively. The first is managing **binary and non-text files**. A simple attempt to read a `.png` or a compiled binary as text would lead to encoding errors and crash the tool. The second is the potential for **context overflow**. Including large dependency folders, like `node_modules` or `venv`, or massive log files could create a string that exceeds the token limits of the target LLM.

To tackle these challenges, I would use a "defensive" reading strategy. I would create a strict default "ignore list" for common patterns of dependencies and binaries. Additionally, I would place the file reading logic within a `try-except` block to catch `UnicodeDecodeError`. This way, if a file cannot be decoded as UTF-8, it would be skipped with a warning log, instead of stopping the whole process. Finally, I would implement a file size check to skip any file that exceeds a reasonable limit, such as 100KB, focusing on the project structure rather than raw data volume.
