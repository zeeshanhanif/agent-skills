# Manifest Guide (docs/design-manifest.json)

The global, SCR-keyed screen registry — the adapter's output contract. With
per-screen routing, **no design tool ever holds the complete picture; only the
manifest does.** Downstream skills read it and never touch a tool. This skill
is its **single writer**.

## Why it exists (so it's never treated as ceremony)

Three facts demand it: (1) heterogeneous sources — one feature's screens can
be split across registered/generated/code-native, and without one resolution
layer every downstream skill would need tool-specific logic; (2) screens
outlive and cross features — anchors precede any feature folder, and a screen
designed in FEAT-004 is reused by FEAT-009; the pick-if-exists lookup needs a
global index; (3) it's the per-screen lifecycle ledger (states coverage,
conformance, escalations) that nothing else tracks — the screen-level analog
of the RTM.

## Schema

```json
{
  "version": 1,
  "policy": {
    "fallback": "code-native",
    "note": "asked once on first run; overridable per screen"
  },
  "screens": {
    "SCR-WEB-004": {
      "name": "Sign-in",
      "surface": "WEB",
      "feature": "FEAT-004",
      "strategy": "registered",
      "source": {
        "tool": "figma",
        "locator": "https://figma.com/file/…?node-id=…"
      },
      "states": {
        "covered": ["default", "error", "loading"],
        "gaps": [],
        "supplements": [
          { "state": "empty", "strategy": "code-native",
            "source": "features/FEAT-004-sign-in/ui-design.md#empty" }
        ]
      },
      "contract_bindings": ["POST /auth/login"],
      "conformance": {
        "result": "pass",
        "corrections": ["raw #1a5f4a → --color-primary"],
        "escalations": []
      },
      "status": "designed",
      "designed_at": "2026-07-16"
    }
  }
}
```

Field notes:
- **Key = SCR ID**, unique — one entry per screen, ever. A screen reused by a
  later feature updates its entry (bindings may grow); it never duplicates.
- **`feature`**: the FEAT ID that designed it, or `"anchor"`. Reuse by later
  features doesn't rewrite this — it records provenance, not ownership.
- **`strategy`**: `registered` | `generated` | `code-native`.
- **`source`**: tool + locator for tool-backed screens; spec path+section
  for code-native. This is what implementation resolves — it must always
  point at something real.
- **`states`**: covered honestly; gaps recorded, not glossed; supplements are
  code-native fills of states a tool design lacked.
- **`contract_bindings`**: the endpoints (from technical-design.md) the
  screen consumes. Empty for anchors.
- **`conformance`**: pass / corrected / escalation-pending, with the specifics.
- **`status`**: `designed` | `escalation-pending` | `rework-needed` (set by
  re-verification when a design.md amendment invalidates the screen).

## Write rules

- **Single writer**: only this skill writes the file. Downstream reads.
- **Append or update-own-entry**: new screens append; reuse/re-verification
  updates the existing entry in place. Entries are never deleted — a screen
  removed from the inventory (tombstoned SCR) gets `status: removed`,
  mirroring the pipeline's tombstone discipline.
- **The policy header is written once** (first run that needs a fallback) and
  changed only by explicit user instruction.
- **Locators must resolve at write time** — a manifest entry pointing at
  nothing is a verification failure, not a TODO.
- If the file doesn't exist, create it on first write. JSON stays
  pretty-printed and diff-friendly (stable key order: sorted SCR IDs).
