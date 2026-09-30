# CodeRabbit Audit Pipeline Details (Android & Telegram)

This reference file defines the 5-stage automated audit pipeline used by `coderabbit-guard` for Puregram.

---

## Stage 1: AST-like Diff Retrieval & Layer Chunking

1. Retrieve active git changes:
   - For working directory: `git diff HEAD` or staged `git diff --cached`.
   - For specific commit: `git show <commit_hash>`.
2. Parse added/modified chunks:
   - Separate code diffs from configuration (`build.gradle`, `gradle.properties`).
   - Group modified symbols (Classes, Interfaces, Views, Fragments, Controllers, Storage).
3. Classify file layer:
   - **Storage/DB**: `org.telegram.messenger.MessagesStorage`, `org.telegram.SQLite.*`.
   - **Controller/Logic**: `MessagesController`, `MediaController`, `NotificationsController`, `UserConfig`.
   - **Network/Protocol**: `ConnectionsManager`, MTProto RPC, Native NDK bindings.
   - **UI/Presentation**: `BaseFragment`, custom Views, `Theme.java`, action bars.

---

## Stage 2: Mechanical Static Linter Pass

Execute `./gradlew lintDebug` or `./gradlew test` (or module check) via `run_command`:
- Filter output to only include errors/warnings affecting changed files.
- Treat compiler errors as immediate `[CRITICAL]` failures.
- Treat warnings (`UnusedVariable`, `UnnecessarySafeCall`, missing null guards) as input context for Subagents.

---

## Stage 3: AST Symbol Tracing & Graphify Impact Mapping

1. **AST & Caller Search (`code-index-mcp` / `ast-grep`)**:
   - For every modified function or interface, search callers and symbol usages.
   - Verify if any caller site was broken by parameter signature changes or return type modifications.
2. **Graphify Dependency Mapping (`graphify`)**:
   - Query `graphify` knowledge graph for modified file nodes.
   - Trace callers up 2 dependency levels to detect indirect side effects.

---

## Stage 4: Subagent Execution Protocol

Subagents are executed in parallel (or sequential dedicated prompt blocks) with strict non-overlapping responsibilities:
- Subagent A: Focuses exclusively on Security, MTProto Integrity, Exceptions, and Null Safety.
- Subagent B: Focuses exclusively on SQLite Database Concurrency, DispatchQueue usage, and Thread Handoff Safety.
- Subagent C: Focuses exclusively on API Contract Integrity, Signature Parity, and Strict Theme (`Theme.key_*`) Compliance.

---

## Stage 5: Noise Reduction & Severity Deduplication

Before rendering the final report:
1. Deduplicate findings across subagents.
2. Filter out subjective style nitpicks unless explicitly requested.
3. Categorize severity:
   - `[CRITICAL]`: Main thread DB access, SQL injection, hardcoded UI colors bypassing `Theme.key_*`, app crashes, memory leaks, broken contracts.
   - `[MAJOR]`: Silent exception swallowing, missing transaction on multi-step DB writes, worker thread lockups.
   - `[MINOR]`: Redundant imports, minor code duplication.
4. Output concise GitHub-flavored markdown with code diffs.
