---
description: >-
  Help for /command-specification — create or continue a specification file,
  then run the specification pipeline skill on it.
---

# Specification Pipeline — Command Help

Entry command: `/command-specification` (`.cursor/commands/command-specification.md`)

Process root: `@skill-specification-pipeline` (`.cursor/skills/skill-specification-pipeline/SKILL.md`)

The command **always** resolves a single markdown specification **file** before the skill runs. The skill **requires** that path. **Review** (`review-light`, `review-full`) is read-only on the file — findings and proposed fixes in chat; the agent edits the document only when you ask to apply fixes or when the route is fragment-capture / generator / normalizer.

Hard write boundary: the skill may create the specification parent directory during `new` and may edit only the resolved markdown specification file. It must not create, edit, delete, move, rename, format, or patch source code, assets, configs, tests, generated project files, project metadata, or any non-documentation project file.

The command also detects `USER_LANGUAGE` from your natural-language request and passes it to the skill. The skill uses that language for chat and specification body content.

---

## Invocation formats

### Continue an existing specification

```text
/command-specification continue <path-to-specification.md> [-- <work request>]
```

Examples:

```text
/command-specification continue .cursor/tasks/reward-v2/specification.md -- add fragment: daily cap in section 3.2
/command-specification continue .cursor/specs/shop-checkout.md -- full pre-implementation review
```

### Create a new specification

```text
/command-specification new <specification-title> <parent-directory> [-- <work request>]
```

The command creates `<parent-directory>/<slug-from-title>.md` with a regulation §5 section skeleton (empty sections, no invented requirements).

Examples:

```text
/command-specification new "Shop checkout flow" .cursor/specs/shop
/command-specification new "Daily rewards" .cursor/tasks/daily-rewards -- generate complete technical spec from the GDD below: ...
```

If the target file already exists, use `continue` instead of `new`.

---

## What to pass as `<work request>`

Optional text after `--` (or trailing free text) is routed by the skill:

| You might say | Typical route |
|---|---|
| "Add this rule to section 3.2" | `spec-assistant` → fragment-capture |
| "Quick sanity check" | `spec-assistant` → review-light |
| "Full pre-implementation review" | `spec-assistant` → review-full |
| "Generate complete technical spec from GDD" | `spec-generator` |
| "Normalize into implementation-ready markdown" | `spec-normalizer` |

Ambiguous intent → `spec-assistant` (at most one critical clarifying question). Details: `.cursor/skills/skill-specification-pipeline/router/router-map.md`.

---

## Direct skill use (without the command)

You may invoke `@skill-specification-pipeline` directly only if you supply the same contract:

* `SPECIFICATION_PATH` — required, existing `.md` (or create it first using the `new` rules in the command file);
* `SPECIFICATION_DIR` — parent directory of `SPECIFICATION_PATH`; created only by `new`;
* `USER_LANGUAGE` — required, human-readable user language name such as `English`, `Russian`, or `Spanish`;
* `USER_REQUEST` — optional work intent.

---

## Related docs

* modes overview: this file § "What to pass";
* router: `.cursor/skills/skill-specification-pipeline/router/router-map.md`;
* document shape: `.cursor/skills/skill-specification-pipeline/shared/specification-document-regulation.md`;
* open questions: regulation **§7.1** — unresolved `OQ-xxx` only in §11; **§7.2** — closed answers inline in body, not a “closed decisions” archive;
* system decomposition: regulation §5 + `decomposition.md` §2.1 — **tree** (L0 → L1 subsystems → nested L2 entities), not a flat entity list;
* file I/O: `.cursor/rules/rule-windows-source-files-and-encoding.mdc`.

User-facing pipeline output and specification body content use `USER_LANGUAGE`. Keep structural metadata and machine/project identifiers in English where required for navigation or implementation: `PASS-*`, `REQ-*`, `AC-*`, finding `id`, API names, file/folder names, class/method/variable names, namespaces, config keys, Unity/C# terms, and code snippets.
