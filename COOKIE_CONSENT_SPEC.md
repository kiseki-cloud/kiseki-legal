# Cookie Consent Banner — Component Spec

**Status**: Spec for future Kiseki frontend. The legal page at `legal.thekiseki.app/cookies` is published and explains the categories; this document specifies the **interactive consent component** to ship in `app.thekiseki.app`, `thekiseki.app` (marketing), and any future Kiseki-owned web surface.

**Last updated**: 2026-04-26

---

## 1. Why this exists

Publishing `/cookies` is necessary but not sufficient for GDPR / ePrivacy / UAE PDPL compliance. The user must be able to **give granular, informed, opt-in consent before non-essential cookies are set**, and revisit that choice at any time.

This spec is what the Kiseki frontend dev (or AI agent building the dashboard) implements when a Kiseki-owned site goes live. It is not implemented in the static `legal.thekiseki.app` site — that site sets only strictly-necessary cookies and does not need the banner.

## 2. Functional requirements

### 2.1. First-visit behavior

- On first load, **set only Strictly Necessary cookies** (auth session, CSRF). Block all other categories until consent is recorded.
- Show the consent banner above the fold, non-blocking (slides up from the bottom on desktop and mobile).
- The banner MUST present three actions of equal visual weight: **Accept all**, **Reject all** (non-essential), **Manage preferences**.
  - "Reject all" must be no harder to find or click than "Accept all" — UK ICO and CNIL both fine designs that bias toward acceptance.

### 2.2. Granular preferences

When the user clicks "Manage preferences", show a panel with toggles for each category:

| Category | Default | Toggle locked? |
|----------|---------|----------------|
| Strictly necessary | On | Yes (cannot be disabled) |
| Performance & analytics | Off | No |
| Functional | Off | No |
| Marketing | Off | No |

The panel must list the cookies in each category (name, vendor, purpose, retention). Pull this list from a single shared config (see §4) so the banner stays in sync with reality.

### 2.3. Persistence

- Store the user's choices in a first-party cookie `kiseki_consent` (JSON, `Max-Age=31536000` = 1 year).
- Stamp the cookie with the consent version (e.g. `v1`) and an ISO 8601 timestamp.
- When the consent version changes (new sub-processor, new category, material policy update), invalidate stored consent and re-prompt.

### 2.4. Revisit / withdraw

- A persistent footer link **"Privacy choices"** opens the same Manage Preferences panel.
- Withdrawing consent must take effect immediately: the corresponding cookies are deleted client-side and any third-party scripts are unloaded (or, for SDKs that can't be unloaded, fall back to a full-page reload after the change is saved).

### 2.5. Region detection

- Default to **opt-in (EU / UK / Switzerland / Brazil / California)** behavior globally — it's the safer default and the simpler UX.
- Optionally short-circuit to a softer banner ("This site uses cookies. [OK]") in jurisdictions where opt-out is sufficient. Only do this if a clear product reason emerges; until then keep one banner.

### 2.6. Do Not Track

- Respect the browser-level `Sec-GPC` and `DNT` headers when set: do not set any opt-in category, even if a banner choice has not been recorded yet.

### 2.7. Accessibility

- Banner is keyboard-navigable (Tab order: Reject → Manage → Accept).
- All actions reachable without a mouse; visible focus rings (≥3:1 contrast).
- ARIA roles: `role="dialog"`, `aria-modal="false"` (non-blocking), `aria-labelledby` pointing at the banner heading.
- Color contrast ≥ 4.5:1 for text, ≥ 3:1 for interactive elements.
- Banner closes via `Esc` only after a choice is made (closing without choosing should not be possible — that would be an implicit "reject all").

### 2.8. Performance

- The banner is loaded inline (no network request to render).
- Third-party scripts (PostHog, Meta Pixel, Google Tag, etc.) MUST NOT load until consent for the matching category is recorded.
- Use a tag-manager pattern with conditional loaders, not a "load + unload" pattern. Once a tag has been loaded once, the third party already has the data point.

## 3. Implementation options

| Option | Cost | Pros | Cons |
|--------|------|------|------|
| **Klaro** (open source) | $0 | Self-hosted, no vendor data, granular config | Manual config maintenance, plain styling |
| **PostHog Privacy banner** | $0 (already in stack) | Integrated with our analytics, no extra vendor | Tied to PostHog as the consent system of record |
| **Termly** | $30 / month | Auto-updates with policy changes | Recurring cost; another vendor; doesn't beat the V1 we already wrote |
| **Cookiebot** | €15+/month | Mature, full-featured | Vendor lock-in; some teams have flagged tracker-blocking edge cases |

**Recommendation**: ship with Klaro (or a hand-rolled component if it fits the design system) and use PostHog's `consentcategory` filter to gate analytics events. Don't pay $30 / month for a generic banner when the V1 legal pages are already self-hosted.

## 4. Shared config (single source of truth)

Cookie categories, vendors, and retention live in **one** TypeScript file consumed by both:

- The legal `/cookies` page (rendered at build time);
- The consent banner component (rendered at runtime).

Shape:

```ts
export interface CookieCategory {
  id: 'necessary' | 'analytics' | 'functional' | 'marketing';
  label: string;
  description: string;
  required: boolean;
  cookies: Array<{
    name: string;
    vendor: string;
    purpose: string;
    retention: string;
    domain?: string;
  }>;
}

export const COOKIE_CATEGORIES: CookieCategory[] = [ /* ... */ ];
export const CONSENT_VERSION = 'v1';
```

When a new sub-processor is added, update this file and bump `CONSENT_VERSION`. The banner re-prompts; the legal page rebuilds; the sub-processors page is updated separately and customers receive 30 days' notice per the DPA.

## 5. Out of scope for this spec

- Server-side enforcement (the backend trusts that the frontend respects consent — multi-tenant isolation and AuthZ are separate concerns).
- Email marketing consent (handled by Resend's double opt-in flow, not cookies).
- Analytics for end-users of the Customer's funnels (the Customer is the controller for those events; their own consent UX governs).

## 6. Acceptance criteria (for the dev / agent who implements this)

- [ ] Banner appears on first visit, blocks all opt-in categories until choice is made.
- [ ] "Reject all" and "Accept all" are visually equivalent — same size, same color contrast, same affordance.
- [ ] Manage Preferences shows per-category toggles with cookie-level detail.
- [ ] Choice persists in `kiseki_consent` cookie for 1 year, with version stamp.
- [ ] Footer link "Privacy choices" reopens Manage Preferences.
- [ ] Withdrawing consent removes cookies + reloads page.
- [ ] PostHog / Meta Pixel / Google Tag scripts are not loaded until the matching category is opted in.
- [ ] Keyboard navigation works (Tab, Esc after choice).
- [ ] WCAG AA contrast on all banner text.
- [ ] Sec-GPC / DNT respected.
- [ ] Lighthouse audit passes Privacy + Best Practices ≥ 95.
- [ ] Banner config is consumed from a shared file (not duplicated between legal page and banner code).

## 7. References

- ICO (UK): [PECR cookies guidance](https://ico.org.uk/for-organisations/direct-marketing-and-privacy-and-electronic-communications/guide-to-pecr/cookies-and-similar-technologies/)
- CNIL (FR): [Cookies and other trackers](https://www.cnil.fr/en/cookies-and-other-trackers)
- EDPB: Guidelines 03/2022 on dark patterns
- UAE PDPL: Federal Decree-Law No. 45 of 2021
