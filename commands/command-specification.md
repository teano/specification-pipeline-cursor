---
description: >-
  Standalone specification pipeline entrypoint. Resolves or creates the
  specification markdown file, then runs skill-specification-pipeline on it.
---

Read and execute `@skill-specification-pipeline` as the only process root for this request.

## 1. Parse `$ARGUMENTS` (mandatory before the skill)

The command does **not** run until a concrete specification **file** path is resolved.

### 1.1 Continue an existing specification

```text
continue <path-to-specification.md> [-- <free-text user request>]
```

Rules:

1. `<path-to-specification.md>` must be repo-relative or absolute, end with `.md`, and exist on disk.
2. If the file is missing or not readable, stop and ask the user for a valid path (do not invent a path).
3. Set `SPECIFICATION_DIR` to the parent directory of the resolved `SPECIFICATION_PATH`.

### 1.2 Create a new specification

```text
new <specification-title> <parent-directory> [-- <free-text user request>]
```

Rules:

1. `<specification-title>` is the human title (used for the document `#` heading and for the filename slug).
2. `<parent-directory>` is the folder that will contain the file (repo-relative or absolute). Create the directory if it does not exist.
3. Derive `SPECIFICATION_SLUG` from the title: lowercase; trim; replace spaces/underscores with `-`; keep only `[a-z0-9-]`; collapse repeated `-`; trim leading/trailing `-`. If the slug is empty, stop and ask for a valid title.
4. Set `SPECIFICATION_DIR = <parent-directory>`.
5. Set `SPECIFICATION_PATH = <parent-directory>/<SPECIFICATION_SLUG>.md` (normalize path separators for the OS).
6. If `SPECIFICATION_PATH` already exists, stop and tell the user to use `continue` or pick another directory/title.
7. Create the file following `.cursor/rules/rule-windows-source-files-and-encoding.mdc`:
   - create an **empty** UTF-8 file first (shell or editor), then apply_patch the starter body (LF line endings);
   - starter body: `# <specification-title>` plus the section headings from `.cursor/skills/skill-specification-pipeline/shared/specification-document-regulation.md` §5 (sections may stay empty; do not invent requirements).

### 1.3 Invalid or incomplete invocation

If `$ARGUMENTS` does not start with `new` or `continue` (or required parameters are missing), **stop** and show the formats from `.cursor/commands/command-specification-help.md`. Do not route the pipeline without `SPECIFICATION_PATH`.

Optional `--` separator: everything after `--` is `USER_REQUEST` (work intent for routing). If `--` is absent, treat the remainder after the structured parameters as `USER_REQUEST` when unambiguous; otherwise `USER_REQUEST` may be empty and the skill will ask what to do next.

## 2. Determine user language

Before handing off to the skill, determine `USER_LANGUAGE` from the user's natural-language request.

Rules:

1. Use the dominant natural language in `USER_REQUEST`; if `USER_REQUEST` is empty, use the surrounding user message or the new specification title.
2. For `continue` with no usable request text, inspect the existing specification body and use the dominant non-metadata natural language.
3. Ignore project naming when detecting language: API names, file/folder names, class/method/variable names, Unity/C# terms, IDs, and code snippets do not decide `USER_LANGUAGE`.
4. If the language cannot be determined confidently, stop and ask the user to specify the language; do not invoke the skill without `USER_LANGUAGE`.
5. Store the value as a human-readable language name, for example `English`, `Russian`, or `Spanish`.

## 3. Hand off to the skill

Pass these bindings into `@skill-specification-pipeline` (the skill treats them as mandatory):

| Binding | Value |
|---|---|
| `SPECIFICATION_PATH` | resolved markdown file (create or continue) |
| `SPECIFICATION_DIR` | parent directory of `SPECIFICATION_PATH` |
| `SPECIFICATION_TITLE` | `#` title from the file if present; else `<specification-title>` for `new` |
| `USER_REQUEST` | text after `--` or trailing free text |
| `USER_LANGUAGE` | detected user language from §2 |

Then, inside the skill only:

1. route via `.cursor/skills/skill-specification-pipeline/router/router-map.md` using `USER_REQUEST` (and the spec file as context);
2. enforce guards from `.cursor/skills/skill-specification-pipeline/policies/mode-transition-guards.md`;
3. apply shared policies/principles and pass-loading policy;
4. run the selected mode workflow — spec **edits** go to `SPECIFICATION_PATH` when the routed mode/submode allows writes (`spec-generator`, `spec-normalizer`, `fragment-capture`); during `new`, the command may create `SPECIFICATION_DIR`; source code, assets, configs, tests, generated project files, project metadata, and any non-documentation project files are never write targets; **review-light** and **review-full** are read-only on that file until the user explicitly asks to apply fixes.

User request (`$ARGUMENTS`):

$ARGUMENTS
