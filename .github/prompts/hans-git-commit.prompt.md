````prompt
# Git Commit Message Generator

Analyze file changes and generate structured git commit messages. Group related file changes together and create concise, clear commit messages for each group.

Analyze the nature of file changes (additions, modifications, deletions, refactoring, etc.), categorize related changes into the same commit, and generate commit messages that follow conventional format.

# Steps

1. Review all changed files and their modification contents
2. Group files based on file type, functional area, or nature of changes
3. Determine appropriate commit type for each group (feat, fix, docs, style, refactor, test, chore)
4. Generate concise commit messages, keeping each message within one line
5. Arrange commits in logical order (e.g., dependencies, importance)

# Output Format

Provide the following information for each suggested commit:
- Commit message (one line, using conventional format)
- List of included files
- Brief description of changes

```
Commit 1: [type]: [concise description]
Files: file1.js, file2.css
Description: [specific content of changes]

Commit 2: [type]: [concise description]
Files: file3.md, file4.json
Description: [specific content of changes]
```# Examples

Input file changes:
- README.md (added documentation content)
- package.json (updated dependency versions)
- src/components/Button.js (fixed button styling issues)
- src/components/Button.test.js (added button test cases)
- docs/api.md (updated API documentation)

Output:
```
Commit 1: fix: resolve button component styling display issues
Files: src/components/Button.js
Description: Fixed button styling display anomalies on specific devices

Commit 2: test: add button component test cases
Files: src/components/Button.test.js
Description: Increased unit test coverage for button component

Commit 3: docs: update project documentation and API descriptions
Files: README.md, docs/api.md
Description: Enhanced project description and updated API usage documentation

Commit 4: chore: update package dependency versions
Files: package.json
Description: Upgraded dependencies to latest stable versions
```

# Notes

- Commit messages should use English, and commit types should remain in English
- Each commit should be a logically independent unit of change
- Consider separating related test files from corresponding code files into different commits
- Documentation updates can usually be merged into the same commit
- Avoid mixing unrelated changes in the same commit