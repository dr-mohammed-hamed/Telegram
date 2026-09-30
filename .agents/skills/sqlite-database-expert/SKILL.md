---
name: sqlite-database-expert
risk_level: MEDIUM
description: Expert in Android & Telegram SQLite embedded database development (Native Telegram SQLite, MessagesStorage, DispatchQueue, transactions, and secure offline-first persistence).
version: 2.1.0
tags: [database, sqlite, telegram, android, storage, transactions, dispatchqueue, fts]
---

# Android & Telegram SQLite Database Expert

## 1. Overview & Architectural Scope

This skill governs embedded SQLite database architecture, persistence design, and query optimization for **Android applications and Telegram codebase (`TMessagesProj`)**.

### Core Architecture in Telegram (`Puregram`)
Telegram Android does not use third-party ORMs; it communicates directly with SQLite via optimized C++ JNI bindings and native wrapper classes located in `org.telegram.SQLite`:
- `org.telegram.SQLite.SQLiteDatabase`: Native SQLite connection handle.
- `org.telegram.SQLite.SQLitePreparedStatement`: Compiled parameterized SQL statement.
- `org.telegram.SQLite.SQLiteCursor`: Forward-only cursor for reading results.
- `org.telegram.messenger.MessagesStorage`: Central coordinator for all dialogs, messages, users, and media cache, running strictly on its dedicated `DispatchQueue storageQueue`.

---

## 2. Core Architectural Patterns (Telegram SQLite Engine)

### 2.1 Parameterized Prepared Statements
Always use `SQLitePreparedStatement` with indexed parameter bindings (`?`) to prevent SQL injection and maximize SQLite query plan caching:

```java
SQLitePreparedStatement state = database.executeFast(
    "INSERT OR REPLACE INTO user_settings VALUES(?, ?, ?)"
);
state.requery();
state.bindLong(1, userId);
state.bindString(2, key);
state.bindString(3, value);
state.step();
state.dispose();
```

### 2.2 Strict Thread Concurrency via `DispatchQueue`
- **Rule**: Never access SQLite on the UI thread or arbitrary pool threads.
- All `MessagesStorage` queries must execute inside its dedicated worker queue:
```java
storageQueue.postRunnable(() -> {
    try {
        // SQLite query or transaction
    } catch (Exception e) {
        FileLog.e(e);
    }
});
```
- Marshaling data back to UI:
```java
AndroidUtilities.runOnUIThread(() -> {
    // Notify controllers or UI fragments
});
```

### 2.3 Atomic Transactions
Wrap multi-statement writes inside explicit transactions to maintain database integrity and ensure massive performance gains (WAL mode batches commits):

```java
database.beginTransaction();
try {
    // Multiple SQLite statement steps
    database.commitTransaction();
} catch (Exception e) {
    FileLog.e(e);
} finally {
    // Transaction ends
}
```

### 2.4 Cursor Iteration & Disposal
Always dispose of cursors and statements to avoid native memory leaks:

```java
SQLiteCursor cursor = database.queryFinalized("SELECT id, title FROM custom_table WHERE status = 1");
try {
    while (cursor.next()) {
        long id = cursor.longValue(0);
        String title = cursor.stringValue(1);
    }
} finally {
    cursor.dispose();
}
```

---

## 3. General Android SQLite / Room Patterns

When working with modern modular components or external libraries utilizing Android Room:
1. **Parameterized Queries**: Always use `:param` bindings in Room `@Query`.
2. **Atomic Multi-Table Operations**: Use `@Transaction` on DAO methods.
3. **Dispatchers**: Execute DAO calls on `Dispatchers.IO` or collect reactive `Flow<T>`.

---

## 4. Performance & Reliability Standards

- **WAL Mode**: Keep Write-Ahead Logging active for non-blocking concurrent reads during writes.
- **Index Selectivity**: Index columns used in `WHERE`, `JOIN`, and `ORDER BY` clauses (e.g. `mid`, `uid`, `dialog_id`).
- **Zero UI-Thread Access**: Any main thread database access causes immediate UI stutter and potential ANR.
- **Disposal Discipline**: Native SQLite statements, buffers, and cursors MUST be disposed of in `finally` blocks.
