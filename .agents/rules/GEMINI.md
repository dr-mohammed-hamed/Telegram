# Workspace Rules & Quality Standards — Puregram (Telegram Android)

Strict guidelines, architecture contracts, and quality gates for developer agents in this workspace.

---

## 1. Constitution & Architectural Foundation
* **Mandatory Session Boot Hook (إقلاع الجلسة الإلزامي)**: Before proposing any architectural plan, answering queries, or writing code in a new session, the agent **MUST** immediately inspect [.specify/memory/constitution.md](file:///d:/b/Puregram/.specify/memory/constitution.md) to ground itself in the Project Manager's decisions, Telegram architecture integrity, and single-source-of-truth theme contracts.
* **Constitution Supreme**: Read and strictly follow [.specify/memory/constitution.md](file:///d:/b/Puregram/.specify/memory/constitution.md) before research, design, or implementation.
* **Telegram Architecture (`TMessagesProj`)**:
  - `UI Layer`: `BaseFragment`, custom Views, `ActionBar`, dynamic theming.
  - `Controllers & State`: `MessagesController`, `MediaController`, `NotificationsController`, `UserConfig`.
  - `Network & Protocol`: `ConnectionsManager`, MTProto RPC, Native C++ NDK bindings.
  - `Storage Engine`: `MessagesStorage`, native `SQLiteDatabase`, dedicated `DispatchQueue` (`storageQueue`).
  - Zero heavy business logic or blocking queries inside UI Fragments or Views.
* **Design System Single Source of Truth**: All UI components MUST exclusively reference `Theme.getColor(Theme.key_...)` and `AndroidUtilities.dp(...)`. Hardcoded hex color codes anywhere in UI files are **STRICTLY FORBIDDEN**.
* **Graphify & Dependency Mapping**: Query `graphify` / `code-index-mcp` to inspect callers and dependencies before modifying or deleting existing symbols.

---

## 2. Codebase Investigation & Tool Hierarchy
* **Strict Tool Hierarchy**:
  1. `code-index-mcp` (AST / Symbol body / Semantic search) & `ast-grep` (AST search and rewrite) — **Primary tools**.
  2. `graphify` (Knowledge Graph / Impact Analysis).
  3. `grep_search` — **Strictly last resort** (only for unindexed raw strings or non-code files).
* **End-to-End Tracing (Anti-Blindspot)**: Trace complete flow across UI Views, Controllers, MTProto/Network, and MessagesStorage SQLite. Never inspect snippets in isolation.
* **Concurrency & Queue Politeness**: Respect Telegram's `DispatchQueue` model. Perform all database and storage writes strictly on background dispatch queues, and dispatch UI mutations back via `AndroidUtilities.runOnUIThread()`.

---

## 3. Platform Standards (Android / Telegram Architecture)
* **Threading & Lifecycle Safety**:
  - Never block the UI thread. Any database I/O or network RPC must run on dedicated worker threads / `DispatchQueue`.
  - Prevent memory leaks: Clear listeners, handlers, and heavy bitmap caches in `onFragmentDestroy()`.
  - Support RTL (Right-to-Left) natively across all layouts and Arabic typography.
* **Error Handling & Resilience**:
  - Wrap all file operations, database calls, and network parsing in proper `try-catch` blocks.
  - Never crash the application on unexpected server responses or network dropouts; provide graceful recovery states.

---

## 4. Mandatory Code Quality & Safety Standards
* **Strict Null Safety**: Protect against `NullPointerException` with explicit guards and verified checks.
* **Native & NDK Stability**: Do not modify C/C++ native code (`TMessagesProj/jni`) unless explicitly specified in a dedicated, approved specification.
* **Production-Ready & Zero Placeholders**: No `// TODO` or partial code stubs. Write complete, robust, production-ready code.
* **Verification Gate**: Run `./gradlew assembleDebug` (or relevant module build/test) after code changes to ensure zero compiler errors or regressions.

---

## 5. Planning, Ambiguity Gate & Execution Protocol
* **Ambiguity Gate**: If requirements have multiple interpretations or confidence is < **85%**, stop and ask targeted questions or recommend `/grill-me`. Never make silent assumptions on core features.
* **Self-Critique (Red-Teaming)**: Before presenting any plan or completing an analysis, identify at least 3 potential failure modes, race conditions, or edge cases and address them.
* **Spec Kit Workflow**: Respect the Spec Kit cycle (`/speckit-specify`, `/speckit-plan`, `/speckit-tasks`, `/speckit-implement`, `/speckit-converge`).
* **Subagent Delegation & Governance**:
  - Always grant full capabilities (`enable_write_tools: true`, `enable_subagent_tools: true`, `enable_mcp_tools: true`) when defining or invoking subagents.
  - **Zero-Improvisation Blueprint**: The Leader Agent MUST provide an exhaustive architectural spec in the executor's prompt.
  - **Leader Pre-Audit Git Diff Gate**: Before dispatching code to auditors, the Leader MUST inspect `git diff` for scope creep, debug litter (`println`), and partial implementations.
  - **Auditor Sanctity**: Never kill or terminate auditing subagents prematurely. They must run their full test matrix to completion.

---

## 6. Communication & Reporting (Caveman + PM-Focused)
* **Caveman Mode (Active on Demand)**: Compressed, direct, no filler words, no pleasantries. Preserve 100% technical precision.
* **Artifact Protocol**: Never re-summarize artifact contents in chat; point directly to the created/updated artifact file.
* **PM-Focused Verification**: Frame user updates around functionality and clear, actionable manual verification steps.