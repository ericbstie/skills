# Matt Pocock's engineering skills — the main flow

Verified against [`mattpocock/skills`](https://github.com/mattpocock/skills) @ `main`.
Source of truth for the flow itself: [`skills/engineering/ask-matt/SKILL.md`](https://github.com/mattpocock/skills/blob/main/skills/engineering/ask-matt/SKILL.md).

## The corrected graph

```mermaid
flowchart TD
    setup["/setup-matt-pocock-skills<br/><i>once per repo — tracker, labels, doc layout</i>"]

    subgraph win1 ["one unbroken context window"]
        grill["/grill-with-docs<br/><i>(no codebase? /grill-me)</i>"]
        spec["/to-spec"]
        tickets["/to-tickets"]
    end

    docs(["CONTEXT.md + ADRs<br/><i>sharpened domain language</i>"])
    proto["/prototype<br/><i>detour, bridged by /handoff</i>"]

    specIssue{{"Spec issue<br/><i>label: ready-for-agent</i><br/><b>declarative — whole feature</b>"}}

    t1{{"Ticket 1"}}
    t2{{"Ticket 2"}}
    t3{{"Ticket 3"}}

    subgraph impl ["/implement — fresh context per ticket"]
        tdd["/tdd<br/><i>red → green → refactor</i>"]
        review["/code-review<br/><i>Standards ∥ Spec</i>"]
        tdd --> review
    end

    commit(["commit"])

    setup -.-> grill
    grill -- produces --> docs
    grill <-.-> proto
    grill --> spec
    spec -- publishes --> specIssue
    spec --> tickets
    tickets -- "publishes N issues,<br/>blockers first" --> t1
    tickets --> t2
    tickets --> t3
    specIssue -. "## Parent" .-> t1
    specIssue -. "## Parent" .-> t2
    specIssue -. "## Parent" .-> t3
    t1 -- blocks --> t2
    t1 -- blocks --> t3
    grill -- "small enough for<br/>one session? skip ahead" --> impl
    t1 -- "frontier:<br/>blockers all closed" --> tdd
    review --> commit

    classDef issue fill:#f3e8ff,stroke:#a855f7
    classDef doc fill:#dcfce7,stroke:#22c55e
    class specIssue,t1,t2,t3 issue
    class docs,commit doc
```

## What the tickets actually look like

Not a tree. A **DAG of blocking edges**, published one issue per ticket in dependency
order, using GitHub's native issue dependencies (`blocked_by`). You work **the
frontier** — any ticket whose blockers are all closed.

Each ticket is a **tracer bullet**: a narrow but complete vertical path through every
layer (schema → API → UI → tests), demoable on its own, sized to fit one fresh context
window.

## Vocabulary check

| Term in the drawing | Verdict | What the repo actually says |
| --- | --- | --- |
| Tracking Issue (from `/to-spec`) | close enough | It's a **spec issue** (a PRD). "Tracking issue" isn't repo vocabulary, but it does become the `## Parent` the tickets point back at. |
| Tracking Issue (from `/to-tickets`) | ✗ | `/to-tickets` creates **no** parent. It publishes N sibling issues that reference the spec issue from the previous step. |
| Sub-Issues | ✗ | Tickets are siblings joined by **blocking edges**. Native sub-issue links are used only "where the platform has one"; the canonical relationship is `blocked_by`. `sub-issue` as a first-class concept belongs to `/wayfinder`, a different flow. |
| Declarative Goals | ✓ | Spec is Problem / Solution / User Stories / Implementation Decisions, "no specific file paths or code snippets". |
| Imperative Instructions | ✗ | Tickets stay declarative: "the end-to-end behaviour this ticket makes work, from the user's perspective — **not** a layer-by-layer implementation list", and the same no-file-paths rule applies. The spec→ticket gradient is **scope**, not declarative→imperative. |

## Context hygiene — the rule with no shape on a graph

- Keep grilling → `/to-spec` → `/to-tickets` in **one unbroken context window**. Don't
  compact or clear until after `/to-tickets`.
- Each `/implement` then starts **fresh**, working from the ticket alone.
- The ceiling is the **smart zone** (~120k tokens). Approaching it before `/to-tickets`
  means `/handoff` to a new thread — not pushing on degraded.
