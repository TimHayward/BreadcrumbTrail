# Initial backlog: BrowseTrail

Each numbered item can become a GitHub issue. P0 is necessary for MVP correctness; P1 completes MVP usability and release readiness; P2 is later work. Dependencies refer to issue numbers below.

## Milestone 0 — establish the integration

### 01 — Verify the deployed Linkwarden API [P0]

Record the deployed version and confirm access-token authentication, collection listing/creation, parent-child nesting, effective visibility and access, link creation, URL lookup and pagination. Verify collection-scoped duplicates and concurrent collection setup from separate installations. Investigate existing-link behaviour, timeout ambiguity and rate limits using a temporary collection.

Acceptance: document actual requests, sanitised responses and error classifications; identify a workable deduplication strategy; remove test records after the spike. No real credentials enter Git. Resolve unsupported API assumptions before queue implementation.

### 02 — Establish repository and independent identity [P0]

Adopt or revise the contract. Add README, proposed MIT LICENSE with the correct copyright holder, original icons, CONTRIBUTING, SECURITY and a provenance record. Confirm the name is provisional and the project is unofficial.

Acceptance: one documented build produces a loadable MV3 package; type checking and focused tests run in CI; dependency versions are locked; no upstream assets or code are copied without a licence record.

## Milestone 1 — one reliable save

### 03 — Connection setup and permissions [P0; depends on 01, 02]

Add base URL, token entry, connection test and Profile Name and Machine Name entry, fixed browser label Edge, and a preview of `Browsing History\<Profile Name>-Edge-<Machine Name>`. Request only the configured API origin at setup; explain permissions. Bind configuration to a destination identity.

Acceptance: valid setup works; invalid tokens, inaccessible servers and denied permissions produce useful errors; token remains in trusted local extension storage; HTTP remote origins and cross-origin authenticated redirects are rejected; changing destination pauses capture and prevents old queued records reaching the new destination.

### 03a — Profile/machine collection provisioning [P0; depends on 01, 03]

Resolve or create a private Browsing History parent and a private child named `<Profile Name>-Edge-<Machine Name>`. Validate labels, preserve spaces/apostrophes/hyphens, store collection IDs locally and avoid syncing installation identity. Handle ambiguous matches, visibility, permissions and concurrent setup.

Acceptance: Personal on Tim's Mac Studio resolves to `Browsing History\Personal-Edge-Tim's Mac Studio`; Personal on HBlue-02832 resolves to `Browsing History\Personal-Edge-HBlue-02832`. Two profiles on each of two machines produce four correctly nested destinations. Repeated setup reuses the selected child. Ambiguous matches or public/shared collections require a user decision; API errors never fall back to the parent or Unorganized. Concurrent setup is tested, with any non-atomic API limitation documented. Label changes pause capture and resolve a new child; old collections remain untouched and old queue entries retain their destination or are explicitly discarded. Deleted/moved/renamed collections enter a repair state.

### 04 — API client and explicit test save [P0; depends on 03a]

Implement response validation, timeouts and link creation. Add a user-triggered test-save action before automatic capture is introduced.

Acceptance: a supplied example URL appears in the resolved profile/machine child collection; non-success or malformed responses never count as saved; authentication errors stop sending; logs contain neither tokens nor complete browsing URLs.

### 05 — Enablement, pause and local reset [P0; depends on 03a]

Explain data flow, URL sensitivity, server archiving/AI behaviour and limitations. Capture starts disabled. Provide enable, pause/resume and reset.

Acceptance: nothing is captured or transmitted before enablement; pause stops new collection and queued sends, with best-effort cancellation of active requests; reset clears all extension-local data; both actions explain that accepted server links remain.

## Milestone 2 — automatic capture

### 06 — Navigation detection [P0; depends on 04, 05]

Observe top-level navigations in normal Edge tabs, including background tabs, redirects to a final destination and SPA history changes. Define how duplicate navigation events are combined. Do not scan historical browser records.

Acceptance: eligible final URLs are captured without a click; subframes and internal pages are ignored; InPrivate is disabled and rejected; a matrix documents redirect, reload, back/forward, restored-tab and SPA behaviour. Fragment settings cover hash-routed applications.

### 07 — Filtering and URL identity [P0; depends on 06]

Implement scheme checks, credential rejection, Linkwarden-origin exclusion, domain exclusions and fragment settings before storage. Preserve meaningful queries.

Acceptance: tests cover hostname boundary matching, credentials, ports, Unicode hosts, queries and hash routes; blocked URLs never enter persistent storage; editing exclusions prevents affected queued records being sent.

### 08 — Durable bounded queue [P0; depends on 01, 04, 07]

Persist queue entries and transitions with their original instance/account/child collection IDs; serialise mutations and sends; resume using alarms after worker/browser restarts. Enforce 1,000 pending records and seven-day expiry.

Acceptance: worker termination during enqueue/send does not silently lose acknowledged work; simultaneous navigation events do not overwrite entries; overflow and expiry show dropped counts; offline work survives restart; pause prevents draining; destination changes cannot redirect queued data.

### 09 — Retry and duplicate reconciliation [P0; depends on 01, 08]

Use bounded exponential backoff with jitter for transient failures and respect applicable retry headers. Distinguish 401/403, invalid input, rate limiting and server/network failures. Reconcile uncertain creation outcomes using the verified API strategy.

Acceptance: a simulated accepted POST followed by a lost response is reconciled before retry where the API allows it; permanent failures do not loop; revoked credentials stop draining; pending duplicates within one destination coalesce; the same URL visited in different profile/machine children is saved in each, even if it already exists elsewhere; a bounded success cache suppresses revisits. Any remaining duplicate risk is documented without an exactly-once claim.

## Milestone 3 — usable MVP

### 10 — Status and recovery controls [P1; depends on 08, 09]

Add toolbar status and a popup showing enabled/paused/error state, current profile/machine destination, queue count, last successful save, dropped count, exclusion controls, retry and discard actions.

Acceptance: users can identify and fix a failed connection without developer tools; keyboard navigation and labels work; no token or browsing URL appears in notifications; reconnecting never silently enables capture.

### 11 — Automated and manual acceptance [P0; depends on 06–10]

Test behavioural boundaries with an API fixture, then run the extension in Edge against the deployed Linkwarden version.

Acceptance: filtering, permission withdrawal, token redaction, concurrency, restart recovery, retries, duplicate reconciliation, pause/reset and destination switching pass. Manual checks with two profiles on each of two machines confirm four destinations, existing collection reuse, identical URLs in different children, rename/repair behaviour and offline queue isolation. Manual normal-tab, background-tab, redirect, SPA and InPrivate checks pass. Record versions, outcomes and known limitations; avoid committing real browsing fixtures.

### 12 — Package and document the first release [P1; depends on 02, 11]

Provide a versioned package, Edge developer-mode installation steps, setup guide including profile/machine labels and nested collection examples, permissions rationale, troubleshooting, privacy/data-retention documentation and licence/provenance notices.

Acceptance: a clean build is reproducible from the tagged source; a fresh Edge profile can install and configure it using only the guide; published documentation accurately describes behaviour. This milestone produces a sideloadable release, not a store submission.

## Later milestones

### 13 — Optional dwell and capture scope settings [P2]

Add an optional dwell threshold and active-tab-only mode. Defaults preserve all eligible visits. Test that navigation and worker suspension do not save the wrong URL.

### 14 — Public Edge Add-ons release [P2; depends on 12]

Recheck current Microsoft policies, finalise name/icon rights, host a privacy policy, complete permission/data disclosures and prepare review instructions with a dedicated test environment.

Acceptance: policy checklist and branding review are recorded; listing is accurate and explicitly unofficial; no production credentials are supplied to reviewers. Submission and approval are separate outcomes.

### 15 — Historical import [P2]

Design a separately enabled, date-bounded history import with preview, deduplication and cancellation. Request history permission only for this feature.

### 16 — Browser-side page preservation [P2]

Investigate screenshot or HTML uploads separately, including extra permissions, data handling and authenticated content. Do not expand MVP permissions in anticipation of this work.

## Suggested first sequence

Start with 01 and 02, then prove 03, 03a, 04 and 05 before building capture and queue machinery. Complete all P0 and P1 items through 12 for the first release. Add `bug`, `integration`, `privacy`, `ux` and `release` labels as needed; priorities and dependencies should remain visible in issue bodies.
