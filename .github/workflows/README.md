# Shared reusable workflows

Workflows in this directory are **reusable** (`on: workflow_call`) and can
be called from any VA repo's own workflow files. Pin to a commit SHA so
config changes roll out deliberately, not implicitly.

## Available workflows

### `check_md_links.yml`

Runs lychee against `**/*.md`, `**/*.mdx`, `**/*.mdc` in the calling
repository using a hermetic, offline configuration. Validates that
internal relative-path links resolve to a real file on disk. External
URLs are **not** fetched and fragment anchors (`#section`) are **not**
validated — anchor checking produces too many false positives in
practice (rendered headings, generated anchors, intentional drafts) and
network checking is flaky in CI.

Why it exists: the markdown documentation in our repos (`AGENTS.md`,
`CLAUDE.md`, `agents/*.md`, layer-specific instruction files) is read by
AI coding agents at every session. A broken relative-path link silently
degrades the context an agent sees. This workflow enforces that those
docs stay hermetically sealed and self-consistent across every VA repo
using the same config.

Excluded paths: `node_modules`, `.pytest_cache`, `.venv`, `venv`, `dist`,
`build`, `.next` — covers Python and JS toolchains so callers don't
need any local config.

**Caller workflow** (drop into any consumer repo at
`.github/workflows/check_md_links.yaml`):

```yaml
name: 'Markdown Link Check'

on:
  push:
    paths:
      - '**/*.md'
      - '**/*.mdx'
      - '**/*.mdc'
      - '.github/workflows/check_md_links.yaml'

jobs:
  link-check:
    uses: VantageAnalytics/.github/.github/workflows/check_md_links.yml@<sha>
```

Replace `<sha>` with a specific commit SHA (preferred) or tag from this
repo. To pick up config changes, bump the SHA.
