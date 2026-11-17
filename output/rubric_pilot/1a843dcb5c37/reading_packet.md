# Candidate specifications — full snapshot text

Use S IDs and line numbers in the worksheets. No automated quality labels.

## S01

    0001 | ---
    0002 | sidebar_position: 2
    0003 | sidebar_label: Specification
    0004 | title: "2025-12-15: Jira Entity Relationships SOP"
    0005 | ---
    0006 | 
    0007 | # Jira Entity Relationships SOP
    0008 | 
    0009 | **Date**: 2025-12-15
    0010 | **Status**: 🟢 In-use
    0011 | **Author**: Sri Aradhyula
    0012 | **Scope**: Jira Agent MCP Tools
    0013 | 
    0014 | ## Overview
    0015 | 
    0016 | This document defines the relationships between Jira Agile entities and the correct order of operations for the Jira Agent to traverse and manage them.
    0017 | 
    0018 | 
    0019 | ## API Reference Quick Guide
    0020 | 
    0021 | | Entity | Get All | Get One | Create | Update | Delete |
    0022 | |--------|---------|---------|--------|--------|--------|
    0023 | | **Project** | `list_projects()` | `get_project(key)` | N/A | N/A | N/A |
    0024 | | **Board** | `get_all_boards()` | `get_board(id)` | `create_board()` | N/A | `delete_board()` |
    0025 | | **Sprint** | `get_board_sprints(board_id)` | `get_sprint(id)` | `create_sprint()` | `update_sprint()` | `delete_sprint()` |
    0026 | | **Issue** | `search_issues(jql)` | `get_issue(key)` | `create_issue()` | `update_issue()` | `delete_issue()` |
    0027 | | **Backlog** | `get_backlog_issues(board_id)` | N/A | N/A | N/A | N/A |
    0028 | 
    0029 | 
    0030 | ## Related
    0031 | 
    0032 | - Jira MCP README - See `ai_platform_engineering/agents/jira/mcp/README.md` in repository
    0033 | - [Jira Agile REST API](https://developer.atlassian.com/cloud/jira/software/rest/intro/)
    0034 | - [Board API Reference](https://developer.atlassian.com/cloud/jira/software/rest/api-group-board/)
    0035 | - [Sprint API Reference](https://developer.atlassian.com/cloud/jira/software/rest/api-group-sprint/)
    0036 | 
    0037 | 
    0038 | 
    0039 | ## Related
    0040 | 
    0041 | - Architecture: [architecture.md](./architecture.md)

## S02

    0001 | ## ADDED Requirements
    0002 | 
    0003 | ### Requirement: Stream Latency Diagnostics MUST Classify Visible Output Stall After First Delta
    0004 | The system MUST record a distinct visible-output-stall classification when stream ingress exists but frontend visible text does not continue progressing.
    0005 | 
    0006 | #### Scenario: first delta arrives but visible output stalls
    0007 | - **WHEN** a streaming conversation has recorded `firstDeltaAt`
    0008 | - **AND** the active assistant message has not shown continued visible text growth within the bounded live render window
    0009 | - **THEN** diagnostics MUST classify the turn as `visible-output-stall-after-first-delta` or an equivalent explicit category
    0010 | - **AND** diagnostics MUST preserve `workspaceId`, `threadId`, `engine`, `platform`, `firstDeltaAt`, visible render timing, and active mitigation profile
    0011 | 
    0012 | #### Scenario: stall classification is independent of provider and model
    0013 | - **WHEN** the active engine is `claude`
    0014 | - **AND** the platform is Windows
    0015 | - **AND** the turn shows first-delta-then-stall evidence
    0016 | - **THEN** diagnostics MUST record the stall even if provider/model fields are empty, custom, or unrelated to Qwen
    0017 | - **AND** provider/model fields MUST remain correlation dimensions rather than root-cause gates
    0018 | 
    0019 | #### Scenario: upstream pending remains distinct from visible stall
    0020 | - **WHEN** a streaming turn has not received any assistant delta
    0021 | - **THEN** diagnostics MUST classify the condition as upstream pending, first-token delay, timeout, or equivalent
    0022 | - **AND** diagnostics MUST NOT classify it as visible output stall until at least one assistant delta has arrived

## S03

    0001 | # Spec: ctx_explore — FastContext-style iterative exploration  (refs #907)
    0002 | 
    0003 | > SDD spec-anchored: this file is the source of truth for **intent**.
    0004 | > Code + tests enforce it. When requirements change, update the spec first.
    0005 | 
    0006 | ## Problem / Why
    0007 | Microsoft **FastContext** (arXiv 2606.14066) shows that delegating *exploration*
    0008 | to a focused sub-loop — which returns compact `file:line` citations instead of raw
    0009 | file dumps — cuts main-agent tokens by ~60% with a small end-to-end quality gain.
    0010 | lean-ctx already has a stronger retrieval substrate (BM25 + hybrid + graph
    0011 | spreading + SPLADE + AST) than FastContext's grep loop, but its closest tool,
    0012 | [ctx_compose](../../rust/src/tools/ctx_compose.rs), is **single-shot** and emits
    0013 | prose Markdown rather than a parseable citation block. The gap is an **iterative,
    0014 | bounded, deterministic exploration loop with a stable citation contract**.
    0015 | 
    0016 | ## Goal
    0017 | A new `ctx_explore` MCP tool (plus `lean-ctx explore` CLI) that runs a bounded
    0018 | multi-turn retrieval loop over lean-ctx's existing engines and returns a
    0019 | byte-stable `<final_answer>` block of `path:start-end` citations.
    0020 | 
    0021 | ## Acceptance Criteria (EARS)
    0022 | - **#909** WHEN `ctx_explore` is called with a natural-language query, THE tool
    0023 |   SHALL return a `<final_answer>` block whose lines are `path:start-end` citations.
    0024 | - **#909** THE explore loop SHALL be bounded (`max_turns`, default 3) AND SHALL
    0025 |   stop early once additional turns add no new covered files (coverage saturation).
    0026 | - **#909** THE tool output SHALL be a deterministic function of (repo content,
    0027 |   query, options): it SHALL use only side-effect-free retrieval paths
    0028 |   (`search_hits`, static graph) and SHALL NOT write co-access/session state in its
    0029 |   output path (#498).
    0030 | - **#909** WHEN the `citation` option is true, THE tool SHALL emit only the
    0031 |   `<final_answer>` block (no prose summary).
    0032 | - **#910** THE MCP registry SHALL advertise `ctx_explore` AND THE CLI SHALL route
    0033 |   `lean-ctx explore <query>`; neither SHALL fall through to help (#902 gate).
    0034 | - **#910** THE registered tool-count SSOT SHALL be 78 AND the generated MCP
    0035 |   manifest + reference docs SHALL be regenerated to include `ctx_explore`.
    0036 | - **#911** THE test suite SHALL prove byte-stability (`assert_eq!(run(), run())`
    0037 |   over a warm, isolated index) and citation-block parseability.
    0038 | - **#912** THE eval harness SHALL expose an `Explore` arm and per-arm token
    0039 |   accounting so `benchmark eval-ab` can compare recall/MRR **and** tokens.
    0040 | 
    0041 | ## Out of Scope
    0042 | - The FastContext model itself (a small specialist LLM): deferred to **Phase 2**
    0043 |   as an external `explorer-brain` addon (#913) — non-deterministic, sandboxed.
    0044 | - Changing `ctx_compose` behavior or any `docs/contracts/*-v1.md`.
    0045 | - Adaptive/learned ranking signals (Hebbian co-access, session memory, heatmap)
    0046 |   inside the byte-stable block — they stay out of Phase 1's deterministic core.
    0047 | 
    0048 | ## Verification (deterministic first)
    0049 | - `cargo test -q ctx_explore` (determinism, citation parse, loop stop)
    0050 | - `cargo test -q entrypoints_wired` (explore wired in MCP + CLI)
    0051 | - `cargo test -q mcp_manifest_up_to_date` · `cargo test -q reference_docs_drift`
    0052 | - `scripts/preflight.sh full`
    0053 | - `lean-ctx benchmark eval-ab --suite rust/eval/search-suite.ndjson --json`
    0054 | 
    0055 | ## Links
    0056 | - GitLab: #907 (epic) → #908 #909 #910 #911 #912 #913 #914
    0057 | - Plan: ./plan.md · Tasks: ./tasks.md
    0058 | - Related contract: output determinism (#498)

## S04

    0001 | # Git Commit Details Specification
    0002 | 
    0003 | ## ADDED Requirements
    0004 | 
    0005 | ### Requirement: Commit Metadata Panel
    0006 | 
    0007 | The system SHALL display full metadata for selected commit.
    0008 | 
    0009 | #### Scenario: Metadata render
    0010 | 
    0011 | - **WHEN** user selects a commit
    0012 | - **THEN** panel shows full hash, author, committer (if different), timestamps, full message, and parent hashes
    0013 | 
    0014 | #### Scenario: Parent navigation
    0015 | 
    0016 | - **WHEN** user clicks a parent hash
    0017 | - **THEN** system jumps to that parent commit in history list
    0018 | 
    0019 | ---
    0020 | 
    0021 | ### Requirement: File Changes List
    0022 | 
    0023 | The system SHALL list changed files for selected commit.
    0024 | 
    0025 | #### Scenario: File row content
    0026 | 
    0027 | - **WHEN** details load completes
    0028 | - **THEN** each file row shows path, change type (A/M/D/R), additions and deletions count
    0029 | 
    0030 | #### Scenario: Compact directory chain rendering
    0031 | 
    0032 | - **WHEN** a directory path has no direct file and only one child directory
    0033 | - **THEN** the file tree displays that chain as merged label (for example `a.b.c`)
    0034 | 
    0035 | #### Scenario: Default file selection
    0036 | 
    0037 | - **WHEN** file list is non-empty
    0038 | - **THEN** first file is selected by default
    0039 | 
    0040 | #### Scenario: Click file opens diff modal
    0041 | 
    0042 | - **WHEN** user clicks a file item in the changed files tree
    0043 | - **THEN** system opens a modal diff preview for that file
    0044 | - **AND** commit message pane remains in the details layout
    0045 | 
    0046 | ---
    0047 | 
    0048 | ### Requirement: Diff View Modes
    0049 | 
    0050 | The system SHALL support unified and split diff presentation.
    0051 | 
    0052 | #### Scenario: Unified default
    0053 | 
    0054 | - **WHEN** user opens commit details
    0055 | - **THEN** diff uses unified mode by default
    0056 | 
    0057 | #### Scenario: Switch to split mode
    0058 | 
    0059 | - **WHEN** user toggles diff mode
    0060 | - **THEN** diff rerenders in split mode and preserves selected file
    0061 | 
    0062 | #### Scenario: Persist mode preference
    0063 | 
    0064 | - **WHEN** user changes diff mode
    0065 | - **THEN** preference is reused for subsequent file diffs
    0066 | 
    0067 | ---
    0068 | 
    0069 | ### Requirement: Asynchronous Diff Computation
    0070 | 
    0071 | Diff fetching SHALL be asynchronous and cancellable from UI perspective.
    0072 | 
    0073 | #### Scenario: Loading indicator
    0074 | 
    0075 | - **WHEN** diff request is running
    0076 | - **THEN** diff area shows loading state
    0077 | 
    0078 | #### Scenario: Cancel stale request
    0079 | 
    0080 | - **WHEN** user switches to another commit/file before previous request finishes
    0081 | - **THEN** previous response is ignored and SHALL NOT overwrite latest selection
    0082 | 
    0083 | #### Scenario: Timeout handling
    0084 | 
    0085 | - **WHEN** diff request exceeds timeout threshold (30s)
    0086 | - **THEN** system shows timeout error and retry action
    0087 | 
    0088 | ---
    0089 | 
    0090 | ### Requirement: Large Diff Guardrail
    0091 | 
    0092 | The system SHALL protect UI from extremely large diff payloads.
    0093 | 
    0094 | #### Scenario: Truncated large diff
    0095 | 
    0096 | - **WHEN** file diff exceeds 10,000 lines
    0097 | - **THEN** system displays truncated content and warning message
    0098 | 
    0099 | ---
    0100 | 
    0101 | ### Requirement: Binary File Handling
    0102 | 
    0103 | The system SHALL handle binary files gracefully.
    0104 | 
    0105 | #### Scenario: Non-text file changed
    0106 | 
    0107 | - **WHEN** selected file is binary
    0108 | - **THEN** system displays `Binary file changed. Diff preview unavailable.`
    0109 | 
    0110 | ---
    0111 | 
    0112 | ### Requirement: Diff Chunk Navigation
    0113 | 
    0114 | The system SHALL provide chunk-level navigation within current diff.
    0115 | 
    0116 | #### Scenario: Next/previous chunk
    0117 | 
    0118 | - **WHEN** user clicks `Next Chunk` or `Previous Chunk`
    0119 | - **THEN** viewport scrolls to corresponding changed chunk
    0120 | 
    0121 | #### Scenario: Keyboard chunk navigation
    0122 | 
    0123 | - **WHEN** user presses `n` or `p` in diff focus
    0124 | - **THEN** system jumps to next/previous chunk
    0125 | 
    0126 | ---
    0127 | 
    0128 | ### Requirement: Copy Utilities
    0129 | 
    0130 | The system SHALL provide copy actions for troubleshooting and collaboration.
    0131 | 
    0132 | #### Scenario: Copy commit hash
    0133 | 
    0134 | - **WHEN** user clicks `Copy Hash`
    0135 | - **THEN** full hash is copied to clipboard
    0136 | 
    0137 | #### Scenario: Copy commit message
    0138 | 
    0139 | - **WHEN** user clicks `Copy Message`
    0140 | - **THEN** full commit message is copied
    0141 | 
    0142 | #### Scenario: Copy file path
    0143 | 
    0144 | - **WHEN** user copies file path from file list item
    0145 | - **THEN** selected path is copied
    0146 | 
    0147 | ---
    0148 | 
    0149 | ### Requirement: Diff Statistics Summary
    0150 | 
    0151 | The system SHALL display summary statistics for selected commit.
    0152 | 
    0153 | #### Scenario: Summary render
    0154 | 
    0155 | - **WHEN** details are loaded
    0156 | - **THEN** summary includes total files changed, total additions, total deletions

## S05

    0001 | # Feature Specification: Multi-Provider Mail Transport
    0002 | 
    0003 | **Feature Branch**: `035-multi-provider-mail`  
    0004 | **Created**: 2026-06-09  
    0005 | **Status**: Draft  
    0006 | **Input**: Migrate mailing from single SMTP (nodemailer) transport to a swappable provider abstraction supporting SMTP and Resend, configurable via environment variable.
    0007 | 
    0008 | ## User Scenarios & Testing _(mandatory)_
    0009 | 
    0010 | ### User Story 1 - Switch to Resend without code changes (Priority: P1)
    0011 | 
    0012 | An operator configures `MAIL_PROVIDER=resend` and `RESEND_API_KEY=...` in environment variables. On next deploy, all outgoing emails route through Resend instead of SMTP — no code change, no downtime.
    0013 | 
    0014 | **Why this priority**: Core goal of the feature. Enables the Resend migration without a hard cut-over and without touching any business logic.
    0015 | 
    0016 | **Independent Test**: Set `MAIL_PROVIDER=resend` and a valid API key, trigger `sendTestMail()`, confirm email arrives via Resend dashboard.
    0017 | 
    0018 | **Acceptance Scenarios**:
    0019 | 
    0020 | 1. **Given** `MAIL_ENABLED=true`, `MAIL_PROVIDER=resend`, and a valid `RESEND_API_KEY`, **When** any send method is called, **Then** the email is delivered via Resend and does not use SMTP at all.
    0021 | 2. **Given** `MAIL_PROVIDER=resend` and an invalid or missing `RESEND_API_KEY`, **When** a send is attempted, **Then** the error is logged and the application continues without crashing.
    0022 | 
    0023 | ---
    0024 | 
    0025 | ### User Story 2 - Keep SMTP working unchanged (Priority: P1)
    0026 | 
    0027 | An operator running `MAIL_PROVIDER=smtp` (or no `MAIL_PROVIDER` set at all) sees zero change in behavior: emails still route through nodemailer with `MAIL_HOST`, `MAIL_USER`, `MAIL_PASS`.
    0028 | 
    0029 | **Why this priority**: No regressions. Existing deployments must be unaffected before any migration.
    0030 | 
    0031 | **Independent Test**: Leave `MAIL_PROVIDER` unset, trigger any send method, confirm delivery via SMTP as before.
    0032 | 
    0033 | **Acceptance Scenarios**:
    0034 | 
    0035 | 1. **Given** `MAIL_ENABLED=true` and `MAIL_PROVIDER` absent or set to `smtp`, **When** any send method is called, **Then** the email is delivered via nodemailer SMTP.
    0036 | 2. **Given** SMTP credentials are invalid, **When** a send is attempted, **Then** the error is logged and the application continues (same behaviour as today).
    0037 | 
    0038 | ---
    0039 | 
    0040 | ### User Story 3 - Mailing disabled path still works (Priority: P2)
    0041 | 
    0042 | When `MAIL_ENABLED=false`, compiled email HTML is saved to file in development mode and no transport is invoked — regardless of which provider is configured.
    0043 | 
    0044 | **Why this priority**: Preserves existing local-dev workflow used to inspect email output without a live mail server.
    0045 | 
    0046 | **Independent Test**: Set `MAIL_ENABLED=false` and `NODE_ENV=development`, trigger `sendTestMail()`, confirm `mails/test.html` is written and no network call is made.
    0047 | 
    0048 | **Acceptance Scenarios**:
    0049 | 
    0050 | 1. **Given** `MAIL_ENABLED=false` and `NODE_ENV=development`, **When** any send method is called, **Then** the rendered HTML is saved to `mails/<template>.html` and no transport sends a network request.
    0051 | 2. **Given** `MAIL_ENABLED=false` and `NODE_ENV=production`, **When** any send method is called, **Then** the method returns silently without sending or writing a file.
    0052 | 
    0053 | ---
    0054 | 
    0055 | ### User Story 4 - Add a third provider in future with minimal friction (Priority: P3)
    0056 | 
    0057 | A future developer adds a third mail provider (e.g. Mailgun) by creating one new file implementing the transport interface, registering it in the module factory — with zero changes to `MailingService` or any of its 16 public methods.
    0058 | 
    0059 | **Why this priority**: Validates the abstraction is genuinely open for extension without modification.
    0060 | 
    0061 | **Independent Test**: Verify that `MailingService` has no direct import of any transport implementation — only the interface token.
    0062 | 
    0063 | **Acceptance Scenarios**:
    0064 | 
    0065 | 1. **Given** the transport abstraction is in place, **When** reviewing `MailingService` source, **Then** it imports only the interface/token, not `nodemailer` or `resend` directly.
    0066 | 2. **Given** a new provider class implementing `IMailTransport`, **When** it is wired into the module factory and selected via config, **Then** it works without changes to `MailingService`.
    0067 | 
    0068 | ---
    0069 | 
    0070 | ### Edge Cases
    0071 | 
    0072 | - What happens when `MAIL_PROVIDER` is set to an unknown value? → Log a warning and fall back to SMTP.
    0073 | - What happens when `RESEND_API_KEY` is missing while `MAIL_PROVIDER=resend`? → Log an error and treat mailing as disabled for that boot.
    0074 | - What happens when the SMTP `verify()` fails? → Log warning, mark provider as disabled; subsequent `send()` calls no-op. Verify runs lazily on the first `send()`, not at module startup, so a temporarily unavailable SMTP server does not block app boot.
    0075 | - What happens if `cc` is omitted from a send call? → Transport providers must handle `cc` as optional without error.
    0076 | - What if a send method is called concurrently from multiple workers? → Providers are stateless per-send; no shared mutable state issues.
    0077 | - What happens when a provider `send()` call fails with a transient error (timeout, rate limit)? → Log the error and return silently. No retry. Callers (Bull queue processors) own retry semantics at a higher level.
    0078 | 
    0079 | ## Test Suite
    0080 | 
    0081 | Tests are written **before** implementation (TDD). No provider code ships without a passing spec.
    0082 | 
    0083 | ### `mailing.service.spec.ts` (new)
    0084 | 
    0085 | Unit tests for `MailingService` with a mock transport injected via `MAIL_TRANSPORT_TOKEN`:
    0086 | 
    0087 | - Service calls `transport.send()` with pre-compiled HTML (not raw template name)
    0088 | - Subject prefix is prepended when `MAIL_SUBJECT_PREFIX` is set
    0089 | - Dev email override replaces `to`/`cc` when `NODE_ENV !== production`
    0090 | - `DEV_EMAIL_DESTINATION` missing → method returns without sending
    0091 | - `MAIL_ENABLED=false` (transport is `null`) → HTML written to file in development, silent return in production; `transport.send()` never called
    0092 | - Representative public method (e.g. `sendEnrollmentMail`) passes correct context shape to `compileService.toHtml()`
    0093 | 
    0094 | ### `smtp.provider.spec.ts` (new)
    0095 | 
    0096 | Unit tests for `SmtpProvider` with nodemailer mocked:
    0097 | 
    0098 | - `send()` calls `transporter.sendMail()` with `from`, `to`, `cc`, `subject`, `html`
    0099 | - `cc` omitted → passed as `undefined`, no error thrown
    0100 | - Constructor reads `MAIL_HOST`, `MAIL_USER`, `MAIL_PASS` from config; does NOT call `verify()` at construction
    0101 | - First `send()` triggers `verify()`; subsequent calls skip it (cached flag)
    0102 | - `verify()` failure → provider logs warning, marks itself disabled, `send()` no-ops for all subsequent calls
    0103 | 
    0104 | ### `resend.provider.spec.ts` (new)
    0105 | 
    0106 | Unit tests for `ResendProvider` with Resend SDK mocked:
    0107 | 
    0108 | - `send()` calls `resend.emails.send()` with correct shape
    0109 | - `cc` omitted → not included in Resend call
    0110 | - Missing `RESEND_API_KEY` → provider logs error, `send()` no-ops gracefully
    0111 | - Resend SDK throws → error is caught and logged, does not propagate
    0112 | 
    0113 | ### Existing tests
    0114 | 
    0115 | - `clubenrollment.template.spec.ts` — unchanged, must stay green throughout
    0116 | 
    0117 | ## Requirements _(mandatory)_
    0118 | 
    0119 | ### Functional Requirements
    0120 | 
    0121 | - **FR-001**: System MUST support selecting a mail transport provider via a single environment variable (`MAIL_PROVIDER`: `smtp` | `resend`).
    0122 | - **FR-002**: System MUST default to SMTP transport when `MAIL_PROVIDER` is absent.
    0123 | - **FR-003**: System MUST deliver emails via Resend when `MAIL_PROVIDER=resend` and `RESEND_API_KEY` is set.
    0124 | - **FR-004**: System MUST continue delivering emails via nodemailer SMTP when `MAIL_PROVIDER=smtp`, using `MAIL_HOST`, `MAIL_USER`, `MAIL_PASS`.
    0125 | - **FR-005**: All 16 existing `MailingService` public methods MUST retain identical signatures and behaviour.
    0126 | - **FR-006**: Template compilation (Pug → HTML) MUST remain in `MailingService`, not inside any transport provider.
    0127 | - **FR-007**: Dev email override logic (`DEV_EMAIL_DESTINATION`, non-production redirect) MUST remain in `MailingService`.
    0128 | - **FR-008**: The disabled-mailing file-save path (`MAIL_ENABLED=false` + `NODE_ENV=development`) MUST remain functional regardless of configured provider.
    0129 | - **FR-009**: Transport providers MUST be interchangeable via a shared interface without modifying `MailingService`.
    0130 | - **FR-010**: The `resend` package MUST be added as a dependency; `nodemailer` MUST NOT be removed.
    0131 | - **FR-011**: `.env.example` MUST be updated to document `MAIL_PROVIDER` and `RESEND_API_KEY` alongside existing mail vars.
    0132 | - **FR-012**: On any send failure (network error, API rejection, timeout), providers MUST log the error and return silently — no retry, no exception propagation to the caller.
    0133 | - **FR-013**: `SmtpProvider` MUST initialize the nodemailer transporter lazily — `verify()` runs on the first `send()` call, not at module startup. Result is cached; subsequent calls skip re-verification.
    0134 | 
    0135 | ### Key Entities
    0136 | 
    0137 | - **IMailTransport**: Interface defining the contract for any mail transport provider (`send(options): Promise<void>`).
    0138 | - **MailSendOptions**: Data shape passed to providers (`from`, `to`, `cc?`, `subject`, `html`).
    0139 | - **SmtpProvider**: Concrete implementation wrapping nodemailer; reads `MAIL_HOST`, `MAIL_USER`, `MAIL_PASS`.
    0140 | - **ResendProvider**: Concrete implementation wrapping Resend SDK; reads `RESEND_API_KEY`.
    0141 | - **MAIL_TRANSPORT_TOKEN**: NestJS injection token used to inject the active provider into `MailingService`.
    0142 | 
    0143 | ## Success Criteria _(mandatory)_
    0144 | 
    0145 | ### Measurable Outcomes
    0146 | 
    0147 | - **SC-001**: Switching mail provider requires only environment variable changes — zero code changes and zero redeployment of application logic.
    0148 | - **SC-002**: All existing email delivery paths continue working after the refactor (verified by existing unit tests passing without modification).
    0149 | - **SC-003**: A new provider can be added by creating one file and one line in the module factory — no changes to `MailingService` or its callers.
    0150 | - **SC-004**: `MailingService` source contains no direct import of `nodemailer` or `resend` after the refactor.
    0151 | - **SC-005**: Both SMTP and Resend paths are exercisable in a development environment without a live mail server (via `MAIL_ENABLED=false` file-save mode).
    0152 | - **SC-006**: Unit test suites for `MailingService`, `SmtpProvider`, and `ResendProvider` exist and pass before any provider implementation is merged. Each provider is tested in full isolation with no live network calls.
    0153 | 
    0154 | ## Assumptions
    0155 | 
    0156 | - `MAIL_PROVIDER` defaults to `smtp` to ensure zero-change backward compatibility for existing deployments.
    0157 | - The Resend SDK (`resend` npm package v4.x) is publicly available and compatible with Node.js 20+.
    0158 | - Template compilation behaviour (Pug → HTML via `CompileService`) is not changing as part of this feature.
    0159 | - The `cc` field is optional in all provider implementations, consistent with how it is used today (only `sendEnrollmentMail` sets it).
    0160 | - Subject prefix (`MAIL_SUBJECT_PREFIX`) and dev-override (`DEV_EMAIL_DESTINATION`) config vars remain in `MailingService` scope.
    0161 | - Mobile/frontend code is unaffected — this change is entirely within `libs/backend/mailing/`.
    0162 | 
    0163 | ## Clarifications
    0164 | 
    0165 | ### Session 2026-06-09
    0166 | 
    0167 | - Q: What should providers do when a send call fails with a transient error (network timeout, rate limit)? → A: Log error, return silently — no retry. Bull queue processors own retry semantics at a higher level.
    0168 | - Q: When should `SmtpProvider` run nodemailer `verify()`? → A: Lazily on first `send()`, result cached. Keeps module startup fast and consistent with ResendProvider (no verify step).

## S06

    0001 | ## MODIFIED Requirements
    0002 | 
    0003 | ### Requirement: Multi-tailnet profile schema is supported
    0004 | The CLI configuration SHALL support an `active-tailnet` key and a `tailnets` collection of profile objects. Each profile SHALL include a `name` field and MAY include `tailnet`, `api-key`, `oauth-client-id`, and `oauth-client-secret`. A valid profile SHALL contain either `api-key`, or both `oauth-client-id` and `oauth-client-secret`. When profile-backed configuration is persisted, that profile schema SHALL be the canonical stored representation and SHALL NOT require duplicate top-level `tailnet` or `api-key` keys.
    0005 | 
    0006 | #### Scenario: Config with API-key profile is loaded
    0007 | - **WHEN** the config file contains `active-tailnet` and a matching profile with `name` and `api-key`
    0008 | - **THEN** the CLI SHALL treat that profile as available for API-key runtime credential resolution
    0009 | 
    0010 | #### Scenario: Config with OAuth profile is loaded
    0011 | - **WHEN** the config file contains `active-tailnet` and a matching profile with `name`, `oauth-client-id`, and `oauth-client-secret`
    0012 | - **THEN** the CLI SHALL treat that profile as available for OAuth-backed command authentication
    0013 | 
    0014 | #### Scenario: Profile tailnet defaults to profile name
    0015 | - **WHEN** a profile omits `tailnet`
    0016 | - **THEN** commands that require a tailnet value SHALL treat the profile `name` as the effective tailnet
    0017 | 
    0018 | #### Scenario: Invalid profile auth shape is provided
    0019 | - **WHEN** a profile contains neither `api-key` nor a complete `oauth-client-id` and `oauth-client-secret` pair
    0020 | - **THEN** profile validation SHALL fail with an actionable invalid-auth-shape error
    0021 | 
    0022 | #### Scenario: Duplicate profile names are provided
    0023 | - **WHEN** two profile entries use the same `name`
    0024 | - **THEN** profile validation SHALL fail with an actionable duplicate-name error
    0025 | 
    0026 | #### Scenario: Profile config is persisted canonically
    0027 | - **WHEN** a config write occurs for a file that contains profile data
    0028 | - **THEN** the persisted config SHALL store profile state using `active-tailnet` and `tailnets`
    0029 | - **AND** the persisted config SHALL preserve the relevant auth fields for each profile
    0030 | - **AND** the persisted config SHALL NOT depend on duplicated top-level `tailnet` or `api-key` keys to represent the active profile
    0031 | 
    0032 | ### Requirement: Runtime credential resolution follows deterministic precedence
    0033 | For operational commands, effective values SHALL be resolved in this order: flags, environment variables, active profile, legacy flat config keys where supported by the command. API-key commands SHALL continue to resolve `api-key` and `tailnet` using that precedence. OAuth lifecycle commands SHALL resolve `oauth-client-id` and `oauth-client-secret` using the same precedence model, then exchange those credentials for a bearer token at runtime.
    0034 | 
    0035 | #### Scenario: Flags override OAuth profile values
    0036 | - **WHEN** `--oauth-client-id` or `--oauth-client-secret` is provided and OAuth profile data exists
    0037 | - **THEN** the CLI SHALL use the flag values as the effective OAuth credential input
    0038 | 
    0039 | #### Scenario: Environment overrides OAuth profile values
    0040 | - **WHEN** OAuth flags are absent and `TSCLI_OAUTH_CLIENT_ID` or `TSCLI_OAUTH_CLIENT_SECRET` is set while OAuth profile data exists
    0041 | - **THEN** the CLI SHALL use the environment values as the effective OAuth credential input
    0042 | 
    0043 | #### Scenario: Active OAuth profile is used by default
    0044 | - **WHEN** OAuth flags and environment variables are absent and `active-tailnet` maps to a profile with `oauth-client-id` and `oauth-client-secret`
    0045 | - **THEN** OAuth lifecycle commands SHALL use that profile for token exchange
    0046 | 
    0047 | #### Scenario: API-key command keeps current precedence
    0048 | - **WHEN** an existing API-key-authenticated command runs and flags, environment variables, profile data, and legacy keys are present
    0049 | - **THEN** the CLI SHALL continue to resolve `api-key` and `tailnet` using flags over environment over active profile over legacy config
    0050 | 
    0051 | #### Scenario: OAuth credentials are missing from all sources
    0052 | - **WHEN** an OAuth-authenticated command runs and flags, environment variables, and the active profile do not provide a complete OAuth client id and secret pair
    0053 | - **THEN** command execution SHALL fail with an actionable required OAuth credentials error
    0054 | 
    0055 | ### Requirement: Config commands manage tailnet profiles
    0056 | The `config` command group SHALL provide operations to list profiles, set active profile, upsert profile credentials, and remove profiles for both API-key-backed and OAuth-backed profile entries.
    0057 | 
    0058 | #### Scenario: List profiles displays active selection and auth shape
    0059 | - **WHEN** multiple profiles exist and one is active
    0060 | - **THEN** the list output SHALL include all profile names
    0061 | - **AND** the output SHALL indicate which profile is active
    0062 | - **AND** the output SHALL indicate whether each profile is API-key-backed or OAuth-backed
    0063 | 
    0064 | #### Scenario: Set active profile validates existence
    0065 | - **WHEN** a user selects an active profile name that does not exist in `tailnets`
    0066 | - **THEN** the command SHALL fail with a validation error that names the missing profile
    0067 | 
    0068 | #### Scenario: Upsert API-key profile writes persistent config
    0069 | - **WHEN** a user runs `config profiles upsert <name> --api-key <key>`
    0070 | - **THEN** the CLI SHALL persist an API-key-backed profile entry
    0071 | - **AND** the profile SHALL be available on the next command execution
    0072 | 
    0073 | #### Scenario: Upsert OAuth profile writes persistent config
    0074 | - **WHEN** a user runs `config profiles upsert <name> --oauth-client-id <id> --oauth-client-secret <secret>`
    0075 | - **THEN** the CLI SHALL persist an OAuth-backed profile entry
    0076 | - **AND** the profile SHALL be available on the next command execution
    0077 | 
    0078 | #### Scenario: Profile mutations keep canonical file shape
    0079 | - **WHEN** a user runs `config profiles upsert` or `config profiles set-active`
    0080 | - **THEN** the saved config SHALL retain the selected `active-tailnet` and `tailnets` entries
    0081 | - **AND** the saved config SHALL NOT add or refresh duplicated top-level `tailnet` and `api-key` keys for the active profile
    0082 | 
    0083 | #### Scenario: Remove active profile is blocked
    0084 | - **WHEN** a user attempts to remove the currently active profile without first selecting another profile
    0085 | - **THEN** the command SHALL fail with guidance to switch active profile first
    0086 | 
    0087 | ### Requirement: Profile behavior is covered by automated tests
    0088 | The project SHALL include unit and integration tests for profile parsing, validation, runtime resolution, and profile command behavior across both API-key-backed and OAuth-backed profiles.
    0089 | 
    0090 | #### Scenario: Unit tests cover OAuth resolver precedence
    0091 | - **WHEN** automated unit tests execute for config resolution
    0092 | - **THEN** tests SHALL verify precedence across OAuth flags, environment variables, active profile values, and missing-credential failures
    0093 | 
    0094 | #### Scenario: Unit tests cover canonical persistence for mixed auth profiles
    0095 | - **WHEN** automated config persistence tests execute for profile-backed config
    0096 | - **THEN** tests SHALL verify that saved files keep `active-tailnet` and `tailnets`
    0097 | - **AND** tests SHALL verify that persisted profiles retain the relevant API-key or OAuth fields without reintroducing duplicated flat keys
    0098 | 
    0099 | #### Scenario: Integration tests cover OAuth profile command flows
    0100 | - **WHEN** command-level integration tests execute for config profile operations and OAuth-backed lifecycle commands
    0101 | - **THEN** tests SHALL verify validation failures, persisted state transitions, auth resolution, and successful lifecycle requests using active profiles

## S07

    0001 | # Spec: Always-Visible Source File Button
    0002 | 
    0003 | **Branch**: 015-source-button-visibility | **Date**: 2026-02-28
    0004 | 
    0005 | ## Summary
    0006 | 
    0007 | The "Edit Source" button in the spec viewer footer is currently hidden when the spec status is "spec-completed" (the entire footer gets `display: none`). This makes it impossible to open the raw markdown file for completed specs. The button should always be visible regardless of completion status. Additionally, an "Open Source" inline action should be added to spec document items in the sidebar tree view for quick access.
    0008 | 
    0009 | ## Requirements
    0010 | 
    0011 | - **R001** (MUST): The "Edit Source" button in the spec viewer footer must remain visible and functional when spec status is "spec-completed"
    0012 | - **R002** (MUST): Other footer actions (Regenerate, Enhance, Approve/Generate) must still be hidden when spec is completed — only "Edit Source" stays
    0013 | - **R003** (SHOULD): Each spec document item (spec, plan, tasks) in the sidebar tree view should have an inline "Open Source" action that opens the raw markdown file in the editor
    0014 | 
    0015 | ## Scenarios
    0016 | 
    0017 | ### Viewing a completed spec in the viewer
    0018 | 
    0019 | **When** a user opens a spec with status "spec-completed" in the spec viewer
    0020 | **Then** the "Edit Source" button is visible and clickable in the footer; other action buttons (Regenerate, Enhance, primary CTA) are hidden
    0021 | 
    0022 | ### Opening source from sidebar
    0023 | 
    0024 | **When** a user hovers over a spec document item (spec/plan/tasks) in the sidebar explorer tree
    0025 | **Then** an inline "Open Source" icon button appears that opens the raw `.md` file in the VS Code editor
    0026 | 
    0027 | ## Out of Scope
    0028 | 
    0029 | - Changing the "Edit Source" button label or behavior (it already works via `editSource` message)
    0030 | - Modifying the sidebar context menu (right-click) — only adding an inline hover action

## S08

    0001 | ## ADDED Requirements
    0002 | 
    0003 | ### Requirement: Project-Specific Semgrep Rule Execution and Finding Extraction
    0004 | 
    0005 | The system SHALL invoke semgrep with the project-specific clean_code ruleset and map findings to `List[ReviewFinding]`, filtered to the provided file list.
    0006 | 
    0007 | #### Scenario: Semgrep finding maps to ReviewFinding
    0008 | 
    0009 | - **GIVEN** semgrep output with a match on the `get-modify-same-method` rule
    0010 | - **WHEN** `run_semgrep(files=[...])` is called
    0011 | - **THEN** a `ReviewFinding` is returned with `tool="semgrep"` and `category="clean_code"`
    0012 | 
    0013 | #### Scenario: Non-provided files filtered out
    0014 | 
    0015 | - **GIVEN** semgrep finds matches in `file_a.py` and `file_b.py`, only `file_a.py` provided
    0016 | - **WHEN** `run_semgrep(files=[file_a.py])` is called
    0017 | - **THEN** only findings from `file_a.py` are returned
    0018 | 
    0019 | #### Scenario: Semgrep unavailable produces tool_error
    0020 | 
    0021 | - **GIVEN** semgrep binary is unavailable
    0022 | - **WHEN** `run_semgrep(files=[...])` is called
    0023 | - **THEN** one `ReviewFinding` with `category="tool_error"` is returned and no exception propagates
    0024 | 
    0025 | #### Scenario: Clean file produces no findings
    0026 | 
    0027 | - **GIVEN** a file that matches none of the 5 custom rules
    0028 | - **WHEN** `run_semgrep(files=[file])` is called
    0029 | - **THEN** an empty list is returned

## S09

    0001 | # 005: SOUL Identity + Skills System
    0002 | 
    0003 | ## Problem
    0004 | 
    0005 | Matrix OS currently has no personality. Every response is generic. Users can't customize how the OS communicates, and the OS can't learn new behaviors without code changes.
    0006 | 
    0007 | ## Solution
    0008 | 
    0009 | Two complementary systems:
    0010 | 
    0011 | 1. **SOUL** -- a file at `~/system/soul.md` that defines the OS identity (personality, values, communication style). Injected into every prompt at L0 cache level (never evicted).
    0012 | 
    0013 | 2. **Skills** -- markdown files at `~/agents/skills/*.md` that teach the kernel new behaviors. Skills are prompt injections, not tools. They expand what the OS knows how to do without code changes.
    0014 | 
    0015 | ## SOUL Identity
    0016 | 
    0017 | ### What SOUL Is
    0018 | 
    0019 | A markdown file that defines WHO the OS is. Not capabilities (those are skills) or state (that's `state.md`), but identity.
    0020 | 
    0021 | ### Location
    0022 | 
    0023 | `~/system/soul.md` -- follows Everything Is a File principle. User can edit directly or ask the OS to change it ("be more formal", "your name is Jarvis").
    0024 | 
    0025 | ### Default SOUL
    0026 | 
    0027 | ```markdown
    0028 | # Matrix OS
    0029 | 
    0030 | ## Identity
    0031 | I am Matrix OS -- a personal operating system and AI assistant. I run on your machine (or your server), I generate software from conversation, and I'm always here when you need me.
    0032 | 
    0033 | ## Personality
    0034 | - Direct and clear. I don't waste words.
    0035 | - Curious about your needs. I ask when something is ambiguous.
    0036 | - Proactive. I suggest improvements and notice patterns.
    0037 | - Honest about limitations. I say when I can't do something.
    0038 | 
    0039 | ## Values
    0040 | - Your data is yours. Everything is a file you own.
    0041 | - Privacy first. I don't phone home.
    0042 | - Transparency. I explain what I'm doing and why.
    0043 | - Reliability. I'd rather do less than promise more than I can deliver.
    0044 | 
    0045 | ## Communication Style
    0046 | - Concise responses for simple questions
    0047 | - Detailed explanations when building something complex
    0048 | - I adapt to the channel: shorter for messaging, richer for web
    0049 | - I use the user's language and tone
    0050 | ```
    0051 | 
    0052 | ### How SOUL Loads
    0053 | 
    0054 | `buildSystemPrompt()` reads `~/system/soul.md` and injects it after the core identity section, before state/knowledge. SOUL is L0 -- always present, never compressed. Cost: ~300-500 tokens per request.
    0055 | 
    0056 | ## Identity Files
    0057 | 
    0058 | Beyond SOUL, the kernel loads three additional system files that define the OS and its relationship with the user:
    0059 | 
    0060 | ### `~/system/identity.md` -- Public Identity
    0061 | 
    0062 | Who the OS is externally: name, nature (AI Operating System), vibe, emoji, avatar URL. This is the "business card" -- used when the OS introduces itself or responds to "who are you?". Fields are user-editable.
    0063 | 
    0064 | ### `~/system/user.md` -- User Profile
    0065 | 
    0066 | Who the human is: name, what to call them, pronouns, timezone, notes, and free-form context. The OS uses this to personalize responses (language, units, tone). Replaces the old `agents/user-profile.md`.
    0067 | 
    0068 | ### `~/system/bootstrap.md` -- First-Run Ritual
    0069 | 
    0070 | Only exists on first boot. Guides the onboarding conversation: greet the user, update identity/user/soul files with their preferences, suggest building something together, then self-delete. The prompt includes a "FIRST RUN" instruction when this file exists.
    0071 | 
    0072 | ### Loading Order
    0073 | 
    0074 | `buildSystemPrompt()` injects identity files in this order (all L0, never evicted):
    0075 | 
    0076 | 1. SOUL (personality, values, communication style)
    0077 | 2. Identity (public-facing identity card)
    0078 | 3. User (human profile and preferences)
    0079 | 4. Bootstrap (first-run only, deleted after onboarding)
    0080 | 
    0081 | ## Skills System
    0082 | 
    0083 | ### What Skills Are
    0084 | 
    0085 | Markdown files that teach the kernel new behaviors. NOT tools (tools are code). Skills are prompt injections that expand capability.
    0086 | 
    0087 | ### Format
    0088 | 
    0089 | ```markdown
    0090 | ---
    0091 | name: weather
    0092 | description: Look up current weather for any location
    0093 | triggers:
    0094 |   - weather
    0095 |   - forecast
    0096 |   - temperature
    0097 | ---
    0098 | 
    0099 | # Weather Lookup
    0100 | 
    0101 | When the user asks about weather:
    0102 | 1. Use WebSearch to find current weather
    0103 | 2. Extract: temperature, conditions, humidity, wind
    0104 | 3. Format based on channel (concise for messaging, detailed for web)
    0105 | ```
    0106 | 
    0107 | ### Loading
    0108 | 
    0109 | 1. On boot, kernel scans `~/agents/skills/*.md`
    0110 | 2. Parses frontmatter: `name`, `description`, `triggers`
    0111 | 3. Builds skills TOC (~5 tokens per skill) injected into system prompt
    0112 | 4. On matching request, loads full skill body via `load_skill` IPC tool
    0113 | 5. Skill body injected into current turn context
    0114 | 
    0115 | ### Built-in Skills
    0116 | 
    0117 | | Skill | Triggers | What it does |
    0118 | |-------|----------|-------------|
    0119 | | `summarize.md` | summarize, tldr, summary | Summarize text/articles/conversations |
    0120 | | `weather.md` | weather, forecast, temperature | Weather lookup via web search |
    0121 | | `reminder.md` | remind, reminder, alarm, schedule | Create cron reminders |
    0122 | | `skill-creator.md` | learn, new skill, teach | Meta-skill: create new skill files |
    0123 | 
    0124 | ### Self-Expanding
    0125 | 
    0126 | The `skill-creator.md` meta-skill teaches the kernel how to create new skills. "Learn how to check my GitHub stars" -> kernel writes `~/agents/skills/github-stars.md` -> available on next interaction.
    0127 | 
    0128 | ## Dependencies
    0129 | 
    0130 | - Phase 3 (Kernel) -- complete
    0131 | - `buildSystemPrompt()` exists in `packages/kernel/src/prompt.ts`
    0132 | - Agent frontmatter parser exists in `packages/kernel/src/agents.ts`
    0133 | - IPC MCP server exists in `packages/kernel/src/ipc.ts`
    0134 | 
    0135 | ## Inspired By
    0136 | 
    0137 | - Nanobot's `SOUL.md` (workspace/SOUL.md)
    0138 | - Nanobot's `skills/` (YAML frontmatter + markdown body)
    0139 | - OpenClaw's agent identity system

## S10

    0001 | ## ADDED Requirements
    0002 | 
    0003 | ### Requirement: 在线 AI 卡死必须有服务端权威兜底
    0004 | 系统 SHALL 在 `GameTransportServer` 内部为在线房间的 AI 座位维护权威 recovery watchdog，而不是只依赖房主页面上的前端桥接层。
    0005 | 
    0006 | #### Scenario: 房主页面未能提交 AI 恢复命令时，服务端仍可接管
    0007 | - **GIVEN** 某个在线房间存在 AI seat
    0008 | - **AND** 当前阻塞状态属于该 AI seat 的隐藏交互、可见交互、响应窗口或活动回合
    0009 | - **WHEN** 进度 marker 在阈值窗口内持续不变
    0010 | - **THEN** 服务端 MUST 在权威状态上生成并执行恢复动作
    0011 | - **AND** 不得要求房主页面存活、联网或持有可用 AI seat 凭据才可恢复
    0012 | 
    0013 | #### Scenario: 当前轮到真人时，服务端不得误判为 AI 卡死
    0014 | - **GIVEN** 当前活动回合属于 human seat，或当前 response window responder 属于 human seat
    0015 | - **WHEN** recovery watchdog 扫描房间状态
    0016 | - **THEN** 服务端 MUST 不生成任何 AI 强制恢复动作
    0017 | 
    0018 | ### Requirement: 在线 AI 强制恢复必须采用两段式权威确认
    0019 | 系统 SHALL 先解开 AI 当前阻塞，再基于最新权威状态多步推进阶段直到安全收口。
    0020 | 
    0021 | #### Scenario: 交互或响应窗口卡住时先解阻塞
    0022 | - **GIVEN** 某个 AI seat 正卡在交互或响应窗口
    0023 | - **WHEN** 服务端 watchdog 触发恢复
    0024 | - **THEN** 第一段恢复 MUST 只提交 `RESPOND / CANCEL / PASS` 之类的最小恢复命令
    0025 | - **AND** 不得在同一段恢复里直接假定后续阶段必然可推进
    0026 | 
    0027 | #### Scenario: 解阻塞后仍是该 AI 的回合才允许 follow-up ADVANCE_PHASE
    0028 | - **GIVEN** 第一段恢复已执行成功
    0029 | - **WHEN** 服务端读取最新权威状态
    0030 | - **THEN** 只有在“仍是该 AI 的回合 + 没有新交互 + 没有新 response window”时，系统才可以继续生成 `ADVANCE_PHASE`
    0031 | - **AND** 系统 MUST 在每次推进后重新读取权威状态，直到切回 human、出现新阻塞或命中安全步数上限
    0032 | - **AND** 若最新权威状态已切回 human 或出现新阻塞，系统 MUST 立即终止后续 follow-up
    0033 | 
    0034 | ### Requirement: 无解交互不得继续把 AI 困在阻塞态
    0035 | 系统 SHALL 在交互层与 AI 决策层共同兜底“当前 interaction 无任何可解路径”的场景，而不是只能等 watchdog 超时救火。
    0036 | 
    0037 | #### Scenario: simple-choice 选项为空、全 disabled 或最少选择数不可达时，系统应立即变为可解
    0038 | - **GIVEN** 当前 `simple-choice` interaction 在创建后或动态刷新后出现以下任一情况：
    0039 |   - 选项为空
    0040 |   - 所有选项都为 disabled
    0041 |   - `multi.min` 大于当前 enabled 选项数
    0042 | - **WHEN** 系统准备把该 interaction 交给玩家或 AI 消费
    0043 | - **THEN** 系统 MUST 注入或保留一条可执行的恢复路径（如 `__emergency_skip__`、空选择或显式 skip/cancel）
    0044 | - **AND** 不得继续保留“阻塞了 `ADVANCE_PHASE` 但又没有任何可执行动作”的死锁态
    0045 | 
    0046 | #### Scenario: AI 缺少游戏专属交互动作时，仍应能走通用恢复动作
    0047 | - **GIVEN** 某个游戏自己的 AI `buildLegalActions` 没有为当前无解 interaction 生成动作
    0048 | - **WHEN** 引擎构建 AI 决策上下文
    0049 | - **THEN** 引擎 SHOULD 自动补一个通用 interaction fallback action
    0050 | - **AND** 该 fallback 至少应覆盖空选择或 `__emergency_skip__` 这类确定性恢复路径
    0051 | 
    0052 | ### Requirement: 强制恢复 incident 必须自动反馈且可去重
    0053 | 系统 SHALL 对在线 AI recovery incident 执行 best-effort 自动反馈，并对同一 incident 做去重与冷却。
    0054 | 
    0055 | #### Scenario: 强制结束或恢复失败达到阈值时自动写入反馈
    0056 | - **GIVEN** 服务端 watchdog 成功执行了 `force-end-turn`，或同一 incident 连续恢复失败达到阈值
    0057 | - **WHEN** 系统准备记录该 incident
    0058 | - **THEN** 系统 MUST 向现有 `/feedback` 管道提交一条结构化 bug 反馈
    0059 | - **AND** 反馈 MUST 包含至少 `gameId`、`matchId`、`playerId`、incident kind、reason 与精简 `stateSnapshot`
    0060 | 
    0061 | #### Scenario: 交互类 incident 的反馈应尽量说明“为什么无法选择”
    0062 | - **GIVEN** 某条 AI recovery incident 与 interaction choice 有关
    0063 | - **WHEN** 系统生成自动反馈
    0064 | - **THEN** `stateSnapshot` SHOULD 包含共享视角或 seat 视角下的 interaction options 摘要
    0065 | - **AND** 若存在 disabled 选项，反馈 SHOULD 记录其 disabled 状态与可用的禁用原因字段
    0066 | - **AND** 系统 SHOULD 额外给出可选性诊断（如 `no-options`、`all-options-disabled`、`manual-selection-required`、`recoverable-option-available`）
    0067 | 
    0068 | #### Scenario: AI 触发 emergency skip 时应立即自动反馈
    0069 | - **GIVEN** AI seat 没有等待 watchdog，而是在当前 interaction 中直接走了 `__emergency_skip__`
    0070 | - **WHEN** 服务端收到该交互取消事件
    0071 | - **THEN** 系统 SHOULD 立即提交一条自动反馈，而不是等 8 秒后的 watchdog incident
    0072 | - **AND** 反馈中 SHOULD 包含触发前的 interaction 快照与可选性诊断，供后续把该交互修回“有解”
    0073 | 
    0074 | #### Scenario: 同一 incident 在冷却窗口内不得重复刷反馈
    0075 | - **GIVEN** 某条 AI recovery incident 已在当前去重键下成功或失败上报过一次
    0076 | - **WHEN** 冷却窗口内再次命中同一 incident key
    0077 | - **THEN** 系统 MUST 跳过重复反馈提交
    0078 | - **AND** 仍可记录本地日志计数，但不得持续向反馈后台刷同一问题
    0079 | 
    0080 | ### Requirement: 前端桥接层必须在服务端接管后降噪
    0081 | 系统 SHALL 将房主页面上的 `OnlineAiSeatBridge` 视为次级 fallback；一旦服务端已接管同一 incident，前端不得持续以失败 toast 轰炸用户。
    0082 | 
    0083 | #### Scenario: 服务端已接管同一 incident 时，前端不再重复提示失败
    0084 | - **GIVEN** 前端桥接层识别到服务端已经推进了同一 AI recovery incident
    0085 | - **WHEN** 原本的客户端 fallback batch 返回 stale failure 或状态已被权威推进
    0086 | - **THEN** 前端 MUST 先同步最新权威状态
    0087 | - **AND** 不得继续重复显示“强制结束 AI 回合未成功，会继续重试”的噪音提示
    0088 | 
    0089 | #### Scenario: 前端仍需展示精确失败原因
    0090 | - **GIVEN** recovery incident 确实在当前阶段失败
    0091 | - **WHEN** 前端向用户展示提示
    0092 | - **THEN** 提示 MUST 能区分至少“阶段（recover-interaction / follow-up-advance）”与“reason（unauthorized / disconnected / command_failed / not_connected）”
    0093 | - **AND** 不得把所有失败都压成同一句模糊文案

## S11

    0001 | # Feature Specification: Support Multi-Statement SQL in mssql_scan
    0002 | 
    0003 | **Feature Branch**: `020-multi-statement-scan`
    0004 | **Created**: 2026-01-27
    0005 | **Status**: Draft
    0006 | **Input**: User description: "Support multi-statement SQL in mssql_scan table function"
    0007 | 
    0008 | ## User Scenarios & Testing *(mandatory)*
    0009 | 
    0010 | ### User Story 1 - Temp Table Queries via mssql_scan (Priority: P1)
    0011 | 
    0012 | A DuckDB user wants to execute multi-statement SQL Server batches through `mssql_scan()` where earlier statements prepare data (e.g., populating temp tables) and a later statement returns the result set. Currently, if the first statement in the batch does not produce columns (e.g., `SELECT INTO`, `INSERT`, `CREATE TABLE`), the function fails with an internal error: "Table function must return at least one column." The user expects the function to skip non-result-producing statements and return the columns from the first statement that produces a result set.
    0013 | 
    0014 | **Why this priority**: This is the core use case — temp table patterns are extremely common in SQL Server workflows. Without this, users cannot use `mssql_scan` for any multi-step data preparation query.
    0015 | 
    0016 | **Independent Test**: Execute `FROM mssql_scan('db', 'select * into #t from dbo.test; select * from #t')` and verify it returns the expected rows and columns.
    0017 | 
    0018 | **Acceptance Scenarios**:
    0019 | 
    0020 | 1. **Given** an attached MSSQL database, **When** a user runs `mssql_scan` with a batch containing `SELECT INTO` followed by `SELECT`, **Then** the result set from the `SELECT` statement is returned with correct columns and data.
    0021 | 2. **Given** an attached MSSQL database, **When** a user runs `mssql_scan` with a batch containing `CREATE TABLE #t (...); INSERT INTO #t ...; SELECT * FROM #t`, **Then** the result set from the final `SELECT` is returned.
    0022 | 3. **Given** an attached MSSQL database, **When** a user runs `mssql_scan` with a single `SELECT` statement (no multi-statement), **Then** behavior is identical to before — no regression.
    0023 | 
    0024 | ---
    0025 | 
    0026 | ### User Story 2 - DML Followed by SELECT (Priority: P2)
    0027 | 
    0028 | A DuckDB user runs a batch where earlier statements perform DML operations (INSERT, UPDATE, DELETE) and a later statement queries the results. The function skips the DML statements and returns the result set from the query.
    0029 | 
    0030 | **Why this priority**: DML-then-query is a common pattern but less frequent than temp table usage.
    0031 | 
    0032 | **Independent Test**: Execute `FROM mssql_scan('db', 'UPDATE dbo.test SET col=1 WHERE id=5; SELECT * FROM dbo.test WHERE id=5')` and verify it returns the updated row.
    0033 | 
    0034 | **Acceptance Scenarios**:
    0035 | 
    0036 | 1. **Given** an attached MSSQL database, **When** a user runs `mssql_scan` with an `UPDATE` followed by a `SELECT`, **Then** the result set from the `SELECT` is returned.
    0037 | 2. **Given** an attached MSSQL database, **When** a user runs `mssql_scan` with an `INSERT` followed by a `SELECT`, **Then** the result set from the `SELECT` is returned.
    0038 | 
    0039 | ---
    0040 | 
    0041 | ### User Story 3 - Error in Intermediate Statement (Priority: P1)
    0042 | 
    0043 | A DuckDB user runs a multi-statement batch where an intermediate statement fails (e.g., referencing a non-existent table). The system reports the error clearly rather than silently skipping it or returning unexpected results.
    0044 | 
    0045 | **Why this priority**: Error handling is critical for correctness — users must know when part of their batch fails.
    0046 | 
    0047 | **Independent Test**: Execute `FROM mssql_scan('db', 'SELECT * FROM nonexistent_table; SELECT 1 AS x')` and verify an error is reported.
    0048 | 
    0049 | **Acceptance Scenarios**:
    0050 | 
    0051 | 1. **Given** an attached MSSQL database, **When** a user runs `mssql_scan` with a batch where an intermediate statement produces an error, **Then** the error is reported to the user with the SQL Server error message.
    0052 | 2. **Given** an attached MSSQL database, **When** all statements in the batch fail, **Then** the first error is reported to the user.
    0053 | 
    0054 | ---
    0055 | 
    0056 | ### User Story 4 - No Result Set in Entire Batch (Priority: P2)
    0057 | 
    0058 | A DuckDB user runs a multi-statement batch where no statement produces a result set (e.g., all DML or DDL). The system returns gracefully rather than hanging or crashing.
    0059 | 
    0060 | **Why this priority**: Edge case handling — users should get a clear response even for pure DML batches.
    0061 | 
    0062 | **Independent Test**: Execute `FROM mssql_scan('db', 'CREATE TABLE #t (x INT); DROP TABLE #t')` and verify the function returns gracefully.
    0063 | 
    0064 | **Acceptance Scenarios**:
    0065 | 
    0066 | 1. **Given** an attached MSSQL database, **When** a user runs `mssql_scan` with a batch where no statement returns columns, **Then** the function completes without error and returns an empty result set (zero columns, zero rows).
    0067 | 
    0068 | ---
    0069 | 
    0070 | ### User Story 5 - Connection Cleanup on Pool Return (Priority: P1)
    0071 | 
    0072 | When a connection in autocommit mode is returned to the pool after executing a query, session-scoped artifacts (temp tables, session variables, SET options, open cursors) persist on the connection. The next user of that pooled connection would inherit these artifacts, causing unexpected behavior. The system must reset the connection state before returning it to the pool.
    0073 | 
    0074 | **Important**: Connection reset MUST only happen on pool return in autocommit mode. Connections pinned to an explicit transaction MUST NOT be reset — they retain their session state until the transaction is committed or rolled back. The reset happens after commit/rollback, when the connection is finally released back to the pool.
    0075 | 
    0076 | **Why this priority**: Without cleanup, temp tables from one query leak into the next user's session — a correctness and security issue. The standard SQL Server mechanism for this is a session reset command used by ADO.NET, JDBC, and ODBC drivers.
    0077 | 
    0078 | **Independent Test**: Execute `mssql_scan('db', 'select * into #t from dbo.test; select * from #t')`, then on the next query using the same pooled connection, verify that `#t` does not exist.
    0079 | 
    0080 | **Acceptance Scenarios**:
    0081 | 
    0082 | 1. **Given** a pooled connection in autocommit mode that just executed a batch creating temp tables, **When** the connection is returned to the pool, **Then** session state is reset (temp tables dropped, variables cleared, SET options restored to defaults).
    0083 | 2. **Given** a pooled connection in autocommit mode that just executed a batch with `SET NOCOUNT ON`, **When** a different user acquires that connection from the pool, **Then** `NOCOUNT` is back to its default (OFF).
    0084 | 3. **Given** a connection pinned to an explicit transaction, **When** the connection is used for multiple queries within the transaction, **Then** session state (temp tables, variables) is preserved throughout the transaction.
    0085 | 4. **Given** a connection pinned to an explicit transaction, **When** the transaction is committed or rolled back and the connection is returned to the pool, **Then** session state is reset before the connection becomes available to other users.
    0086 | 5. **Given** a pooled connection, **When** the reset operation itself fails, **Then** the connection is discarded from the pool rather than returned in a dirty state.
    0087 | 
    0088 | ---
    0089 | 
    0090 | ### Edge Cases
    0091 | 
    0092 | - What happens when the result-producing statement is in the middle of the batch (not the last statement)? The system should return columns from the first result-producing statement.
    0093 | - What happens with very large batches (10+ statements)? The system should handle them without timeout issues for the non-result statements.
    0094 | - What happens when a single-statement DML query (e.g., `UPDATE ...`) is passed to `mssql_scan`? Behavior should remain unchanged — empty result, no error.
    0095 | - What happens when the batch includes `SET NOCOUNT ON`? The system should handle it correctly since `SET NOCOUNT ON` suppresses row count messages.
    0096 | - What happens when the connection reset itself fails (e.g., broken connection)? The connection should be discarded from the pool, not returned dirty.
    0097 | 
    0098 | ## Requirements *(mandatory)*
    0099 | 
    0100 | ### Functional Requirements
    0101 | 
    0102 | - **FR-001**: When the system receives a multi-statement batch, it MUST continue processing protocol responses beyond the first non-result-producing statement if more results are expected.
    0103 | - **FR-002**: The system MUST return columns and data from the first statement in the batch that produces a result set (has column metadata).
    0104 | - **FR-003**: If an error occurs in any statement before a result set is found, the system MUST report that error to the user with the server-provided error message and error number.
    0105 | - **FR-004**: If no statement in the batch produces a result set and no errors occur, the system MUST return an empty result without crashing or hanging.
    0106 | - **FR-005**: Single-statement queries that return columns MUST behave identically to before this change — zero regression.
    0107 | - **FR-006**: Single-statement DML queries (INSERT/UPDATE/DELETE) that return no columns MUST behave identically to before this change.
    0108 | - **FR-007**: The system MUST correctly handle intermediate protocol messages (informational messages, environment changes, row counts) between statements without treating them as errors.
    0109 | - **FR-008**: When a connection in autocommit mode is returned to the pool, the system MUST reset the connection's session state to clear temp tables, session variables, SET options, and other session-scoped artifacts. Connections pinned to an explicit transaction MUST NOT be reset until after commit or rollback.
    0110 | - **FR-009**: If the connection reset fails, the system MUST discard the connection from the pool rather than returning it in a dirty state.
    0111 | - **FR-010**: After an explicit transaction is committed or rolled back, the connection MUST be reset before being returned to the pool.
    0112 | 
    0113 | ## Success Criteria *(mandatory)*
    0114 | 
    0115 | ### Measurable Outcomes
    0116 | 
    0117 | - **SC-001**: Users can execute `SELECT INTO` followed by `SELECT` via `mssql_scan` and receive correct results — zero failures for this common pattern.
    0118 | - **SC-002**: All existing tests continue to pass with zero regressions (108 test cases, 2741 assertions).
    0119 | - **SC-003**: Errors in intermediate statements are reported with the original SQL Server error message — users see actionable error information.
    0120 | - **SC-004**: Batches with no result-producing statements complete without crashes or hangs.
    0121 | - **SC-005**: Temp tables created in one query do not leak to subsequent queries on the same pooled connection — session state is fully reset between uses.
    0122 | 
    0123 | ## Assumptions
    0124 | 
    0125 | - SQL Server sends a completion marker with a "more results" flag between statements in a batch. The final statement's completion marker does not have this flag.
    0126 | - The existing protocol parser already distinguishes between final and non-final completion markers.
    0127 | - The `mssql_exec` function (for executing DML without returning results) is not affected by this change, but may benefit from the same underlying improvement.
    0128 | - Temp tables created in a batch are visible to subsequent statements in the same batch because they share the same connection and session.

## S12

    0001 | ## ADDED Requirements
    0002 | 
    0003 | ### Requirement: Selected session card auto-scrolls into view on background re-sort
    0004 | The session list SHALL automatically scroll the currently selected session card into view ONLY when the card moves under the user due to a background state change of an unchanged selection. The scroll SHALL be a no-op when the card is already within the visible viewport.
    0005 | 
    0006 | The set of position-affecting state changes that SHALL trigger a scroll is:
    0007 | 
    0008 | - A change to the selected session's `status` field (e.g. `active` → `ended`, `ended` → `streaming`).
    0009 | - A change to the selected session's `hidden` flag.
    0010 | - A change to the selected session's `cwd` (folder reassignment).
    0011 | - A change to the selected session's index inside its folder's `sessionOrderMap` slice.
    0012 | 
    0013 | The system SHALL NOT scroll when:
    0014 | 
    0015 | - The user clicks a different session card (or `selectedId` changes by any other mechanism after first mount). The user already knows where the card they clicked is located, and auto-scrolling would hijack their scroll position.
    0016 | - A field of the selected session changes that does not affect the card's position (`currentTool`, `tokensIn`, `tokensOut`, `cost`, `model`, etc.).
    0017 | - The selected card has no DOM element at the time the trigger fires (filtered out by search, in a collapsed folder, in a collapsed Ended bucket, in a collapsed Hidden disclosure). The system SHALL noop and SHALL NOT auto-expand any collapsed container.
    0018 | - No session is selected.
    0019 | 
    0020 | The scroll alignment SHALL be `block: "nearest"` so that already-visible cards do not scroll. The scroll behavior SHALL be `"auto"` (instant).
    0021 | 
    0022 | #### Scenario: Selected session ends and is re-sorted
    0023 | - **WHEN** the currently selected session's `status` flips to `"ended"` and a re-sort moves its card to a different position in the list
    0024 | - **THEN** the session list SHALL scroll the card into view with `behavior: "auto"` and `block: "nearest"`
    0025 | 
    0026 | #### Scenario: Selected session resumed from ended
    0027 | - **WHEN** the currently selected session's `status` flips from `"ended"` back to an alive value
    0028 | - **THEN** the session list SHALL scroll the card into view with `behavior: "auto"`
    0029 | 
    0030 | #### Scenario: Selected session toggled hidden
    0031 | - **WHEN** the currently selected session's `hidden` flag flips while it remains the selected session
    0032 | - **THEN** the session list SHALL scroll the card into view with `behavior: "auto"` if and only if the card is still in the DOM
    0033 | - **AND** the system SHALL NOT auto-expand the Hidden disclosure when the card has been moved into it
    0034 | 
    0035 | #### Scenario: Selected session reordered by reattach policy or drag
    0036 | - **WHEN** the bridge re-registers and the configured `reattachPlacement` policy (or another reorder) changes the selected card's position in `sessionOrderMap`
    0037 | - **THEN** the session list SHALL scroll the card into view with `behavior: "auto"`
    0038 | 
    0039 | #### Scenario: User clicks a different session card — no scroll
    0040 | - **WHEN** the user clicks a session card whose id differs from the current `selectedId`
    0041 | - **THEN** the session list SHALL NOT scroll
    0042 | - **AND** the prev-selectedId tracking ref SHALL be updated so that subsequent background re-sorts of the new selection scroll correctly
    0043 | 
    0044 | #### Scenario: Programmatic selection switch — no scroll
    0045 | - **WHEN** `selectedId` changes for any reason other than initial mount (e.g. App-level navigation, keyboard shortcut, clearing selection)
    0046 | - **THEN** the session list SHALL NOT scroll
    0047 | 
    0048 | #### Scenario: Selected card already visible
    0049 | - **WHEN** any qualifying state change fires AND the selected card is already within the visible viewport of the session list scroll container
    0050 | - **THEN** the system SHALL noop (no observable scroll movement) — guaranteed by `block: "nearest"`
    0051 | 
    0052 | #### Scenario: Selected card filtered out by search
    0053 | - **WHEN** the selected session is filtered out by `sessionSearch` or `workspaceFilter` and has no DOM element in the list
    0054 | - **THEN** the system SHALL noop and SHALL NOT clear the search filter
    0055 | 
    0056 | #### Scenario: Selected card inside collapsed folder or bucket
    0057 | - **WHEN** the selected card has been moved into a collapsed folder, a collapsed Ended bucket, or a collapsed Hidden disclosure and has no DOM element in the list
    0058 | - **THEN** the system SHALL noop and SHALL NOT auto-expand the collapsed container
    0059 | 
    0060 | #### Scenario: Non-position-affecting field changes
    0061 | - **WHEN** a field of the selected session changes that does not affect the card's position (for example `currentTool`, `tokensIn`, `tokensOut`, `cost`, or `model`)
    0062 | - **THEN** the system SHALL NOT scroll
    0063 | 
    0064 | #### Scenario: Selected session unregistered
    0065 | - **WHEN** the selected session is unregistered and removed from the `sessions` array
    0066 | - **THEN** the system SHALL noop (no scroll attempted on a missing card)
    0067 | 
    0068 | ### Requirement: One-shot deep-link scroll on first mount
    0069 | The session list SHALL scroll the currently selected card into view exactly once on first mount, if a `selectedId` is set at that time. This handles deep-link arrival on `/session/:id` where the user has not had a chance to scroll the list manually.
    0070 | 
    0071 | After the first mount, subsequent `selectedId` changes SHALL NOT trigger a scroll (per the previous requirement).
    0072 | 
    0073 | #### Scenario: Deep-link arrival on /session/:id
    0074 | - **WHEN** the dashboard mounts with `selectedId` already set (for example via a `/session/<id>` route) and the matching card is in the DOM
    0075 | - **THEN** the session list SHALL scroll the card for that id into view exactly once with `behavior: "auto"` and `block: "nearest"`
    0076 | 
    0077 | #### Scenario: Mount without selection
    0078 | - **WHEN** the dashboard mounts with `selectedId === undefined`
    0079 | - **THEN** the session list SHALL NOT scroll on mount
    0080 | - **AND** subsequent user clicks SHALL NOT trigger a scroll either
    0081 | 
    0082 | #### Scenario: First-mount deep-link card not yet in DOM
    0083 | - **WHEN** the dashboard mounts with `selectedId` set but the matching card is not in the DOM (for example because the session list has not finished its first render of the relevant folder, or the card is filtered out)
    0084 | - **THEN** the system SHALL noop on the mount-time scroll attempt
    0085 | 
    0086 | ### Requirement: Selected card scroll fingerprint helper
    0087 | The session list SHALL derive its scroll trigger from a pure helper function `selectedCardScrollFingerprint(selectedId, sessions, sessionOrderMap)` that returns a stable string when the position-affecting inputs are unchanged and a different string when any of them change.
    0088 | 
    0089 | The helper SHALL return `null` when `selectedId` is undefined OR when no session in `sessions` matches `selectedId`.
    0090 | 
    0091 | The helper SHALL be exported so it can be unit-tested without rendering the component.
    0092 | 
    0093 | The helper alone is insufficient to suppress click-driven scrolls — that suppression is the consumer's responsibility, implemented via a `useRef` that tracks the previous `selectedId`. The helper's contract is purely about producing a position-aware change signal.
    0094 | 
    0095 | #### Scenario: Helper returns null without selection
    0096 | - **WHEN** `selectedCardScrollFingerprint` is called with `selectedId === undefined`
    0097 | - **THEN** the helper SHALL return `null`
    0098 | 
    0099 | #### Scenario: Helper returns null when session not found
    0100 | - **WHEN** `selectedCardScrollFingerprint` is called with a `selectedId` not present in `sessions`
    0101 | - **THEN** the helper SHALL return `null`
    0102 | 
    0103 | #### Scenario: Helper produces stable string for unchanged inputs
    0104 | - **WHEN** `selectedCardScrollFingerprint` is called twice with the same `selectedId`, an unchanged session object, and an unchanged `sessionOrderMap` slice for that session's `cwd`
    0105 | - **THEN** the helper SHALL return identical strings
    0106 | 
    0107 | #### Scenario: Helper string changes when status flips
    0108 | - **WHEN** the selected session's `status` changes between two calls
    0109 | - **THEN** the helper's two return values SHALL differ
    0110 | 
    0111 | #### Scenario: Helper string changes when hidden toggles
    0112 | - **WHEN** the selected session's `hidden` flag changes between two calls
    0113 | - **THEN** the helper's two return values SHALL differ
    0114 | 
    0115 | #### Scenario: Helper string changes when cwd changes
    0116 | - **WHEN** the selected session's `cwd` changes between two calls
    0117 | - **THEN** the helper's two return values SHALL differ
    0118 | 
    0119 | #### Scenario: Helper string changes when order index changes
    0120 | - **WHEN** the selected session's index inside `sessionOrderMap.get(session.cwd)` changes between two calls
    0121 | - **THEN** the helper's two return values SHALL differ
    0122 | 
    0123 | #### Scenario: Helper string is stable for non-position-affecting changes
    0124 | - **WHEN** only `currentTool`, `tokensIn`, `tokensOut`, `cost`, `model`, or any field other than `status`, `hidden`, `cwd`, or order-index changes between two calls
    0125 | - **THEN** the helper's two return values SHALL be identical
    0126 | 
    0127 | ### Requirement: Selected card DOM addressing
    0128 | Each rendered session card root element SHALL carry a `data-session-id` attribute equal to the session's id, so that the auto-scroll effect can locate it via attribute selector without ref forwarding through `dnd-kit`'s `useSortable` wrapper.
    0129 | 
    0130 | #### Scenario: Card root carries data-session-id
    0131 | - **WHEN** any session card is rendered in the session list
    0132 | - **THEN** the card's root element SHALL have an attribute `data-session-id` equal to the session's `id`

## S13

    0001 | # Spec 012: editor reload state
    0002 | 
    0003 | ## Goal
    0004 | 
    0005 | Keep users in the editor after a browser refresh and restore the local board
    0006 | draft in the same browser session.
    0007 | 
    0008 | ## Scope
    0009 | 
    0010 | - Encode the active editor view in the static app URL.
    0011 | - Open the editor automatically when the page loads with the editor route.
    0012 | - Clear the editor route when the user intentionally returns to the landing
    0013 |   view.
    0014 | - Continue relying on local draft persistence for canvas objects and uploaded
    0015 |   images.
    0016 | - Keep editor interactions locked while the local draft restore is still
    0017 |   pending.
    0018 | - Document the editor route and draft restore contract.
    0019 | 
    0020 | ## Non-Goals
    0021 | 
    0022 | - No backend draft storage or account state.
    0023 | - No framework/router migration.
    0024 | - No new visible draft status UI.
    0025 | - No export/download behavior changes.
    0026 | 
    0027 | ## Acceptance Criteria
    0028 | 
    0029 | 1. Opening the editor updates the URL to an editor route.
    0030 | 2. Refreshing while on the editor route returns to the editor instead of the
    0031 |    landing view.
    0032 | 3. Refreshing while on the editor route does not flash the landing view before
    0033 |    the editor appears.
    0034 | 4. A saved local draft restores uploaded images and text objects after refresh.
    0035 | 5. Editor controls cannot mutate the canvas before draft restore completes.
    0036 | 6. Returning to the landing view clears the editor route so future reloads stay
    0037 |    on the landing view.
    0038 | 7. Browser back/forward navigation switches between landing and editor routes.
    0039 | 8. `pnpm run ci` passes.

## S14

    0001 | ## ADDED Requirements
    0002 | 
    0003 | ### Requirement: Project-aware operations reference page exists
    0004 | 
    0005 | The agents reference SHALL include a page at `docs/reference/agents/operations/project-aware-operations.md` documenting how managed-agent commands resolve against the active project overlay. The page SHALL explain:
    0006 | 
    0007 | - What "project-aware" means: commands like `agents launch`, `agents list`, `agents state`, and others automatically discover and use the active `.houmao/` project overlay for agent definition resolution, mailbox root binding, and registry scoping.
    0008 | - Resolution order: explicit `--agent-def-dir` → `HOUMAO_AGENT_DEF_DIR` → `HOUMAO_PROJECT_OVERLAY_DIR` → nearest ancestor `.houmao/houmao-config.toml` → default `<cwd>/.houmao/agents`.
    0009 | - The `HOUMAO_PROJECT_DIR` environment variable as an override for selecting the project root in CI and automation contexts.
    0010 | - The catalog-backed overlay storage model: how `project/catalog.py` and `project/overlay.py` resolve the overlay directory and provide `ProjectAwareLocalRoots`.
    0011 | - Which commands are project-aware and what project context they consume (agent definitions, mailbox root, registry scoping).
    0012 | 
    0013 | The page SHALL be derived from `project/overlay.py`, `project/catalog.py`, and the project-aware initialization in `srv_ctrl/commands/`.
    0014 | 
    0015 | #### Scenario: Reader understands project-aware resolution
    0016 | 
    0017 | - **WHEN** a reader opens the project-aware operations page
    0018 | - **THEN** they find the full resolution precedence chain for agent definition directory discovery
    0019 | - **AND THEN** they understand that commands automatically discover the nearest `.houmao/` overlay without explicit flags
    0020 | 
    0021 | #### Scenario: Reader can override project resolution for CI
    0022 | 
    0023 | - **WHEN** a reader needs to run commands in a CI environment without `.houmao/` on disk
    0024 | - **THEN** the page documents `HOUMAO_PROJECT_OVERLAY_DIR` and `HOUMAO_AGENT_DEF_DIR` as environment overrides
    0025 | - **AND THEN** the page explains when each override is appropriate
    0026 | 
    0027 | #### Scenario: Reader understands which commands are project-aware
    0028 | 
    0029 | - **WHEN** a reader wants to know which commands use project context
    0030 | - **THEN** the page lists the major project-aware command families: `agents launch`, `agents join`, `brains build`, `agents list`, `agents state`, and project-scoped mailbox operations

## S15

    0001 | # UI cleanup: distinct sidebar install icon + ordered title-bar icons; Create-Spec touchups
    0002 | 
    0003 | ## Overview
    0004 | 
    0005 | Tighten two surfaces found while testing. In the SpecKit sidebar title bar, give the install action a glyph distinct from the upgrade action and reorder the title-bar actions into sensible groups with dividers. In the Create-Spec webview, remove the unused Load Template control entirely, shorten the turbo workflow option label to a single word while preserving its full description on hover, and make the panel wrap gracefully at narrow widths instead of overflowing horizontally.
    0006 | 
    0007 | ## Functional Requirements
    0008 | 
    0009 | - **FR-001**: The sidebar install action MUST use a codicon visually distinct from the upgrade action's codicon, so the two are distinguishable when the install state toggles. (Title-bar icons are monochrome-themed; distinctness is by glyph, not color.)
    0010 | - **FR-002**: The `view/title` actions for the specs explorer view MUST be ordered into distinct menu groups so VS Code renders dividers between them: create, list-view controls (filter, clear-filter, sort, collapse/expand), and install/upgrade.
    0011 | - **FR-003**: The Load Template button MUST be removed from the Create-Spec panel along with every supporting artifact: the button markup, the `requestTemplateDialog` / `loadTemplate` / `templateLoaded` message types, the extension-side handler methods, the webview click wiring and message handling, the CSS for the button/loader, and the Storybook/mock representation.
    0012 | - **FR-004**: Removing Load Template MUST NOT leave dead code, unreferenced message types, orphaned commands, unused CSS classes, or stale mocks.
    0013 | - **FR-005**: The turbo workflow picker option's visible label MUST read `Turbo` (with the `(beta)` suffix omitted from the visible label), while the full descriptive text MUST remain available as the option's hover tooltip/description.
    0014 | - **FR-006**: The turbo workflow option value MUST remain `speckit-turbo` (unchanged).
    0015 | - **FR-007**: The `.workflow-row` and `.image-attachment-header` layouts MUST wrap (rather than overflow) at narrow viewport widths, using `flex-wrap` and `min-width:0`/ellipsis where a child can overflow; the responsive breakpoint MUST cover these rows, not only the footer actions.
    0016 | 
    0017 | ## Success Criteria
    0018 | 
    0019 | - **SC-001**: The install and upgrade title-bar actions render with different glyphs (verifiable by inspecting the two `icon` codicon ids in `package.json`).
    0020 | - **SC-002**: The specs explorer `view/title` entries resolve to at least three distinct `navigation@N` group numbers, producing dividers between create, list controls, and install/upgrade.
    0021 | - **SC-003**: A repository-wide search for `loadTemplate`, `requestTemplateDialog`, `templateLoaded`, `LoadTemplate`, `load-template`, and `Load Template` returns zero matches in shipping source, styles, mocks, and `package.json`.
    0022 | - **SC-004**: The turbo option's visible text is exactly `Turbo`; its tooltip/description contains the full descriptive sentence.
    0023 | - **SC-005**: `npm run compile` and `npm test` pass (excluding the 6 known pre-existing Python `test_context.py` failures unrelated to this change).
    0024 | - **SC-006**: At a narrow panel width the Create-Spec workflow row and image-attachment header wrap onto multiple lines with no horizontal overflow.
    0025 | 
    0026 | ## Assumptions
    0027 | 
    0028 | - The chosen distinct install glyph is `$(desktop-download)` (a download-to-disk glyph that reads differently from the upgrade `$(cloud-download)`).
    0029 | - "Full description on hover" is satisfied by the existing `<option title="...">` rendering path; only the option's `description` text needs to carry the full label, since the visible `displayName` becomes `Turbo`.
    0030 | - No automated test currently asserts on the Load Template message types or the turbo `displayName`, so their removal/change requires no test rewrite (verified by grep before editing).
    0031 | - The Storybook representation of Load Template lives only in the `CreateSpecMock` component and is not asserted by any test.

## S16

    0001 | ## MODIFIED Requirements
    0002 | ### Requirement: Face Feature Enrollment Command
    0003 | The CLI MUST manage descriptor encryption keys via Secret Service when enrolling descriptors, rotating them on every run, and writing the per-user store in encrypted form.
    0004 | 
    0005 | #### Scenario: Enrollment creates or rotates AES-GCM key
    0006 | - **WHEN** `chissu-cli faces enroll --user <name>` runs
    0007 | - **THEN** the command fetches the existing AES-GCM descriptor key for `<name>` from Secret Service (service `chissu-pam`)
    0008 | - **AND** if a key exists it decrypts the user’s current store before appending descriptors
    0009 | - **AND** the command generates a new 32-byte AES-256-GCM key, registers it in Secret Service, and encrypts the updated store with that key before exiting.
    0010 | 
    0011 | #### Scenario: Secret Service errors abort enrollment
    0012 | - **WHEN** Secret Service is locked/unavailable or returns an invalid/malformed key
    0013 | - **THEN** the enroll command logs the failure, exits non-zero, and leaves both the feature store and the previously registered key untouched so operators can retry safely.
    0014 | 
    0015 | #### Scenario: Legacy plaintext store migration
    0016 | - **WHEN** a user’s feature store is still plaintext (no AES key registered)
    0017 | - **THEN** the first enrollment run accepts the legacy file, generates/registers a new key, and re-writes the store using the encrypted format so subsequent PAM authentications can decrypt it via the helper key.
    0018 | 
    0019 | ### Requirement: Face Feature Removal Command
    0020 | The removal flow MUST reuse the encrypted store format and Secret Service key so descriptors remain protected when entries are deleted.
    0021 | 
    0022 | #### Scenario: Removal decrypts and re-encrypts store
    0023 | - **WHEN** `chissu-cli faces remove` deletes descriptors for a user with an encrypted store
    0024 | - **THEN** it fetches the user’s AES-GCM key from Secret Service, decrypts the store, removes the requested descriptors, and rewrites the store encrypted with the same key before exiting.
    0025 | 
    0026 | #### Scenario: Missing key detection
    0027 | - **WHEN** the removal command encounters an encrypted store but cannot obtain the Secret Service key
    0028 | - **THEN** it fails with a descriptive error instructing the operator to unlock Secret Service (or rerun enroll) instead of corrupting or rewriting the store.

## S17

    0001 | # Feature Specification: Observability — Usage Statistics & Graphs in the Web UI
    0002 | 
    0003 | **Feature Branch**: `069-observability-usage-graphs`
    0004 | **Created**: 2026-05-31
    0005 | **Status**: Draft
    0006 | **Lineage**: H2-2026 roadmap PILLAR A (Adopt / usability). Builds on the activity log (specs 016/019/024). Paperclip goal `d7164fdf`.
    0007 | **Input**: Help users SEE where their AI agents spend tokens and time, so they can speed up MCP usage and cut token cost. Surface usage statistics as graphs in the Web UI — tool-call histogram (calls + tokens per tool), response-size-by-tool, tool errors, a tool-call timeline — behind a dashboard switcher, with a fast backend (caching / precomputation).
    0008 | 
    0009 | ## Clarifications
    0010 | 
    0011 | ### Session 2026-05-31
    0012 | 
    0013 | - Q: Build new telemetry, or surface existing data? → A: Surface existing data. The activity log already records per-call server, tool, status, duration, and request/response byte sizes; this feature aggregates and visualizes it.
    0014 | - Q: Tokens per tool — do we have them? → A: Not per call. Bytes are recorded; tokens are not. v1 uses response **bytes** as the token proxy and shows the existing "tokens saved by retrieve_tools" metric; accurate per-call token estimation is an explicit later requirement.
    0015 | - Q: Where do the graphs live? → A: On the existing Dashboard, behind a switcher/tab ("Overview" ↔ "Usage"), reusing the activity REST API.
    0016 | - Q: How is fast response guaranteed? → A: An incrementally-maintained, actor-owned stats aggregate updated on each activity write (O(1) per write/read), periodically persisted, with a short-TTL cache for wide windows — not an on-demand full scan.
    0017 | - Q: Charting library? → A: None is installed today; the plan selects one lightweight Vue-compatible library (decision recorded in plan.md). The spec stays library-agnostic.
    0018 | 
    0019 | ## User Scenarios & Testing *(mandatory)*
    0020 | 
    0021 | ### User Story 1 — See which tools consume the most tokens (Priority: P1)
    0022 | 
    0023 | A user whose agent burns through context opens the Web UI dashboard, switches to the Usage view, and immediately sees which tools return the largest responses (the "token sinks") and which are called most often. They learn that one tool returns huge blobs on every call, and adjust how their agent uses it — saving tokens.
    0024 | 
    0025 | **Why this priority**: This is the core value — "where do my tokens go?" — and it answers the user's primary goal (save tokens / speed up MCP). It is buildable entirely on data already captured (response bytes + call counts), so it delivers the headline insight with no new capture.
    0026 | 
    0027 | **Independent Test**: With a populated activity log, open the Usage view; confirm a per-tool ranking by total + average response size and a per-tool call-count histogram render, ordered correctly, matching the underlying records.
    0028 | 
    0029 | **Acceptance Scenarios**:
    0030 | 
    0031 | 1. **Given** recorded tool calls with response sizes, **When** the user opens the Usage view, **Then** tools are shown ranked by total response bytes (the token-sink view) and by call count.
    0032 | 2. **Given** the same data, **When** the user inspects a tool, **Then** its average and total response size and call count are shown.
    0033 | 3. **Given** no activity yet, **When** the user opens the Usage view, **Then** an empty-state is shown (no error).
    0034 | 
    0035 | ---
    0036 | 
    0037 | ### User Story 2 — Spot failing and slow tools (Priority: P1)
    0038 | 
    0039 | A user sees an error-rate breakdown per tool and per-tool latency, so they can identify tools that fail often (wasting agent round-trips) or respond slowly (slowing the agent), and fix or replace them.
    0040 | 
    0041 | **Why this priority**: Errors and latency directly waste tokens and time — a tool that fails half the time doubles the agent's round-trips. Same data is already captured (status, error, duration). Equal priority to US1 as the other half of "speed up MCP usage."
    0042 | 
    0043 | **Independent Test**: With a log containing successes and errors across tools, confirm an error-rate-per-tool graph and a latency (e.g. p50/p95) view render and match the records.
    0044 | 
    0045 | **Acceptance Scenarios**:
    0046 | 
    0047 | 1. **Given** tool calls with success/error statuses, **When** the user opens the Usage view, **Then** error rate (and count) per tool is shown.
    0048 | 2. **Given** calls with recorded durations, **When** viewed, **Then** per-tool latency (median and a high percentile) is shown.
    0049 | 
    0050 | ---
    0051 | 
    0052 | ### User Story 3 — See activity over time on a timeline (Priority: P2)
    0053 | 
    0054 | A user views a timeline of tool calls (volume over time, sliceable by tool/server/status) to understand usage patterns and correlate spikes with their agent sessions — an at-a-glance companion to the activity log.
    0055 | 
    0056 | **Why this priority**: Valuable for understanding *when* and *in what bursts* tokens are spent, and a natural visual layer over the existing activity log. P2 because US1/US2 deliver the primary "what's expensive" insight first.
    0057 | 
    0058 | **Independent Test**: With time-distributed records, confirm a timeline/volume graph renders with correct buckets and respects the active filters.
    0059 | 
    0060 | **Acceptance Scenarios**:
    0061 | 
    0062 | 1. **Given** records across a time window, **When** the user opens the timeline, **Then** call volume is shown bucketed over time.
    0063 | 2. **Given** an active filter (tool/server/status), **When** applied, **Then** the timeline reflects only matching calls.
    0064 | 
    0065 | ---
    0066 | 
    0067 | ### User Story 4 — Toggle graphs without losing the existing dashboard (Priority: P2)
    0068 | 
    0069 | A user switches between the current status/overview dashboard and the new usage graphs via a clear switcher, and can pick a time window (e.g. 24h / 7d / all). The graphs load fast enough not to delay the dashboard's first paint.
    0070 | 
    0071 | **Why this priority**: The switcher + windowing is what makes the graphs usable day-to-day rather than a separate page; the performance bar is what keeps the dashboard snappy. P2 because the graphs (US1–3) must exist first.
    0072 | 
    0073 | **Independent Test**: Confirm the switcher toggles Overview ↔ Usage, the window selector changes the data range, and the dashboard's initial render is not blocked waiting on graph data.
    0074 | 
    0075 | **Acceptance Scenarios**:
    0076 | 
    0077 | 1. **Given** the dashboard, **When** the user toggles to Usage, **Then** the graphs appear and Overview state is preserved on switch-back.
    0078 | 2. **Given** a window selector, **When** the user picks 7d, **Then** all graphs reflect that window.
    0079 | 
    0080 | ---
    0081 | 
    0082 | ### User Story 5 — Headline "tokens saved by mcpproxy" (Priority: P3)
    0083 | 
    0084 | The dashboard shows a hero number: tokens saved by serving tool discovery through `retrieve_tools` instead of exposing all tools — making mcpproxy's core value visible.
    0085 | 
    0086 | **Why this priority**: Strong motivational/marketing metric and the data partially exists (token-savings metrics). P3 because it draws on a different (discovery-side) data source than the per-call graphs and is a single number, not a graph suite.
    0087 | 
    0088 | **Independent Test**: Confirm the saved-tokens figure renders from the existing token-savings metrics and updates with usage.
    0089 | 
    0090 | **Acceptance Scenarios**:
    0091 | 
    0092 | 1. **Given** discovery token-savings metrics, **When** the dashboard loads, **Then** a tokens-saved figure is shown.
    0093 | 
    0094 | ## Requirements *(mandatory)*
    0095 | 
    0096 | ### Context & Constraints (locked)
    0097 | 
    0098 | - **CN-001**: Surface and aggregate EXISTING activity-log data; do not add a parallel telemetry system. (External telemetry, spec 042, is separate and unaffected.)
    0099 | - **CN-002**: The graphs backend MUST be fast — incremental precompute + cache, not an on-demand full scan of the log on every request. It MUST NOT block the dashboard's first paint or the request hot path (Constitution I).
    0100 | - **CN-003**: Aggregation MUST follow actor-based concurrency (Constitution II) — owned by the activity service, updated on write, snapshot for readers.
    0101 | - **CN-004**: All data stays local (no new external calls). Privacy: graphs show the user their own usage; any sensitive-data flags already in the activity log are not exposed in aggregates.
    0102 | 
    0103 | ### Functional Requirements
    0104 | 
    0105 | - **FR-001**: The system MUST provide a per-tool usage aggregation: for each tool (and server), total calls, total + average response size, total + average request size, error count/rate, and latency (median + a high percentile), over a selectable time window.
    0106 | - **FR-002**: The Web UI MUST present at minimum four visualizations: (a) per-tool call histogram, (b) per-tool response-size ranking (token-sink view), (c) per-tool error rate, (d) a tool-call timeline — all reading the aggregation.
    0107 | - **FR-003**: The Web UI MUST provide a dashboard switcher between the existing overview and the usage graphs, preserving overview state.
    0108 | - **FR-004**: The system MUST support time-window selection (at least 24h, 7d, all) applied consistently across all graphs.
    0109 | - **FR-005**: The aggregation MUST be served fast via incremental precomputation maintained on each activity write and/or a short-TTL cache, with a stated freshness bound; cold start MUST rebuild from the log.
    0110 | - **FR-006**: v1 MUST use response bytes as the token proxy for the token-spend graphs and clearly label them as size-based.
    0111 | - **FR-007**: The system MUST display the existing "tokens saved by retrieve_tools / discovery" metric as a headline figure.
    0112 | - **FR-008**: Graphs MUST honor filtering by tool, server, and status consistent with the existing activity log filters.
    0113 | - **FR-009**: Empty/low-data states MUST render gracefully (no errors, clear "no data yet").
    0114 | - **FR-010**: (Phase 2) The system SHOULD capture an estimated token count per call (request and response) and switch the token graphs from bytes to estimated tokens, recorded as an explicit follow-on requirement.
    0115 | - **FR-011**: The graphs endpoint MUST be covered by API tests, and the UI by the project's Web-UI verification workflow (Playwright + report).
    0116 | 
    0117 | ### Key Entities
    0118 | 
    0119 | - **Tool usage aggregate**: per (server, tool) rollup — counts, byte sums/averages, error count, latency percentiles — over a window.
    0120 | - **Time bucket**: pre-bucketed call counts/sizes over time for the timeline.
    0121 | - **Tokens-saved figure**: the discovery-side savings metric surfaced on the dashboard.
    0122 | - **Usage view + switcher**: the dashboard toggle, window selector, and graph components.
    0123 | 
    0124 | ## Success Criteria *(mandatory)*
    0125 | 
    0126 | - **SC-001**: A user can identify their top token-consuming (largest-response) tools within one view, without reading raw logs.
    0127 | - **SC-002**: A user can see error rate and latency per tool in the same view.
    0128 | - **SC-003**: The timeline shows call volume over time and respects active filters.
    0129 | - **SC-004**: The usage graphs do not measurably delay the dashboard's first paint (graphs load asynchronously).
    0130 | - **SC-005**: The aggregation endpoint returns within a fast bound on a large activity log (precompute/cache proven by not scanning the full log per request).
    0131 | - **SC-006**: Switching Overview ↔ Usage and changing the time window updates all graphs consistently.
    0132 | - **SC-007**: With an empty log, the Usage view shows a clean empty state.
    0133 | - **SC-008**: The tokens-saved headline renders from existing metrics.
    0134 | 
    0135 | ## Assumptions
    0136 | 
    0137 | - `ActivityRecord` already carries timestamp, type, server, tool, status, error, duration, and request/response byte sizes (confirmed in the activity-log backend); no new per-call capture is needed for v1 graphs.
    0138 | - A `GET /api/v1/activity/stats` endpoint and an `Activity.vue` view already exist and can be extended rather than replaced.
    0139 | - No charting library is currently a frontend dependency; adding one lightweight library is acceptable (recorded in plan.md). Frontend changes require `make build` (the UI is embedded).
    0140 | - Accurate per-call token counts are out of scope for v1 (bytes proxy); FR-010 tracks the upgrade.
    0141 | 
    0142 | ## Dependencies
    0143 | 
    0144 | - The activity-log backend (`internal/runtime/activity_service.go`, `internal/storage/activity.go`, `internal/storage/models.go`) and its REST surface (`internal/httpapi/activity.go`).
    0145 | - The existing token-savings metrics (`internal/config/config.go` TokenMetrics / discovery path).
    0146 | - The Vue Web UI (`frontend/src/views/Dashboard.vue`, `Activity.vue`) + a charting library.
    0147 | 
    0148 | ## Out of Scope
    0149 | 
    0150 | - A new/parallel telemetry pipeline (external telemetry is spec 042).
    0151 | - Accurate per-call token counting (FR-010 is the phase-2 hook; not v1).
    0152 | - Cost-in-currency estimates (needs token + pricing; later).
    0153 | - Cross-session/historical export beyond the existing activity export.
    0154 | 
    0155 | ## Edge Cases
    0156 | 
    0157 | - **Very large activity log**: aggregation must not scan it per request (incremental precompute + cache; CN-002/FR-005).
    0158 | - **Restart**: precomputed aggregate rebuilds from the persisted log/rollup on cold start.
    0159 | - **Bytes ≠ tokens**: token graphs labeled as size-based until FR-010 lands; avoid implying exact token counts.
    0160 | - **Sparse/empty data**: clean empty states (FR-009).
    0161 | - **High-cardinality tools**: rank + top-N with an "other" bucket so the histogram stays readable.

## S18

    0001 | ## ADDED Requirements
    0002 | 
    0003 | ### Requirement: Package-contributed EventWriter processors
    0004 | The ingestion pipeline SHALL allow approved package-backed plugins, native add-ons, and
    0005 | sidecars to contribute EventWriter processor definitions through package metadata. Core
    0006 | EventWriter routing SHALL use approved processor contributions and core platform
    0007 | defaults, not producer-specific aliases or subject clauses for integrations such as
    0008 | PowerDNS, Falco, Trivy, Bumblebee, or endpoint inventory.
    0009 | 
    0010 | #### Scenario: Add-on package contributes a processor
    0011 | - **GIVEN** a native add-on package includes a valid EventWriter processor contribution
    0012 | - **AND** the package version is approved
    0013 | - **WHEN** EventWriter builds its routing snapshot
    0014 | - **THEN** the processor contribution SHALL be included in the active routes
    0015 | - **AND** messages matching its approved subject filters SHALL be processed by the
    0016 |   declared platform processor engine
    0017 | 
    0018 | #### Scenario: Unapproved contribution is inert
    0019 | - **GIVEN** a package includes a valid EventWriter processor contribution
    0020 | - **AND** the package version has not been approved
    0021 | - **WHEN** EventWriter builds its routing snapshot
    0022 | - **THEN** the processor contribution SHALL NOT be included in active routes
    0023 | 
    0024 | #### Scenario: Core has no producer-specific alias
    0025 | - **GIVEN** a PowerDNS package contributes an OCSF DNS Activity processor
    0026 | - **WHEN** EventWriter processes `pdns.ocsf` messages
    0027 | - **THEN** routing SHALL resolve through the approved processor registry entry
    0028 | - **AND** core pipeline code SHALL NOT require a `ServiceRadar.EventWriter.Processors.PowerDNS`
    0029 |   alias or producer-specific batcher clause
    0030 | 
    0031 | ### Requirement: Processor contracts are submitted before runtime processing
    0032 | The system SHALL collect EventWriter processor contracts during package import,
    0033 | installation, or add-on registration and persist approved contracts in CNPG. Core SHALL
    0034 | NOT call a running add-on at event-processing time to fetch processor definitions,
    0035 | mapping logic, catalogs, or executable code.
    0036 | 
    0037 | #### Scenario: Add-on is installed on a remote agent
    0038 | - **GIVEN** an add-on package is installed or assigned to an agent
    0039 | - **AND** the package includes a processor contribution contract
    0040 | - **WHEN** the package is approved or registered
    0041 | - **THEN** core SHALL validate and persist the effective processor contract
    0042 | - **AND** EventWriter SHALL use the persisted contract for later matching messages
    0043 | 
    0044 | #### Scenario: Running add-on is unreachable
    0045 | - **GIVEN** a running add-on is offline or only reachable through agent-gateway command bus
    0046 | - **WHEN** EventWriter processes a message emitted by that add-on's approved subject
    0047 | - **THEN** EventWriter SHALL process the message using the persisted contract
    0048 | - **AND** SHALL NOT attempt to call the add-on to retrieve processor details
    0049 | 
    0050 | ### Requirement: Processor contribution registry snapshots
    0051 | EventWriter SHALL consume a versioned registry snapshot containing approved processor
    0052 | contributions, normalized subject filters, processor engine ids, destination metadata,
    0053 | schema/display references, and device-correlation mappings. Snapshot refresh SHALL be
    0054 | atomic; if a new snapshot is invalid, EventWriter SHALL keep the previous valid snapshot.
    0055 | 
    0056 | #### Scenario: Registry refresh succeeds
    0057 | - **GIVEN** a newly approved package processor contribution has no conflicts
    0058 | - **WHEN** the registry snapshot refreshes
    0059 | - **THEN** EventWriter SHALL atomically activate the new snapshot
    0060 | - **AND** subsequent matching messages SHALL use the new route
    0061 | 
    0062 | #### Scenario: Registry refresh fails validation
    0063 | - **GIVEN** a newly staged processor contribution conflicts with an existing route
    0064 | - **WHEN** EventWriter validates a refreshed registry snapshot
    0065 | - **THEN** the refreshed snapshot SHALL be rejected
    0066 | - **AND** EventWriter SHALL continue processing with the previous valid snapshot
    0067 | - **AND** the conflict SHALL be visible to operators
    0068 | 
    0069 | ### Requirement: Processor subject ownership is validated
    0070 | The system SHALL validate requested processor subject filters against package ownership
    0071 | and platform-reserved namespaces before activation. Processor contributions SHALL NOT
    0072 | claim internal health subjects, unrelated producer subjects, or cross-partition routing
    0073 | subjects.
    0074 | 
    0075 | #### Scenario: Package claims reserved subject
    0076 | - **GIVEN** a package processor contribution requests an internal health subject
    0077 | - **WHEN** the package is validated for approval
    0078 | - **THEN** validation SHALL fail with a subject ownership error
    0079 | - **AND** the contribution SHALL NOT become active
    0080 | 
    0081 | ### Requirement: Processor contributions use platform-owned engines
    0082 | Package processor contributions SHALL select platform-owned processor engines and
    0083 | bounded declarative mappings. They SHALL NOT upload arbitrary Elixir modules, SQL,
    0084 | JavaScript, native code, or database DDL for EventWriter execution.
    0085 | 
    0086 | #### Scenario: Package attempts executable processor upload
    0087 | - **GIVEN** a package includes an EventWriter processor contribution that references
    0088 |   arbitrary executable code
    0089 | - **WHEN** the package is validated
    0090 | - **THEN** validation SHALL reject the contribution
    0091 | - **AND** EventWriter SHALL NOT execute package-supplied code
    0092 | 
    0093 | ### Requirement: Broadway-backed package ingestion
    0094 | Package-contributed JetStream routes SHALL be consumed through the Broadway-backed
    0095 | EventWriter producer and pipeline so back-pressure, batching, acknowledgement, retries,
    0096 | and telemetry remain consistent with core EventWriter ingestion.
    0097 | 
    0098 | #### Scenario: Package route is activated
    0099 | - **GIVEN** an approved processor contribution adds a new JetStream subject route
    0100 | - **WHEN** EventWriter refreshes its registry snapshot
    0101 | - **THEN** the route SHALL be consumed through the EventWriter Broadway pipeline
    0102 | - **AND** no separate ad hoc receive loop SHALL be required for that package route
    0103 | 
    0104 | ### Requirement: Package-contributed catalog refresh contracts
    0105 | The system SHALL allow packages to contribute catalog or artifact refresh contracts
    0106 | through package metadata. Core SHALL persist approved catalog contracts and execute them
    0107 | with platform-owned fetch, parser, validator, object-store staging, and promotion
    0108 | engines rather than integration-specific core workers.
    0109 | 
    0110 | #### Scenario: Bumblebee package contributes a catalog contract
    0111 | - **GIVEN** the Bumblebee add-on package includes a catalog refresh contract
    0112 | - **WHEN** the package is approved
    0113 | - **THEN** core SHALL persist the catalog contract as a package-owned contribution
    0114 | - **AND** a generic catalog refresh worker SHALL refresh and promote snapshots
    0115 | - **AND** no Bumblebee-specific catalog refresh worker SHALL be required
    0116 | 
    0117 | #### Scenario: Catalog is assigned to an agent
    0118 | - **GIVEN** a generic catalog refresh contribution has promoted a snapshot
    0119 | - **WHEN** an assigned agent receives config for the contributing add-on
    0120 | - **THEN** the config SHALL include an agent-gateway retrievable artifact reference
    0121 | - **AND** the agent SHALL NOT need direct NATS object-store access

## S19

    0001 | # Specification: Unit Tests for src/domain/template
    0002 | 
    0003 | ## Objective
    0004 | Achieve >=80% test coverage for the `src/domain/template` area by adding unit tests for untested services, authorization services, license services, and resolvers.
    0005 | 
    0006 | ## Scope
    0007 | - **In scope**: All `.service.ts`, `.service.authorization.ts`, `.service.license.ts`, `.resolver.fields.ts`, `.resolver.mutations.ts` files under `src/domain/template/`
    0008 | - **Out of scope**: `.entity.ts`, `.interface.ts`, `.module.ts`, `.dto.ts`, `.input.ts`, `.enum.ts`, `.type.ts`, `.types.ts`, `.constants.ts`, `index.ts`
    0009 | 
    0010 | ## Current State
    0011 | - 8 existing test files covering 119 tests
    0012 | - Coverage: ~53.55% statements
    0013 | - Authorization services, license services, and most resolvers have no tests
    0014 | 
    0015 | ## Files Needing New Tests
    0016 | 1. `template/template.service.authorization.ts` - TemplateAuthorizationService
    0017 | 2. `template-content-space/template.content.space.service.authorization.ts` - TemplateContentSpaceAuthorizationService
    0018 | 3. `template-content-space/template.content.space.service.license.ts` - TemplateContentSpaceLicenseService
    0019 | 4. `template-content-space/template.content.space.resolver.fields.ts` - TemplateContentSpaceResolverFields
    0020 | 5. `template-content-space/template.content.space.resolver.mutations.ts` - TemplateContentSpaceResolverMutations
    0021 | 6. `template-default/template.default.service.authorization.ts` - TemplateDefaultAuthorizationService
    0022 | 7. `template-default/template.default.resolver.fields.ts` - TemplateDefaultResolverFields
    0023 | 8. `template/template.resolver.fields.ts` - TemplateResolverFields
    0024 | 9. `templates-set/templates.set.service.authorization.ts` - TemplatesSetAuthorizationService
    0025 | 10. `templates-set/templates.set.resolver.fields.ts` - TemplatesSetResolverFields
    0026 | 11. `templates-set/templates.set.resolver.mutations.ts` - TemplatesSetResolverMutations
    0027 | 12. `templates-manager/templates.manager.service.authorization.ts` - TemplatesManagerAuthorizationService
    0028 | 13. `templates-manager/templates.manager.resolver.fields.ts` - TemplatesManagerResolverFields
    0029 | 14. `templates-manager/templates.manager.resolver.mutations.ts` - TemplatesManagerResolverMutations
    0030 | 15. `template-applier/template.applier.resolver.mutations.ts` - TemplateApplierResolverMutations
    0031 | 
    0032 | ## Testing Strategy
    0033 | - Use Vitest 4.x globals with NestJS Test module
    0034 | - Mock all dependencies using `defaultMockerFactory` and manual constructor injection
    0035 | - Follow existing test patterns from the codebase
    0036 | - Focus on branch coverage for authorization switch statements

## S20

    0001 | ## MODIFIED Requirements
    0002 | 
    0003 | ### Requirement: Error classification
    0004 | The system SHALL classify errors into codes: `ErrTimeout` (E001), `ErrModelError` (E002), `ErrToolError` (E003), `ErrTurnLimit` (E004), `ErrInternal` (E005), `ErrIdleTimeout` (E006). Classification SHALL be based on error content and context state. Errors containing "thought_signature" or "thoughtSignature" in their message SHALL be classified as `ErrModelError` to prevent learning-based retry attempts.
    0005 | 
    0006 | #### Scenario: Context deadline classified as timeout
    0007 | - **WHEN** the error is or wraps `context.DeadlineExceeded`
    0008 | - **THEN** `classifyError` SHALL return `ErrTimeout`
    0009 | 
    0010 | #### Scenario: Turn limit error classified correctly
    0011 | - **WHEN** the error message contains "maximum turn limit"
    0012 | - **THEN** `classifyError` SHALL return `ErrTurnLimit`
    0013 | 
    0014 | #### Scenario: thought_signature error classified as model error
    0015 | - **WHEN** the error message contains "thought_signature"
    0016 | - **THEN** `classifyError` SHALL return `ErrModelError`
    0017 | 
    0018 | #### Scenario: thoughtSignature camelCase error classified as model error
    0019 | - **WHEN** the error message contains "thoughtSignature"
    0020 | - **THEN** `classifyError` SHALL return `ErrModelError`
    0021 | 
    0022 | #### Scenario: Unknown error classified as internal
    0023 | - **WHEN** the error does not match any known pattern
    0024 | - **THEN** `classifyError` SHALL return `ErrInternal`
