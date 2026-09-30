# Puregram (Telegram Android) Constitution

## Core Principles

### I. Safety & Non-Destructive Modularity
All AI agents and contributors must preserve the upstream integrity of the DrKLO/Telegram Android base. Any modifications, feature additions, or custom modules must be isolated and non-intrusive to allow seamless upstream updates and prevent regressions.

### II. Android Architecture & Performance
* **Threading**: Never block the UI thread (`AndroidUtilities.runOnUIThread`, DispatchQueue).
* **Memory Management**: Avoid memory leaks in Views, Fragments, and Bitmaps. Respect low-RAM device constraints.
* **Compatibility**: Target Android SDK 36, maintain backwards compatibility down to minimum SDK requirements.
* **Native / NDK Integrity**: Do not modify C/C++ native code (`TMessagesProj/jni`) unless explicitly specified in a dedicated spec.

### III. Security & MTProto Protocol Discipline
* Strict adherence to Telegram security guidelines and MTProto protocol requirements.
* Sensitive keys and credentials must never be committed to git; rely on `BuildVars.java` and `gradle.properties`.
* Respect user privacy, end-to-end secret chat boundaries, and data encryption.

### IV. Spec-Driven Development Workflow (Mandatory)
Every feature or significant change must follow the Spec Kit cycle:
1. **Specify (`/speckit-specify`)**: Define clear functional user requirements, edge cases, and acceptance criteria.
2. **Clarify & Plan (`/speckit-plan`)**: Outline technical steps, affected components, and risk assessments.
3. **Tasks (`/speckit-tasks`)**: Break down the plan into bite-sized, verifiable tasks.
4. **Implement (`/speckit-implement`)**: Write complete, production-ready code with zero placeholders (`// TODO`).
5. **Verify & Converge (`/speckit-converge`)**: Test and verify against the original specification.

### V. Code Quality & Clean Implementation
* Complete, compilable, and production-ready code only.
* Follow established project conventions in `TMessagesProj` (naming, UI components, themes).
* Avoid unnecessary dependencies or heavy third-party libraries.

## Governance
This constitution governs all AI-assisted and human development in Puregram. Any deviation or architectural change requires explicit documentation in the feature specification and approval.

**Version**: 1.0.0 | **Ratified**: 2026-09-30 | **Last Amended**: 2026-09-30
