---
name: skill-specification-pipeline
description: >-
  Standalone specification workflow for Cursor with deterministic routing across
  assistant, generator, and normalizer modes. Requires SPECIFICATION_PATH (markdown
  file to read; write per routed submode) and USER_LANGUAGE (detected user language).
  Uses shared regulation, core principles, atomic passes, review profiles, and mode
  transition guards. Strict documentation-only write scope: never mutates source
  code or project files. Entry command: command-specification.
---

# Skill: Specification Pipeline

## 1. Purpose

This skill owns:

1. routing across specification-work modes;
2. shared regulation and cross-mode principles;
3. deterministic pass/profile loading;
4. transition guardrails between modes;
5. stable entry via the command help document;
6. a strict documentation-only write boundary: no source code or project-file mutations.

---

## 2. Mandatory runtime inputs

Every run **must** receive a resolved `SPECIFICATION_PATH` and `USER_LANGUAGE` before routing or mode work.

| Source | How `SPECIFICATION_PATH` is set |
|---|---|
| `/command-specification` | Command layer: `new` (create file) or `continue` (existing file); see `.cursor/commands/command-specification.md` |
| Direct `@skill-specification-pipeline` | User must supply an existing `.md` path, or explicit `new` with title + parent directory using the same rules as the command |

| Binding | Requirement |
|---|---|
| `SPECIFICATION_PATH` | resolved readable `.md` file |
| `SPECIFICATION_DIR` | parent directory of `SPECIFICATION_PATH`; may be created only by `new` |
| `USER_LANGUAGE` | required human-readable natural language name detected from the user's request, e.g. `English`, `Russian`, `Spanish` |
| `USER_REQUEST` | optional work intent |
| `SPECIFICATION_TITLE` | optional document `#` heading; default from file front matter / first `#` line |

Hard rules:

1. If `SPECIFICATION_PATH` is missing, not a `.md` file, or not readable — **stop**; do not run passes or modes.
2. If `USER_LANGUAGE` is missing or ambiguous — **stop** and ask for the language; do not run passes or modes.
3. Filesystem writes are limited to creating `SPECIFICATION_DIR` during `new` and creating/editing `SPECIFICATION_PATH` when the routed mode allows writes.
4. Do not create, edit, delete, move, rename, format, patch, or otherwise mutate source code, assets, configs, tests, generated project files, project metadata, or any non-documentation project file.
5. If the user requests implementation/code/project changes inside a pipeline request, stop and state that those changes require a separate non-pipeline request.
6. Another documentation target requires a separate pipeline invocation with its own `SPECIFICATION_PATH`; do not redirect writes mid-run.
7. Pass `SPECIFICATION_PATH`, `USER_LANGUAGE`, and optional `USER_REQUEST` into every selected mode wrapper; modes follow `shared/specification-document-regulation.md` for read vs write:
   - **read + write:** `spec-generator`, `spec-normalizer`, `spec-assistant` → `fragment-capture`;
   - **read-only:** `spec-assistant` → `review-light`, `review-full` — do **not** mutate `SPECIFICATION_PATH` until the user explicitly asks to apply fixes (e.g. “apply F-003”, “внеси правки”, “patch the spec”).
8. File I/O: `.cursor/rules/rule-windows-source-files-and-encoding.mdc`.

---

## 3. Required Package Layout

Use only this package as the process root:

```text
.cursor/skills/skill-specification-pipeline/
  SKILL.md
  router/router-map.md
  router/scenario-validation.md
  shared/
    specification-document-regulation.md
    source-priority-policy.md
    pass-loading-policy.md
    core-principles/
      system-thinking.md
      decomposition.md
      grounding.md
    passes/
  review-profiles/
  modes/
    spec-assistant/...
    spec-generator/...
    spec-normalizer/...
  policies/mode-transition-guards.md
```

Rules:

* only mode folders under `modes/`;
* pass semantics live only in `shared/passes/*`.

---

## 4. Runtime Flow

For each user request:

0. verify `SPECIFICATION_PATH` and `USER_LANGUAGE` (§2); read the file for context;
1. classify intent via `router/router-map.md` using `USER_REQUEST` when present, else infer from the user message;
2. apply `policies/mode-transition-guards.md`;
3. apply `shared/source-priority-policy.md`;
4. always load regulation + core principles (`system-thinking.md`, `decomposition.md`, `grounding.md`);
5. load pass/profile scope via `shared/pass-loading-policy.md`;
6. run the selected mode wrapper (`modes/spec-*/SKILL.md`);
7. aggregate findings per `shared/pass-loading-policy.md` §6 — **bare block/warning is forbidden**.

---

## 5. Pipeline Modes

| Mode | Entry |
|---|---|
| `spec-assistant` | `modes/spec-assistant/SKILL.md` |
| `spec-generator` | `modes/spec-generator/SKILL.md` |
| `spec-normalizer` | `modes/spec-normalizer/SKILL.md` |

User overview (documentation, not a runtime mode): `.cursor/commands/command-specification-help.md`.

---

## 6. DRY Contract

1. mode files must not duplicate atomic pass text;
2. review profiles reference pass IDs only;
3. mode files reference shared docs by path;
4. mode files may add activation criteria, not pass logic.

---

## 7. Hard Normalizer Gate

`spec-normalizer` must not return `Ready` unless machine addressability and traceability are confirmed.

Sources: `PASS-008`, `PASS-009`, `modes/spec-normalizer/addressability-traceability/`, `anti-weakening-readiness/`.

---

## 8. Final Instruction

Route all specification requests only through this package. Never run without `SPECIFICATION_PATH` and `USER_LANGUAGE`.

Use `USER_LANGUAGE` for user-facing chat and for specification body content. Do not translate structural specification metadata or machine/project identifiers: section structure when it is required for navigation, front matter keys, `PASS-*`, `REQ-*`, `AC-*`, finding `id`, API names, file/folder names, class/method/variable names, namespaces, config keys, Unity/C# terms, and code snippets.
