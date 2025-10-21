<!--
Sync Impact Report - Constitution Update
═════════════════════════════════════════
Version Change: INITIAL → 1.0.0
Bump Rationale: Initial constitution creation with comprehensive principles for Android widget development

Added Sections:
  ✓ I. Code Quality Standards
  ✓ II. Testing Standards (NON-NEGOTIABLE)
  ✓ III. Security & Secrets Management (NON-NEGOTIABLE)
  ✓ IV. User Experience Consistency
  ✓ V. Performance & Resource Management
  ✓ VI. Ethics & Privacy (NON-NEGOTIABLE)
  ✓ Quality Gates & CI/CD Requirements
  ✓ Release Criteria
  ✓ Development Workflow

Templates Requiring Updates:
  ✅ .specify/templates/plan-template.md - Constitution Check section aligns with new principles
  ✅ .specify/templates/spec-template.md - User scenarios align with UX/testing principles
  ✅ .specify/templates/tasks-template.md - Task categorization includes security, testing, and performance tasks

Follow-up TODOs:
  - Create pre-commit hook script for secret detection (Principle III requirement)
  - Set up CI pipeline configuration for ktlint/detekt static analysis
  - Configure dependency vulnerability scanning tool (e.g., Dependabot, OWASP)
  - Document Figma design system reference for UX consistency validation

Generated: 2025-10-21
-->

# TradeLens Constitution

## Core Principles

### I. Code Quality Standards

All code committed to TradeLens MUST meet the following quality standards:

- **Code Review**: Every pull request MUST be reviewed and approved by at least one (1) qualified reviewer before merge. Reviewers MUST verify compliance with all constitutional principles.
- **Static Analysis**: All Kotlin code MUST pass ktlint formatting checks and detekt static analysis without errors. Warnings MUST be justified or resolved.
- **Naming & Documentation**: Public APIs, classes, and non-trivial functions MUST include KDoc documentation describing purpose, parameters, return values, and exceptions.
- **Modularity**: Code MUST be organized into clear, single-responsibility modules. Widget UI logic, data fetching, storage, and API interactions MUST be separated into distinct layers.
- **Dependency Management**: All third-party dependencies MUST be declared explicitly with version pinning. Transitive dependencies MUST be audited for vulnerabilities before adoption.

**Rationale**: High code quality standards reduce bugs, improve maintainability, and enable confident refactoring. Static analysis catches common errors before runtime. Code reviews distribute knowledge and catch logic errors that tooling cannot.

### II. Testing Standards (NON-NEGOTIABLE)

TradeLens follows a test-driven development (TDD) approach with mandatory coverage targets:

- **Unit Test Coverage**: Core business logic modules MUST achieve ≥60% line coverage. Data transformation, API parsing, and widget update logic are considered core modules.
- **Test-First for Core Logic**: For new core features, unit tests MUST be written before implementation. Tests MUST fail initially, then pass after correct implementation (Red-Green-Refactor cycle).
- **Integration Tests**: API client interactions, WorkManager background jobs, and database operations MUST have integration tests validating end-to-end behavior.
- **UI Tests**: Critical user journeys (e.g., widget configuration, data refresh) SHOULD have instrumentation tests where feasible, but are not mandatory for all releases.
- **No Untested Code in Production**: Code lacking reasonable test coverage MUST NOT be merged to the main branch. Exceptions require explicit justification and approval from a maintainer.

**Rationale**: Testing catches regressions early, documents expected behavior, and enables safe refactoring. The 60% target balances thoroughness with pragmatism for a mobile widget project. Test-first development forces clear API design and prevents over-engineering.

### III. Security & Secrets Management (NON-NEGOTIABLE)

Sensitive data (API keys, tokens, credentials) MUST NEVER be committed to source control:

- **Environment Variables**: API keys and secrets MUST be passed via environment variables, Gradle build config fields, or Android Keystore.
- **Encrypted Storage**: User-provided API keys or tokens MUST be stored using Android EncryptedSharedPreferences or equivalent encrypted storage.
- **Pre-Commit Hook**: A pre-commit hook MUST be configured to scan for accidental secrets (e.g., using detect-secrets or git-secrets). Commits containing potential secrets MUST be blocked.
- **Secret Review**: Before every release, a security review MUST verify that no secrets are present in source code, build artifacts, or logs.
- **No Hardcoded Credentials**: Placeholder or default credentials in code MUST be clearly marked with `TODO(SECURITY)` and replaced before production use.

**Rationale**: Committed secrets can be extracted from Git history even after deletion, leading to unauthorized API access, data breaches, and financial liability. Encrypted storage protects user credentials if a device is compromised. Pre-commit hooks provide immediate feedback during development.

### IV. User Experience Consistency

TradeLens MUST provide a consistent, accessible, and visually polished user experience:

- **Design System Adherence**: All UI components MUST follow the Figma design system specifications (colors, typography, spacing, iconography). Deviations require design review approval.
- **Theme Support**: Widgets MUST support both light and dark themes, automatically adapting based on system settings. Colors MUST be defined using theme-aware resource qualifiers.
- **Accessibility**: Text MUST use scalable units (sp) and respect user font size preferences. Minimum touch target size is 48dp. Color contrast ratios MUST meet WCAG AA standards.
- **Responsive Layouts**: Widgets MUST provide optimized layouts for standard Android widget sizes: 1×1, 2×1, 2×2, and 4×1. Content MUST scale gracefully without truncation or overflow.
- **Loading & Error States**: Widgets MUST display clear loading indicators during data refresh and user-friendly error messages when operations fail. Empty states MUST guide users on next steps.

**Rationale**: Consistent UX builds user trust and reduces cognitive load. Theme and accessibility support ensure the app is usable by a wide audience. Responsive layouts prevent poor user experiences on different device configurations.

### V. Performance & Resource Management

TradeLens MUST be resource-efficient and battery-conscious:

- **Background Work Constraints**: Widget updates MUST use WorkManager with exponential backoff policies. Updates MUST be rate-limited to avoid battery drain (e.g., no more frequent than every 15 minutes for automatic updates).
- **API Caching**: API responses MUST be cached with appropriate expiration policies. Duplicate or redundant API calls MUST be deduplicated within a configurable time window.
- **UI Responsiveness**: Visible UI updates MUST complete in <100ms. Long-running operations MUST be offloaded to background threads using coroutines or WorkManager.
- **Memory Efficiency**: Object allocations in hot paths (e.g., widget rendering) MUST be minimized. Large bitmaps MUST be sampled or scaled to appropriate dimensions before display.
- **Performance Testing**: Before each release, a performance smoke test MUST verify that widget updates complete within acceptable time bounds and do not cause ANRs (Application Not Responding) errors.

**Rationale**: Poor performance degrades user experience and increases battery consumption. Caching reduces network usage and API costs. Rate-limiting background work respects device resources and extends battery life. Performance testing catches regressions before users do.

### VI. Ethics & Privacy (NON-NEGOTIABLE)

TradeLens respects user privacy and handles data ethically:

- **Data Minimization**: User portfolio data (holdings, balances, transaction history) MUST NOT be transmitted off-device except where the user explicitly authorizes it and the destination is audited.
- **User Control**: Users MUST have a settings toggle to opt out of any data collection, analytics, or remote diagnostics. Opt-out MUST be honored immediately without requiring app restart.
- **Transparency**: Any data transmitted off-device MUST be documented in a privacy policy accessible from app settings. The purpose, destination, and retention period MUST be clearly stated.
- **Authorized Endpoints Only**: If analytics are implemented, they MUST only send data to user-configured endpoints (e.g., a self-hosted analytics server) or industry-standard privacy-respecting services explicitly approved by the user.
- **No Tracking by Default**: TradeLens MUST NOT include third-party tracking SDKs (e.g., Google Analytics, Facebook SDK) without explicit user consent during initial setup.

**Rationale**: Financial data is highly sensitive. Users have a right to privacy and control over their data. Off-device transmission creates security and privacy risks. Transparent, opt-in data handling builds user trust and complies with privacy regulations (GDPR, CCPA).

## Quality Gates & CI/CD Requirements

All pull requests MUST pass the following automated checks before merge:

- **CI Pipeline**: GitHub Actions (or equivalent) MUST run on every PR, executing linting, unit tests, and static analysis.
- **Static Analysis**: ktlint (formatting) and detekt (code smells, complexity) MUST pass without errors. New warnings MUST be addressed or explicitly suppressed with justification.
- **Unit Tests**: All unit tests MUST pass. Coverage reports MUST be generated and reviewed. PRs that decrease coverage below 60% on core modules MUST NOT be merged without justification.
- **Dependency Scanning**: Automated vulnerability scanning (e.g., Dependabot, OWASP Dependency-Check) MUST flag high or critical vulnerabilities. PRs introducing vulnerable dependencies MUST NOT be merged until vulnerabilities are resolved or mitigated.
- **Secret Detection**: Pre-commit hooks MUST prevent commits with detected secrets. CI MUST re-scan for secrets as a secondary check.

**Rationale**: Automated quality gates catch issues early in the development cycle, reducing review burden and preventing regressions. Dependency scanning protects against known vulnerabilities.

## Release Criteria

A release candidate MUST satisfy all of the following before production deployment:

1. **CI/CD Success**: All CI pipeline checks MUST pass, including tests, linting, static analysis, and vulnerability scans.
2. **Test Coverage**: Core modules MUST maintain ≥60% unit test coverage. Critical user journeys MUST have passing integration tests.
3. **Vulnerability Resolution**: No high or critical severity vulnerabilities MUST be present in dependencies or code. Medium-severity issues MUST be triaged and documented.
4. **Changelog**: A changelog entry MUST document new features, bug fixes, breaking changes, and known issues for the release.
5. **Migration Guide**: If the release includes breaking changes (e.g., storage schema changes, API contract changes), a migration guide MUST be provided documenting upgrade steps and data migration procedures.
6. **Performance Validation**: Performance smoke tests MUST pass, confirming that widget updates meet latency requirements and do not cause ANRs.
7. **Security Review**: A security review MUST confirm that no secrets are present in code, builds, or logs, and that new features comply with privacy principles.

**Rationale**: Release criteria ensure that only high-quality, secure, and well-documented code reaches production. Migration guides reduce user friction during upgrades. Security reviews catch last-minute issues before public release.

## Development Workflow

### Code Review Process

- Every PR MUST be reviewed by at least one approving reviewer.
- Reviewers MUST verify compliance with all constitutional principles (code quality, testing, security, UX, performance, privacy).
- PRs MUST include a description of the change, testing performed, and any constitutional considerations (e.g., "Added pre-commit hook check for secrets").

### Branch Strategy

- **Main Branch**: Always production-ready. Direct commits are prohibited.
- **Feature Branches**: Named `###-feature-name` (e.g., `001-widget-layout`). Branch from main, merge via PR after approval.
- **Release Branches**: Optional for hotfixes or release candidates. Named `release/vX.Y.Z`.

### Complexity Justification

- If a PR introduces complexity that violates simplicity principles (e.g., adding a heavyweight dependency, complex architectural change), the PR description MUST include a justification explaining why the complexity is necessary and what alternatives were considered.

## Governance

This constitution supersedes all other development practices and guidelines. All contributors MUST adhere to these principles.

### Amendment Process

- Amendments to this constitution MUST be proposed via a dedicated PR to `.specify/memory/constitution.md`.
- Amendments MUST include a rationale, impact analysis, and migration plan (if applicable).
- Amendments require approval from at least one project maintainer.
- Upon approval, the constitution version MUST be incremented according to semantic versioning:
  - **MAJOR**: Backward-incompatible changes (e.g., removing a principle, fundamentally redefining a principle).
  - **MINOR**: New principles, materially expanded guidance, or new sections added.
  - **PATCH**: Clarifications, wording improvements, typo fixes, non-semantic refinements.

### Versioning Policy

- The constitution version follows semantic versioning (MAJOR.MINOR.PATCH).
- Each amendment MUST update the `Last Amended` date to the date of merge.
- The `Ratified` date reflects the original adoption date and MUST NOT be changed.

### Compliance Review

- All PRs and code reviews MUST verify compliance with constitutional principles.
- During retrospectives, the team SHOULD review whether the constitution remains practical and whether any principles need amendment.
- If a constitutional principle proves impractical or counterproductive, it SHOULD be amended rather than ignored.

**Version**: 1.0.0 | **Ratified**: 2025-10-21 | **Last Amended**: 2025-10-21
