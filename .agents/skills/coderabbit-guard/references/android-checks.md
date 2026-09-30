# Android & Telegram Architecture Guardrails

This reference document outlines specific anti-patterns and failure modes unique to Android & Telegram codebase (`TMessagesProj`).

---

## 1. UI Layer & Custom Views Anti-Patterns

### Anti-Pattern 1.1: Blocking the Main (UI) Thread
- **Defect**: Performing disk I/O, heavy JSON parsing, or network calls directly on the UI thread instead of worker threads or `DispatchQueue`.
- **Impact**: UI freezes, ANR (Application Not Responding) dialogs, and dropped 60/120 FPS frames.

### Anti-Pattern 1.2: Hardcoded Colors & Theme Bypass
- **Defect**: Using raw hex color integers (e.g. `0xff...` or `#aabbcc`) directly in custom Views, Paints, or Drawables instead of `Theme.getColor(Theme.key_...)`.
- **Impact**: Violates Telegram's dynamic theme system; breaks night mode switching, custom themes, and user theme preferences.

### Anti-Pattern 1.3: Fragment & Context Memory Leaks
- **Defect**: Holding static references to `Activity`, `BaseFragment`, or `Context`, or failing to clear listeners, callbacks, and bitmap references in `onFragmentDestroy()`.
- **Impact**: Memory leaks leading to `OutOfMemoryError` (OOM), especially on low-RAM devices during repeated navigation.

### Anti-Pattern 1.4: Hardcoded Dimensions & RTL Non-Compliance
- **Defect**: Hardcoding pixel dimensions or using rigid `left`/`right` coordinates instead of `AndroidUtilities.dp()` and start/end directional alignment.
- **Impact**: Distorted UI across different screen densities and broken alignment in Arabic Right-to-Left (RTL) mode.

---

## 2. SQLite Database & Storage Anti-Patterns

### Anti-Pattern 2.1: Storage Access on Main Thread
- **Defect**: Calling `MessagesStorage` queries or accessing `SQLiteDatabase` directly from the UI thread without dispatching to `storageQueue`.
- **Impact**: UI jank, stutter, and potential database locking errors.

### Anti-Pattern 2.2: Non-Transactional Multi-Step Operations
- **Defect**: Performing multiple related database writes (e.g. inserting dialogs, messages, and media) without wrapping inside `database.beginTransaction()` and `database.commitTransaction()`.
- **Impact**: Partial failure leaves database corrupted or out-of-sync with server state if an exception occurs mid-operation.

### Anti-Pattern 2.3: Unparameterized Raw SQL Queries
- **Defect**: Executing raw SQLite queries using string interpolation (`"WHERE id = " + id`) instead of bound arguments via `SQLitePreparedStatement.bindLong()` / `bindString()`.
- **Impact**: SQL Injection vulnerability and syntax crashes on text containing apostrophes or quotes.

---

## 3. Network & MTProto Anti-Patterns

### Anti-Pattern 3.1: Direct Socket Operations Bypassing MTProto
- **Defect**: Attempting raw unencrypted HTTP/TCP network calls instead of using Telegram's MTProto RPC transport via `ConnectionsManager`.
- **Impact**: Violates security guidelines, exposes user communications, and fails to handle datacenter migration and key exchange.

### Anti-Pattern 3.2: Swallowing RPC Errors Silently
- **Defect**: Catching TL error responses without updating UI state, notifying the user, or retrying recoverable network errors.
- **Impact**: User sees a perpetual spinner or unresponsive action with zero feedback.

### Anti-Pattern 3.3: Exposing Secret Credentials
- **Defect**: Committing API hashes, phone numbers, or encryption keys directly into code rather than referencing `BuildVars.java` or `gradle.properties`.
- **Impact**: Severe security breach and violation of Telegram developer terms.
