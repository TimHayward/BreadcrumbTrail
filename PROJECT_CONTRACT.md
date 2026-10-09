# Project contract: BrowseTrail

Status: draft for adoption. Reviewed 9 October 2026.

## Purpose

Build an independent Microsoft Edge desktop extension that automatically saves eligible pages visited after activation to an existing Linkwarden instance. No manual save action should be needed during normal browsing.

Working name: **BreadcrumbTrail**. Repository suggestion: `breadcrumbtrail`. This name is provisional; check for conflicting extensions and brands before publication.

Suggested description: “Automatically save visited pages to your Linkwarden instance. An independent, unofficial extension.”

This is an engineering agreement about scope, behaviour and delivery, rather than a legal agreement with Linkwarden or Microsoft.

## Relationship to Linkwarden

Use the Linkwarden HTTP API as an external service. Do not modify the deployed server for the first release. Do not imply endorsement, affiliation or official support.

Use an original extension icon and independent product name. Refer to Linkwarden in compatibility descriptions, setup labels and attribution. Do not use its logo or copy the official extension’s appearance without establishing the relevant rights.

Include this statement in the README, About view and any store listing: “BrowseTrail is an independent project and is not affiliated with or endorsed by Linkwarden.” This disclosure does not itself grant rights to names or artwork.

No dedicated Linkwarden trademark or third-party extension naming policy was found in the reviewed sources. Descriptive compatibility wording is the project’s proposed approach, not a claim of permission from Linkwarden. See BRANDING_AND_RULES.md.

## First release

1. Configure a Linkwarden base URL, access token, profile name and machine name; resolve the required parent and child collections.
2. Validate the connection without creating test links unless the user requests a test save.
3. Explain what will be saved and require an explicit Enable action. Capture starts disabled.
4. Save eligible top-level HTTP(S) navigations across normal tabs, including background tabs and single-page application URL changes. Do not capture subframes, subresources or URLs merely appearing on a page.
5. Send the resulting URL to this installation’s profile/machine child collection. Page titles are optional and must not require content access just for convenience.
6. Provide pause/resume, domain exclusions, queue count, last successful save time, clear local data and actionable error states.
7. Persist pending work across browser and extension worker restarts. Deliver sequentially with bounded retries and visible failures.

“Every link visited” means every eligible navigation observed while enabled. It does not mean an exact browsing-history audit, every redirect hop, pages from before installation, or guaranteed delivery during a browser crash. Linkwarden stores links rather than a visit-by-visit history.

The default dwell threshold is zero. A dwell filter is a later optional feature; it must never silently narrow the default behaviour.

## Multiple profiles and machines — required for MVP

Each Edge profile on each machine has its own extension setup and saves under a shared parent collection, using real Linkwarden nesting:

```text
Browsing History
├── Personal-Edge-Tim's Mac Studio
└── Personal-Edge-HBlue-02832
```

The display path is `Browsing History\<Profile Name>-<Browser>-<Machine Name>`. The backslash denotes a collection boundary, not a literal character in one collection name. Browser is `Edge` in the first release.

Ask for Profile Name and Machine Name during setup. These are user-entered labels, so no operating-system helper or extra permission is needed to discover them. Preserve spaces, apostrophes and hyphens, trim surrounding whitespace, reject empty values and path separators, and validate lengths against the deployed API. Show the complete destination path before activation. Never try to reconstruct the components by splitting the composite name on hyphens.

Resolve or create the private `Browsing History` parent and the correctly named private child during setup, reusing matching collections within the same account and parent. Store their server IDs locally. Verify ownership, write access and effective visibility; do not silently reuse a public/shared collection. If names are ambiguous, ask the user to choose the intended collection. Concurrent setup on multiple machines must re-query and resolve collisions rather than blindly create duplicates; document any API limitation that prevents atomic creation.

Each installation maintains independent local settings, credentials, queue and deduplication state. Do not sync profile/machine identity through Edge sync. Each queued record is bound to the resolved instance/account/child ID when captured. The same URL visited from two different profile/machine combinations must appear in both child collections; finding it elsewhere must not suppress the save.

Renaming either label is a destination change: pause and resolve the newly named child without automatically renaming or moving historical collections. Require a choice to finish existing work at its original destination or discard it; never silently reroute pending records. Missing, moved, renamed or inaccessible collections produce a repair state rather than a fallback to the parent or Unorganized collection. Reinstalling can explicitly reconnect to an existing matching child.

This must be designed during the API spike and implemented in setup before automatic capture. The first release supports many independent installations, with one active destination per installation.

## Eligibility and URL identity

Exclude InPrivate sessions, browser/internal/file/extension URLs, the configured Linkwarden origin and user-blocked domains. Disable InPrivate support in the manifest as well as checking runtime state.

Apply exclusions before persistence or transmission. Domain rules match the hostname and its subdomains at label boundaries; `example.com` must not match `notexample.com`.

Use the browser URL parser for identity. Preserve paths and query parameters; do not automatically remove tracking parameters or other parameters that might change the resource. Strip ordinary fragments by default. Provide an explicit preserve-fragments setting for hash-routed applications. Never transmit embedded URL credentials.

The setup disclosure must explain that URLs can contain search terms and private identifiers. Optional exclusions must be available before activation. No universal sensitive-site detector is promised.

Deduplicate pending and successful URLs per instance/account/collection. A successful-save cache is bounded to 10,000 entries; revisits outside that cache need server reconciliation. Existing server links and ambiguous network results require a verified lookup strategy. Do not claim exactly-once delivery unless the deployed API provides suitable guarantees.

## Data and authentication

Transmit configured profile/machine labels for collection setup, and eligible URLs and optional titles to the user-configured Linkwarden instance. No telemetry, advertising, analytics, developer backend or remote executable code.

Keep the token in extension-local storage, restrict access to trusted extension contexts, never use browser sync storage, and redact it from logs and exports. Local storage is not a secure vault; disclose that limitation. Never pass the token to page scripts or content scripts.

Require HTTPS for non-loopback servers. A localhost HTTP development exception must be explicit and documented. Reject cross-origin redirects on authenticated API requests. Keep the token and queue bound to their configured destination; changing the instance, account or collection pauses capture and requires a decision about existing queued records.

Pause stops collection and queue transmission. Abort in-flight requests where possible; a request already accepted by Linkwarden cannot be recalled. Resume rechecks queued records against current exclusions. Clearing local data deletes the token, settings, pending URLs and deduplication cache; it does not delete links already saved in Linkwarden.

Queue maximum: 1,000 URLs. Pending URLs expire after seven days. On overflow or expiry, show a dropped-record count and explanation. Keep diagnostic records free of full URLs and titles by default.

Linkwarden may independently archive pages or pass data to configured AI providers. Explain that downstream behaviour follows the user’s server settings. This extension neither guarantees an archive nor captures authenticated page contents.

## Implementation boundaries

Use Manifest V3, TypeScript and a background service worker. Start with navigation events, storage and alarms; justify each additional permission. Request optional access only to the configured API origin at setup. Do not request browsing-history access, cookies, content injection or access to page bodies for the first release.

Persist queue transitions before relying on them. Register event listeners at worker startup. Use alarms and resumable state rather than assuming long-lived timers. Serialise queue writes and sends to prevent concurrent navigation events losing data.

Use the documented `POST /api/v1/links` endpoint. Confirm collection listing/creation, parent-child nesting, visibility, permissions, concurrent provisioning, authentication, lookup, pagination, duplicate handling, errors and any rate limits against the actual deployed version before implementing assumptions. Record the tested Linkwarden and Edge versions in the repository.

Start with independently written code under a proposed MIT licence. Before copying any upstream code, record its exact source revision and applicable licence. Do not assume the historical extension’s MIT licence applies to files in the current monorepo.

## Excluded from the first release

Historical browsing import; browser-bookmark sync; screenshots, PDF or HTML uploads; authenticated content capture; server changes; multiple simultaneous destinations within one installation; Firefox, Safari or mobile support; public store publication.

## Completion criteria

The first release is complete when the backlog’s MVP items pass, a reproducible package loads in Edge, a manual acceptance run succeeds against the user’s Linkwarden deployment, and the README explains setup, permissions, limitations, data handling and recovery. No credentials or real browsing records are committed.

Automated checks must cover filtering, URL identity, consent/pause behaviour, queue restart recovery, concurrency, API response validation and failure handling. Manual Edge checks must cover two profiles on each of two machines, collection reuse, cross-collection URL deduplication, destination changes, and normal and single-page navigation, background tabs, InPrivate exclusion, permission withdrawal and browser restart.

## Decisions to resolve during implementation

- Exact deployed Linkwarden version and API behaviour.
- Whether titles can be obtained with justified minimal permissions.
- Server lookup semantics for pre-existing links and uncertain POST outcomes.
- Final product name, repository owner and copyright holder.

Public distribution is a separate milestone. It requires current Microsoft policy review, branding review, a public privacy policy and accurate permission/data disclosures; approval by a store is not assumed.
