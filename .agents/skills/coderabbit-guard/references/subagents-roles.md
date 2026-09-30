# Specialized Subagent Roles & Prompts (Android & Telegram)

`coderabbit-guard` uses 3 specialized reviewer roles to audit code diffs with maximum depth and zero blind spots.

---

## Subagent A: Security, MTProto Integrity & Storage Safety

**Core Mission**: Find silent failure points, unhandled boundary cases, MTProto integrity issues, and security/data leak risks.

**Strict Audit Checklist**:
1. **Error Swallowing**:
   - Flag any `catch (Exception e) {}` that suppresses exceptions without logging, reporting, or graceful fallback.
   - Flag returning dummy empty objects when the caller expects an explicit error or valid response.
2. **Null & Collection Safety**:
   - Check if array/collection access assumes non-empty state without checking bounds or size.
   - Guard against `NullPointerException` on unverified Java object references.
3. **MTProto & Network Politeness**:
   - Ensure MTProto TL RPC requests are routed properly through `ConnectionsManager`.
   - Ensure network responses and serialization errors are handled gracefully without application crashes.
4. **Data Leaks & Credentials**:
   - Ensure sensitive tokens, API hashes, or phone numbers are never printed via `Log.e()` or debug prints in production release paths.
   - Verify secrets are read from `BuildVars.java` or `gradle.properties`.
5. **SQL Injection & Parameterized Bindings**:
   - Flag any SQLite query using raw string interpolation instead of `SQLitePreparedStatement` parameterized bindings (`?`).

---

## Subagent B: Concurrency, SQLite DB & UI Thread Safety

**Core Mission**: Detect race conditions, UI freezing, thread collisions, and memory leaks.

**Strict Audit Checklist**:
1. **Thread Safety & DispatchQueue**:
   - Flag any `MessagesStorage` SQLite query or transaction called on the Android Main/UI thread.
   - Verify that storage operations execute on dedicated worker dispatch queues (`storageQueue`).
   - Verify UI updates are dispatched back via `AndroidUtilities.runOnUIThread()`.
2. **Memory Leaks & Fragment Lifecycles**:
   - Flag keeping static references to `Activity` or `Context`.
   - Ensure listeners, observers, and bitmap drawables are released in `onFragmentDestroy()`.
3. **Database Transactions & Atomicity**:
   - Verify multi-step writes in `MessagesStorage` are wrapped inside database transactions (`database.beginTransaction()`, `commitTransaction()`).
4. **Preventing Rapid Duplicate Action Race Conditions**:
   - Check if action buttons (e.g. Send, Delete, Forward, Clear History) guard against rapid duplicate clicks while async requests are executing.
5. **Mandatory Red-Teaming Audit (3 Failure Scenarios)**:
   - *Scenario 1 (Race Conditions / Rapid Inputs)*: Audit rapid overlapping taps or out-of-order network responses.
   - *Scenario 2 (Network / Connection Timeout)*: Evaluate handling when network disconnects during large file upload/download or RPC request.
   - *Scenario 3 (Database vs Memory Desync)*: Check if memory state updates reflect local SQLite changes accurately.

---

## Subagent C: Contracts, Theme System & Arabic RTL Parity

**Core Mission**: Enforce interface contract integrity, strict Telegram theme token compliance (`Theme.key_*`), and RTL layout parity.

**Strict Audit Checklist**:
1. **Strict Design System Compliance (Zero Hardcoded Hex Colors)**:
   - **MANDATORY**: Flag ANY hardcoded hex color (e.g. `0xff...` or `#aabbcc`) in UI Views or Fragments.
   - Verify that all visual elements use `Theme.getColor(Theme.key_...)` or `Theme.key_*` constants so dynamic theme switching works seamlessly.
2. **Call Site & Signature Matching**:
   - For every modified method or interface, verify that ALL callers across `TMessagesProj` match the updated signature.
3. **Arabic & RTL Parity**:
   - Verify layout direction support and proper typography scaling using `AndroidUtilities.dp()`.
   - Use start/end padding and gravity rather than hardcoded left/right.
