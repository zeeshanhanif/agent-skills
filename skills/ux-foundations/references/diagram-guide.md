# Diagram Guide

Two diagram types carry most of the UX-foundations document: **sitemaps** (the IA
of each surface) and **user flows** (the critical journeys). Embed both as Mermaid
in the document so it stays one portable artifact that renders on GitHub.

## Reliability rules (read first)

The same rules that keep architecture diagrams from breaking apply here:

- **Quote any node text** with spaces or punctuation: `dash["Dashboard"]`,
  `orders["Orders List"]`.
- **Keep edge labels short and quoted:** `login -->|"success"| home`.
- Prefer plain `graph` / `flowchart` over experimental diagram types for
  portability.
- Don't use the literal word `end` as node text or an id (it breaks the parser);
  write `"End"` in quotes or use a different label like `done`.

---

## Sitemap (information architecture)

A top-down tree of a surface's sections and key screens. One sitemap per surface.

```mermaid
graph TD
    root["Admin Portal"]
    root --> dash["Dashboard"]
    root --> orders["Orders"]
    root --> catalog["Catalog"]
    root --> users["Users"]
    root --> settings["Settings"]

    orders --> ol["Orders List"]
    orders --> od["Order Detail"]
    orders --> oref["Refunds"]

    catalog --> cl["Products List"]
    catalog --> ce["Product Editor"]

    users --> ul["Users List"]
    users --> ur["Roles & Permissions"]
```

For a multi-surface product, give each surface its own sitemap rather than one
giant tree — they have different roots and navigation models.

## User flow

The end-to-end path for one important job, across screens, including the
consequential branches (errors, empty, permission). Use a left-to-right
`flowchart` with diamonds for decisions.

```mermaid
flowchart LR
    start(["Admin opens refund"]) --> review["Review order & reason"]
    review --> decision{"Approve?"}
    decision -->|"yes"| process["Issue refund"]
    decision -->|"no"| reject["Record rejection + note"]
    process --> notify["Customer notified"]
    reject --> notify
    notify --> done(["Done"])
```

Keep each flow to a single job. If a flow sprawls, it's usually two flows.

## Cross-surface flow (optional)

When a journey hops between surfaces, show the surface as a lane using subgraphs,
so it's clear where the user is at each step.

```mermaid
flowchart TB
    subgraph web["Website"]
        a["Customer places order"] --> b["Order confirmed on screen"]
    end
    subgraph mobile["Mobile App"]
        c["Push notification: shipped"] --> d["Track delivery"]
    end
    b -->|"later"| c
```

## User-type / navigation map (optional)

When several roles see different navigation, a simple map of role to top-level
sections can clarify access at a glance — a plain `graph LR` from each role node
to the sections it reaches.

---

## Diagram checklist

- One sitemap per surface, rooted at that surface.
- A flow diagram for each critical job (happy path plus the edge cases that
  matter).
- All node text with spaces/punctuation quoted; no bare `end`.
- Labels on decision branches ("yes"/"no", "success"/"error").
- The diagrams say the same thing as the surrounding prose and the screen
  inventory.
