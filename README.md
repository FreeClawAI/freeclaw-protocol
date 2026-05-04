# FreeClaw Protocol v1.1

**A Markdown-based protocol for structured code file delivery from AI to file systems.**

FreeClaw Protocol defines a simple, deterministic way for AI models to output multiple source code files in a format that can be automatically parsed and written to a local project.

[中文说明](README_zh.md)

---

## Overview

Large language models are excellent at generating code, but lack a standardized way to deliver multi-file outputs in a machine-readable format.

FreeClaw Protocol solves this by:

* Using **Markdown structure (AST)** instead of custom syntax
* Mapping **headings → file paths** (with optional line ranges)
* Mapping **code blocks → file contents**
* Ensuring outputs are both **human-readable** and **machine-parsable**

---

## Protocol Definition

Each file is represented by:

1. An `h2` heading containing the **relative file path** (optionally with a line range)
2. A fenced code block containing the **complete file content** (or replacement content for range mode)

---

## Syntax

### Full Replacement (default)

````markdown
## relative/path/to/file.ext
```language
complete source code
```
````

### Range Replacement

````markdown
## relative/path/to/file.ext[start,end]
```language
replacement code
```
````

---

## Range Replacement Semantics

### Range Definition

`[start,end]` refers to line numbers of the **target file before modification** (closed interval).

### Line Number Rules

1. Line numbering starts from 1
2. Line endings MUST be normalized to LF before processing
3. Line numbers refer to raw text lines, not AST nodes

### Replacement Behavior

The range `[start,end]` is **deleted entirely** and replaced by the provided code block content.

### Single Range Only

Only one range per file entry is allowed. Multiple ranges MUST be split into multiple entries.

### Invalid Cases

The following are considered invalid and the entry is ignored:
- `start > end`
- Missing file content
- Malformed range format

---

## Examples

````markdown
## js/services/extractor.js
```javascript
export function extract() {
    return [];
}
```

## js/ui/toast.js[3,4]
```javascript
    _timer: null,
```
````

---

## Core Rules

1. Each file MUST be represented by one h2 heading
2. The heading MUST contain a relative file path
3. The path MUST NOT start with / or ./
4. The path MUST NOT contain .. (no directory traversal)
5. Only forward slashes (/) are allowed
6. No extra text or annotations are allowed in the path

7. The h2 heading MUST be followed by a code block
8. Parsers MAY skip blank lines or non-structural elements before the code block
9. Only the FIRST code block after an h2 is considered valid

10. The code block MUST contain complete, non-truncated source code
11. Empty code blocks are NOT allowed

12. Each h2 corresponds to exactly one file
13. Multiple entries for the same file path ARE allowed:
    - Full replacement + range replacement can coexist
    - All entries remain valid; users choose which to save
    - Parsers MUST retain all entries without overwriting

14. Language tags in code blocks are OPTIONAL and for readability only
15. Parsers SHOULD rely on file extensions, not language tags

---

## Parsing Model

A compliant parser SHOULD:

1. Traverse all `h2` elements in the Markdown AST
2. Extract the text content as the file path (strip `[start,end]` if present)
3. If `[start,end]` exists, record the line range
4. Locate the nearest following `<pre><code>` block
5. Extract the code content
6. Write the file to the corresponding relative path

---

## Design Principles

### 1. Structure over syntax

The protocol relies on Markdown AST, not custom delimiters.

### 2. Language-agnostic

Supports any file type (JSON, CSS, Markdown, etc.) without modifying content.

### 3. Non-invasive

No metadata is embedded inside the file content.

### 4. AI-native

Optimized for how LLMs naturally generate Markdown.

### 5. Human-readable

Outputs remain readable and editable by humans.

---

## Comparison

| Approach | Issues |
|---|---|
| Inline comments (`// file:`) | Breaks JSON / non-comment languages |
| Custom delimiters | Hard for AI to follow consistently |
| JSON output | Less readable, harder for large code |
| FreeClaw Protocol | Simple, structured, robust |

---

## Use Cases

* AI code generation tools
* Browser-based AI assistants
* Code export from chat interfaces
* Automated project scaffolding
* LLM-to-IDE integrations

---

## Limitations

* Depends on Markdown rendering consistency
* Requires prompt discipline for best results
* Not optimized for binary file transfer

---

## Future Work

* Streaming support
* Multi-root workspace support
* Formal schema validation

---

## License

MIT License

---

**Code AI, structured.**
