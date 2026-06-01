# subwaybuilder-tw-maps — Agent orientation

This repo holds **release-artifact map packs for Taiwanese metropolitan
areas** built by the bundle pipeline in
[`ahkimn/subwaybuilder-jp-data`](https://github.com/ahkimn/subwaybuilder-jp-data).
This repo does NOT contain the pipeline itself; it ONLY hosts
finished release ZIPs + listing pages for end users.

## What's in this repo

- `releases/` — per-bundle release artifacts (one ZIP per release tag)
- `README.md` — high-level feature summary for the public
- `LICENSE` — repo license

There is no source code or build step in this repo. New releases are
produced upstream and pushed here.

## What you'll usually be asked to do

- Add or update a release ZIP under `releases/<bundle>/`
- Update the public-facing listing template (the `TW_LISTING_TEMPLATE.md`
  pattern lives upstream; the rendered listing for each bundle is what
  appears here)
- Visual QA on a new release before publishing

## Where the algorithmic context lives

This repo's outputs change when the upstream pipeline ships an arc.
The **canonical cross-repo log** of those arcs is:

**[ahkimn/subwaybuilder-jp-data — ALGORITHMIC_DECISIONS_LOG.md](https://github.com/ahkimn/subwaybuilder-jp-data/blob/main/ALGORITHMIC_DECISIONS_LOG.md)**

Read it **before** the QA pass on any release whose data was
re-generated upstream. Entries cover:

- What changed in the pipeline
- Which release bundles are affected
- Expected visual impact (where mass should shift / what new patterns
  should appear)
- Known regressions (specific bundles / specific 里 (village/borough)
  where the arc has documented hotspots)
- Reviewer-side action — what to specifically check during visual QA

For depth on a specific arc, follow the readout links in the log. The
TW cordon (2026-05-30, outbound on / inbound built-but-off) is the
most recent TW-facing entry.

## Sister repos

- [`subwaybuilder-jp-data`](https://github.com/ahkimn/subwaybuilder-jp-data) — the bundle pipeline (where releases come from)
- [`subwaybuilder-jp-maps`](https://github.com/ahkimn/subwaybuilder-jp-maps) — Japanese release artifacts
- [`subwaybuilder-eu-maps`](https://github.com/ahkimn/subwaybuilder-eu-maps) — European release artifacts (CZ + PL + ...)

The three maps repos share the algorithmic-decisions log via URL link.
Don't fork the log; always read the upstream URL.
