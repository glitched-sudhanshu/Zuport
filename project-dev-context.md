# Zuport: Android-First DSA Revision System

## Document purpose

This document is the durable product context for Zuport. It is intended to keep future product, design, engineering, and AI-assisted development aligned with the product direction.

Detailed database schemas, API contracts, authentication implementation, authorization, synchronization internals, privacy controls, and security design belong in engineering documentation created during implementation. This document defines the product intent, MVP boundaries, technology direction, and evidence standards those designs must serve.

## Current project state

- The workspace started with the planned directory structure but no meaningful implementation.
- Android code has not been created yet.
- Backend code has not been created yet.
- PostgreSQL schema, migrations, API contracts, and deployment configuration do not exist yet.
- No Figma design system or approved user-flow designs have been created.
- The provisional Android application ID is `com.sudhanshuranjan.zuport`.
- The application ID must be reviewed and finalized before any Google Play upload.
- The project is intended to be a Git repository, but repository setup and conventions still need to be established.

The planned repository structure is:

```text
android/
backend/
docs/
docs/learning/
infra/
```

## Product definition

Zuport is a private-first Android application for developers and students who want to remember and revise coding problems they have already solved.

The product is not another problem library. Users already have LeetCode and similar platforms. Zuport addresses the gap that appears after solving a problem: users forget their reasoning, the pattern involved, why an approach works, what made an earlier approach inefficient, and which mistakes they made.

The core product promise is:

> Remember how you solved it.

Zuport helps a user reconstruct their own reasoning before revealing their saved explanation and code.

The initial target user is a developer or student preparing for coding interviews who:

- Has solved approximately 20–50 coding problems.
- Uses LeetCode primarily on desktop.
- Struggles to revise old solutions consistently.
- Wants access to saved material while offline or travelling.
- Currently relies on GitHub, Notion, notes, spreadsheets, or memory.
- May have multiple accepted solutions for the same problem.

LeetCode is the first source platform, but the internal data model must remain source-agnostic so that other platforms such as Codeforces or HackerRank can be considered later.

## Initial release scope

The first useful release focuses on a private revision workflow and supports:

- Email/password registration and login.
- Manual import of an accepted solution.
- Problem metadata and a link to the original source.
- Multiple solution versions for the same problem.
- Personal text explanations attached to solutions.
- User-authored tags and topic categorization.
- GitHub repository connection through a GitHub App.
- Repository and branch selection.
- Initial read-only repository scanning and import.
- Signed, idempotent GitHub webhook synchronization after repository changes.
- Offline access to imported problems, code, notes, and available audio.
- Local-first writes with retryable synchronization.
- Audio voice-note recording, playback, deletion, and retry after failed upload.
- Review queue and deterministic review scheduling.
- Review sessions with Again, Hard, Good, and Easy confidence ratings.
- On-demand AI time and space complexity analysis.
- Personal-data export and account deletion.

The first ingestion target is accepted solutions. Failed submissions and detailed attempt history are deferred.

## Solution ingestion and platform boundaries

Zuport must not depend on direct LeetCode scraping, browser cookies, or LeetCode credentials.

Manual import allows the user to provide:

- Problem URL.
- Problem title or slug.
- Programming language.
- Solution code.
- Accepted status.
- Optional submission date.
- Optional personal explanation.

GitHub import uses a GitHub App and should:

- Request only the minimum repository permissions.
- Use read-only repository contents access.
- Let the user select a repository and branch.
- Perform an initial repository scan.
- Subscribe to push webhooks.
- Validate webhook signatures.
- Process deliveries idempotently.
- Import new or changed solution files.
- Deduplicate identical solutions using a stable content hash.
- Never modify the user’s repository.

The importer should support a documented repository convention and a configurable mapping for common layouts. A third-party browser extension may write LeetCode code to GitHub, but Zuport must not depend on that extension or claim official LeetCode synchronization.

A solution record should be able to represent:

- Source platform.
- Problem slug, title, difficulty, and source URL.
- Programming language.
- Code and accepted status.
- Commit SHA or external submission identifier.
- Submission timestamp.
- Stable content hash.
- Import source and import timestamp.

Zuport may store user-authored code, explanations, tags, voice notes, problem metadata, source URLs, and synchronization metadata. It must not scrape, bulk-copy, or redistribute full LeetCode problem statements, editorials, other users’ solutions, or private/restricted content. Full problem details should be opened from the original source link.

## Explicitly out of scope for the first release

- Public questions or answers.
- Voting, reputation, comments, follows, public profiles, or community feeds.
- Community moderation or accepted community answers.
- Direct LeetCode scraping or official LeetCode synchronization claims.
- LeetCode credentials, passwords, or browser session-cookie storage.
- Failed-submission history and detailed attempt timelines.
- Speech-to-text transcription.
- Automatically generated “better solutions”.
- Automatic AI analysis for every imported solution.
- FSRS or another advanced memory algorithm before the basic review loop is validated.
- Microservices.
- Google sign-in before the email/password flow is stable.

Public community features may be reconsidered only after the private revision workflow demonstrates retention.

## Product strategy and risks

Zuport needs to provide a revision workflow that is meaningfully better than leaving code in GitHub or notes in Notion. A larger feature list is not evidence of that advantage.

The main risks are:

- Users may not maintain explanations or voice notes after the initial import.
- Users may prefer existing tools if the recall-before-reveal workflow is not materially useful.
- GitHub repository layouts may be inconsistent and difficult to import reliably.
- Webhook delivery, duplicate events, branch changes, renamed files, and deleted files may create synchronization errors.
- Offline edits and server changes may produce conflicts that users do not understand.
- Audio permissions, large files, upload failures, and object-storage access may make voice notes unreliable.
- AI complexity analysis may be confidently wrong, especially for unusual code or language-specific behavior.
- Storing private code and voice notes creates authorization, deletion, export, and privacy obligations.
- Building GitHub integration and AI before validating the basic review loop would create expensive scope without proving value.

The primary product risk is retention of the revision habit, not the ability to import data. Validation must test whether users return and remember more effectively.

## Product validation

Zuport is both:

1. A learning and portfolio project for building a production-shaped Android and Go product.
2. A potentially usable private revision tool for developers and interview candidates.

Working software must not be confused with product validation. Downloads, signups, and compliments are weak evidence.

Validation should measure:

- Whether users import real solutions.
- Whether users add a personal explanation or voice note.
- Whether users complete a review session without immediately revealing the code.
- Whether users return for reviews during three consecutive weeks.
- Whether users can explain why Zuport is better than GitHub or Notion for revision.
- Whether offline access works when it matters.
- Whether GitHub synchronization failures are visible and recoverable.

Initial evidence targets are:

- At least five users import real solutions.
- At least five users complete one review session.
- At least three users return for reviews during three consecutive weeks.
- At least half of imported solutions receive a personal explanation or voice note.

These are evidence targets, not a reason to claim product-market fit or to block the learning project indefinitely.

## Revision workflow

The primary user workflow is:

1. Open the review queue.
2. See the problem title, metadata, and personal notes.
3. Recall the approach before viewing the saved code.
4. Optionally record or play a voice explanation.
5. Reveal the saved explanation and solution.
6. Rate confidence as Again, Hard, Good, or Easy.
7. Schedule the next review.

MVP scheduling uses deterministic intervals:

- Again: 1 day.
- Hard: 2 days.
- Good: 7 days.
- Easy: 21 days.
- Repeated successful Easy reviews may advance to 45 days.

The basic review loop must be validated before introducing a more complex memory algorithm.

## Technology direction

### Android

- Kotlin.
- Jetpack Compose.
- ViewModel-based state management.
- Unidirectional data flow where practical.
- Navigation Compose.
- Room for local persistence.
- Kotlin coroutines and Flow.
- WorkManager for retryable background synchronization.
- Android Keystore where secure local secrets are required.
- Repository boundaries separating UI, local data, and remote data.
- Local-first writes and offline browsing.
- Explicit offline, pending, failed, conflict, and synchronized states.

### Backend

- Go.
- Modular monolith initially.
- chi for HTTP routing.
- PostgreSQL.
- pgx for database connectivity.
- sqlc for type-safe generated SQL.
- goose or golang-migrate for migrations.
- Versioned REST/JSON APIs.
- Docker Compose for local PostgreSQL and supporting services.
- S3-compatible object storage for audio files.
- Structured logging and health checks.
- Environment-based configuration.
- Automated tests in CI.

The backend should have clear boundaries for authentication, users, problems, solution versions, explanations, tags, reviews, voice notes, GitHub connections, repositories, webhook deliveries, AI analyses, and synchronization state. These boundaries do not justify microservices at this stage.

## Authentication, privacy, and security direction

Initial authentication uses email and password with:

- Modern password hashing.
- Short-lived access tokens.
- Rotating, revocable refresh tokens.
- Planned email verification and password reset before production launch.

Security requirements include:

- Never storing plaintext passwords.
- Never embedding secrets or AI API keys in the Android application.
- Enforcing ownership and authorization on every authenticated request.
- Limiting users to their own solutions, notes, audio, analyses, and GitHub connections.
- Validating GitHub webhook signatures.
- Making webhook processing idempotent using delivery IDs and commit SHAs.
- Rate-limiting authentication and AI endpoints.
- Avoiding secrets and unnecessary full private code in logs.
- Supporting personal-data export and account deletion.

Audio belongs in object storage rather than PostgreSQL. Access should use authorization-protected signed URLs. Deletion must cover database records, object storage, synchronization metadata, and relevant derived data.

## Product design and Figma

Design will be created feature by feature using one consistent design system.

Initial design areas:

1. Welcome, registration, and login.
2. Home and review dashboard.
3. Review queue.
4. Review session and confidence rating.
5. Problem list, filtering, and tags.
6. Problem detail and source link.
7. Solution-version list and solution detail.
8. Manual import.
9. GitHub connection, repository selection, branch selection, and initial import.
10. Synchronization status and conflict recovery.
11. Voice-note recording and playback.
12. Complexity analysis.
13. Settings, export, and account deletion.

The design system should define typography, spacing, colors, status indicators, forms, cards, code display, audio controls, loading states, empty states, error states, offline states, pending states, and success states.

Critical states must be designed rather than treated as afterthoughts:

- No imported solutions.
- Empty review queue.
- Loading.
- Offline browsing.
- Pending local change.
- Failed synchronization.
- Recoverable conflict.
- Duplicate import.
- GitHub authorization failure.
- Webhook processing failure.
- Failed audio upload with retry.
- Missing audio because it is still syncing.
- AI analysis pending, failed, and uncertain.
- Account export in progress.
- Account deletion confirmation and failure.

If direct Figma integration is unavailable, produce a structured design specification and do not claim that a Figma file was created.

## Development sequence

### Phase 0: Product validation and scope confirmation

- Confirm the target user and the recall-before-reveal workflow.
- Test whether users consider Zuport better than GitHub or Notion for revision.
- Define the smallest useful review loop.
- Confirm terminology and metadata users actually need.
- Validate the need for offline access and voice notes before overbuilding them.
- Define repository conventions and maintain this project context document.

### Phase 1: User flows, information architecture, and design

- Define information architecture and user journeys.
- Define the MVP screen inventory.
- Establish the visual design system.
- Create feature-by-feature Figma designs or a structured design specification.
- Design loading, empty, offline, pending, error, conflict, and success states.
- Review the flows and handoff notes before production engineering.

### Phase 2: Android and Go repository foundation

- Create the Android and Go projects.
- Establish package and module conventions.
- Establish local development instructions.
- Add baseline build and test checks.
- Add environment and secret-handling conventions.
- Add CI checks.

### Phase 3: PostgreSQL, migrations, configuration, logging, and health checks

- Set up local PostgreSQL and Docker Compose.
- Add migrations and database configuration.
- Add structured logging.
- Add health and readiness checks.
- Document the database and operational assumptions.

Do not begin Phase 3 production engineering until the Phase 1 flows and design system have been reviewed.

### Phase 4: Authentication and user ownership

- Implement email/password registration and login.
- Add access and refresh-token handling.
- Add user ownership and authorization checks.
- Add the foundations for email verification, password reset, export, and deletion.
- Test authentication failures, token rotation, revocation, and cross-user access denial.

### Phase 5: Local Android solution library and manual import

- Implement Room entities and repositories for problems, solution versions, explanations, tags, reviews, and sync state.
- Implement local-first manual import.
- Implement problem and solution browsing offline.
- Expose pending, failed, and synchronized states.
- Add retryable synchronization interfaces without hiding conflicts.

### Phase 6: Backend synchronization

- Add versioned APIs for problems, solutions, explanations, tags, and review data.
- Implement idempotent synchronization.
- Handle retries, deduplication, conflict detection, and ownership enforcement.
- Test offline retry and recovery behavior.

### Phase 7: GitHub App integration and webhook import

- Implement GitHub App authorization with minimum permissions.
- Add repository and branch selection.
- Implement initial repository scanning and configurable file mapping.
- Validate webhook signatures.
- Process delivery IDs and commit SHAs idempotently.
- Import new or changed solution files without modifying the repository.
- Surface failures and conflicts to the user.

### Phase 8: Review scheduling and review sessions

- Implement the review queue.
- Implement the recall-before-reveal session flow.
- Implement confidence ratings and deterministic scheduling.
- Add review history and next-review visibility.
- Test all interval rules and repeated Easy advancement.

### Phase 9: Voice notes and object storage

- Implement recording, playback, pause/resume where practical, and deletion.
- Add object-storage upload and signed access.
- Support offline recording and later synchronization.
- Add retry and failure states.
- Test permission failures, interrupted uploads, duplicate uploads, unauthorized access, and deletion.

### Phase 10: On-demand complexity analysis

- Add a backend abstraction for AI providers.
- Protect AI access with a feature flag.
- Analyze only when explicitly requested by the user.
- Store complexity estimates, reasoning, assumptions, uncertainty, provider/model metadata, and timestamps.
- Clearly communicate that the result may be incorrect.
- Test rate limits, provider failures, timeouts, redaction decisions, and authorization.

### Phase 11: Security hardening, testing, observability, and pilot release

- Review authorization and ownership boundaries.
- Test authentication, imports, webhooks, review scheduling, offline retry, audio failures, export, and deletion.
- Add operational metrics and actionable logs without exposing private data.
- Review backups, restoration, migrations, and failure recovery.
- Run a small pilot with real DSA learners.
- Separate implemented, tested, manually verified, planned, blocked, and unknown work in all progress reports.

## Backend learning documentation

Because the project is also a backend learning project, every backend feature must include supporting material covering:

- What problem the feature solves.
- How the request flows through the system.
- Why the selected design was chosen.
- At least one alternative approach and its trade-offs.
- Database involvement.
- Security and failure considerations.
- How the feature is tested.
- Official documentation references first.
- One or two reputable supplementary lectures or articles where useful.

Durable learning notes belong in `docs/learning/`. References should explain the reasoning behind the implementation rather than exist merely to satisfy a link requirement.

## Completion criteria

The MVP is complete only when:

- A user can create an account and authenticate securely.
- A user can manually import an accepted solution.
- A user can connect a GitHub repository and select a branch.
- A GitHub push can trigger an idempotent import.
- Identical solutions are deduplicated.
- Multiple versions of the same problem are visible.
- A user can add text and audio explanations.
- Imported content is available offline.
- Local changes retry after connectivity returns.
- Conflicts and failed synchronization are visible and recoverable.
- A user can complete a review and schedule the next review.
- A user can request on-demand complexity analysis with uncertainty clearly shown.
- A user can export and delete personal data.
- Automated tests cover authentication, authorization, imports, webhook handling, review scheduling, offline retry, audio upload failures, export, and deletion.
- The application is demonstrably runnable in a documented environment.

Do not claim a feature is complete until it is implemented, tested, and verified.

## Resume policy

Resume descriptions will be generated only from features that are actually implemented and tested. Features existing only in plans, design files, or partial prototypes must not be presented as completed engineering work.

The project must not claim:

- Production-scale usage.
- Product-market fit or user adoption without evidence.
- Official LeetCode integration.
- AI accuracy.
- Production-grade security or formal security certification.
- Reliability percentages without measurements.
- Features that exist only in Figma or planning documents.

## Working assumptions

- The private revision workflow is the product; importing code is only an enabling capability.
- The first target user has already solved a meaningful number of coding problems.
- LeetCode is the first source platform, but the data model remains source-agnostic.
- GitHub integration is read-only for Zuport and uses signed, idempotently processed webhooks.
- Full problem statements and editorials are not copied.
- Voice notes are audio-only in the MVP.
- AI complexity analysis is explicit, on-demand, feature-flagged, and potentially incorrect.
- Deterministic review intervals come before advanced spaced-repetition algorithms.
- Design and flow specifications precede production engineering.
- The project should be implemented in small, tested, educational phases.
- The next product artifact is the finalized Phase 0 validation and Phase 1 user-flow specification.
- Figma designs or a structured design specification follow the user-flow specification.
- Engineering implementation follows reviewed flows and designs.
