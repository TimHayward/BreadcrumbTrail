# Branding, licensing and platform review

Reviewed 9 October 2026. This records findings and project choices; it is not a trademark clearance or a promise of store approval.

## Findings from Linkwarden

The [official extension repository](https://github.com/linkwarden/browser-extension) says development has moved to the main monorepo. Upstream extension issues and pull requests should have titles beginning `[Browser Extension]`. That convention concerns contributions to Linkwarden, not issues in this independent repository.

The historical extension has an [MIT licence](https://github.com/linkwarden/browser-extension/blob/main/LICENSE), which requires preserving its copyright and permission notices when copying substantial portions. The [main repository licence](https://github.com/linkwarden/linkwarden/blob/main/LICENSE.md) is AGPL-3.0. Inspect the applicable licence of the exact files and revision before reuse; do not assume either licence applies uniformly to all components.

An independently written HTTP API client is the proposed implementation. Treat the server’s licence and the client’s licence separately; copying server code or incorporating upstream components needs another review.

No dedicated naming/trademark policy was located in the official documentation and repositories reviewed. There is no verified requirement here to include “Linkwarden” in the extension name, nor verified blanket permission to use its logo. The main README lists community integrations, but that is not a general branding grant.

## Naming decision

Use **BrowseTrail** as a provisional independent name and an original icon. Explain compatibility in the description: “Automatically save visited pages to your Linkwarden instance.” Avoid “Official”, “Linkwarden Browser Extension”, lookalike icons and a publisher identity that suggests the Linkwarden team.

Use the correct spelling “Linkwarden” in factual documentation. Include an explicit unofficial-project statement. Before release, check the working name for collisions and resolve any uncertain use of protected branding with its rights holder. A disclaimer alone does not authorise logo use.

## Microsoft Edge publication rules

The [Edge Add-ons developer policies](https://learn.microsoft.com/en-us/legal/microsoft-edge/extensions/developer-policies) govern store publication. Relevant requirements include a single clear purpose, distinct truthful identity, essential permissions, disclosed browsing-data use, secure handling, user controls and a maintained privacy policy. The project’s automatic-saving feature must be prominent in onboarding and the listing.

Design for explicit enablement and immediate pause/opt-out. Use HTTPS for production traffic, original branding, and no analytics or advertising. Store submission requires a separate review of the current rules and implementation; local sideloading does not establish eligibility for publication.

## Technical sources

- [Linkwarden browser extension documentation](https://docs.linkwarden.app/getting-started/browser-extension): supported browsers and official extension setup.
- [Create a link API](https://docs.linkwarden.app/api/create-link): authenticated link creation with collection information.
- [Preserve from your browser](https://docs.linkwarden.app/usage/preserve-from-your-browser): browser uploads are a separate capability and are outside the MVP.
- [Chromium navigation API](https://developer.chrome.com/docs/extensions/reference/api/webNavigation): navigation events and single-page application changes.
- [Extension service-worker lifecycle](https://developer.chrome.com/docs/extensions/develop/concepts/service-workers/lifecycle): workers may stop; durable state is required. Verify behaviour in Edge during implementation.
- [Publish an Edge extension](https://learn.microsoft.com/en-us/microsoft-edge/extensions/publish/publish-extension): submission and disclosure workflow.

## Review record to maintain

For every reused component, record source URL, commit/tag, file paths, licence and retained notices. Record supported server/browser versions and the policy review date with each public release. Keep proposed project controls distinct from requirements actually imposed by upstream or the store.
