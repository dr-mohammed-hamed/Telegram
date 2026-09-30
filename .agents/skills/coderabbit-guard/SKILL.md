---
name: coderabbit-guard
description: Strict CodeRabbit-grade code reviewer & AI failure-mode detector for Android & Telegram codebase. Parses git diffs, runs static analysis, extracts AST/Graphify dependencies, audits UI lifecycle, SQLite persistence on DispatchQueue, and MTProto network safety. Use when user says "coderabbit", "review PR", "audit changes", "coderabbit-guard", "فحص الكود قبل الـ commit", or asks for deep strict AI code review.
---

# CodeRabbit Guard (Android & Telegram Agentic Reviewer)

## Overview

`coderabbit-guard` performs strict, multi-agent AI code reviews tailored for Android & Telegram codebase (`TMessagesProj`). It replicates CodeRabbit's deep inspection pipeline without external paid APIs by combining git diff chunk parsing, AST/symbol analysis (`code-index-mcp`), dependency impact mapping (`graphify`), static compiler diagnostics (`./gradlew lint`), and 3 specialized subagent reviewers.

## Workflow

When triggered (e.g. before `git commit` or when reviewing changes):

1. **Diff Retrieval & Layer Chunking**:
   - Extract staged/unstaged changes or target commit diff.
   - Categorize changed files into: Storage/Data Layer (`MessagesStorage`, native SQLite), Controller/State Layer (`MessagesController`, `MediaController`), Network Layer (`ConnectionsManager`, TL RPC), and UI Layer (`BaseFragment`, custom Views, `Theme.java`).
   - Details: See [references/audit-pipeline.md](references/audit-pipeline.md).

2. **Static Linter & Compiler Pass**:
   - Run `./gradlew lintDebug` or `./gradlew test` (or module compilation check) to capture compiler & linter warnings.

3. **Symbol Tracing & Impact Enrichment**:
   - Query `code-index-mcp` and `graphify` (if available) for modified symbol callers and transitive component impact.

4. **Parallel Subagent Audit**:
   Spawn 3 parallel subagents (or focused inspection passes) using roles defined in [references/subagents-roles.md](references/subagents-roles.md):
   - **Subagent A (Security, MTProto & Network Safety)**: Parameterized SQL bindings in `MessagesStorage`, MTProto credentials protection, exception handling, and data leaks.
   - **Subagent B (Concurrency, Thread Safety & Persistence)**: Main thread SQLite access, worker thread offloading via `DispatchQueue` (`storageQueue`), thread handoffs with `AndroidUtilities.runOnUIThread()`, and **Mandatory Red-Teaming Audit** (3 scenarios: rapid duplicate actions, network timeouts, stale memory cache).
   - **Subagent C (API Contracts & Design System Compliance)**: Method signature changes, broken call sites, and **STRICT compliance with `Theme.getColor(Theme.key_...)`** (flagging any hardcoded hex color values in UI files).

5. **Framework-Specific Guardrails**:
   Apply rules from [references/android-checks.md](references/android-checks.md) or project environment rules.

6. **Synthesis & Severity Rating**:
   Consolidate findings into a clean report:
   - `[CRITICAL]`: Must fix before commit (Main thread DB query, SQL Injection, memory leak in Fragment/Bitmap, app crash, broken contracts, hardcoded colors bypassing Theme.key_*).
   - `[MAJOR]`: Maintainability/performance risk (swallowed errors, UI freeze on worker thread, missing transaction on multi-step DB writes).
   - `[MINOR]`: Code style & minor optimization.
   Provide exact code replacement diffs for all `[CRITICAL]` and `[MAJOR]` findings.

## Output Format

Report findings using:
- Priority breakdown (`[CRITICAL]`, `[MAJOR]`, `[MINOR]`).
- Exact file path & line numbers `[file_basename](file:///path/to/file#L10-L25)`.
- Root cause explanation (why CodeRabbit would flag this).
- Drop-in git diff replacement.
