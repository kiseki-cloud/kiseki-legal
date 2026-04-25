# kiseki-legal

Static legal pages for Kiseki, deployed at `legal.thekiseki.app/*`.

**Operating entity**: MATRIXVISTA - FZCO (UAE Free Zone Company, License 44526 IFZA, TRN 104467565800001).

**Status**: V1 drafts — pending US SaaS attorney review (trigger: $5–10K MRR). All published pages carry a `DRAFT v1` banner.

## Routes

| URL | File |
|-----|------|
| `legal.thekiseki.app/` | `index.html` |
| `legal.thekiseki.app/terms` | `terms/index.html` |
| `legal.thekiseki.app/privacy` | `privacy/index.html` |
| `legal.thekiseki.app/dpa` | `dpa/index.html` |
| `legal.thekiseki.app/sub-processors` | `sub-processors/index.html` |
| `legal.thekiseki.app/cookies` | `cookies/index.html` |

## Stack

- Pure HTML + one shared `style.css` — no build step.
- Hosted on Cloudflare Pages (free tier).
- DNS: CNAME `legal.thekiseki.app` → `kiseki-legal.pages.dev` (proxied via Cloudflare).

## Local preview

```bash
python3 -m http.server -d /Users/edward/claude-workspace/kiseki-legal 8080
# open http://localhost:8080/
```

(Or use any static server. There is no build step.)

## Updating

1. Edit the relevant `*/index.html` file.
2. Bump the `Last updated` date in the page subtitle and footer.
3. Update `Effective from` if material.
4. Commit and push — Cloudflare Pages auto-deploys.

## Companion docs

- [`COOKIE_CONSENT_SPEC.md`](./COOKIE_CONSENT_SPEC.md) — spec for the consent banner component to be implemented in the future Kiseki frontend (`app.thekiseki.app`). Not implemented in this static site.

## Attorney review

Trigger: first $5–10K MRR. Recommended firms: JPG Legal, Gerben Law, Workman Nydegger.

Priorities for review (in order):

1. AUP for MLM / coaching / health-claim verticals (FTC exposure).
2. Liability cap and indemnification across UAE / US / EU jurisdictions.
3. AI agent behavior disclosure (§7 of Terms).
4. Arbitration clause (DIAC vs DIFC LCIA vs ICC vs AAA).
5. DPA Annex II — confirm TOM claims match implementation.
6. Sub-processors list completeness vs production stack.
