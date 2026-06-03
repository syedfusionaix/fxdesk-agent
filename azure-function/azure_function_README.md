# Azure Function — Dynamic Adaptive Card Engine

This Azure Function is the component that makes fxDesk's Adaptive Cards **dynamic and
metadata-driven**. Instead of hardcoding card layouts, the function builds each Adaptive
Card at runtime from the backend system's own UI metadata — so the agent's forms stay in
sync with ServiceNow and Pega automatically.

> This document describes the function's design and behaviour. The implementation is not
> included in this public repository.

---

## What it does

The function sits between the Copilot Studio agent (the Incident Agent) and the backend
systems of record. When the agent needs to render a form or a result, it calls this
function, which returns a ready-to-render Adaptive Card schema generated from live backend
metadata.

It supports two backend systems, each with its own retrieval path, followed by a common
transformation step.

### Pega

1. The function calls **Pega's DX API**, which creates a case and returns the case's **UI
   metadata** (the fields and layout Pega itself defines for that case).
2. The function **transforms that UI metadata into Adaptive Card schema** and returns it to
   the agent for rendering.

Because the card is built from Pega's own returned metadata, any change to the Pega case
UI (for example, a new field added to the process) is reflected in the agent's card the
next time it is rendered — with no change to the function and no change to the agent.

### ServiceNow

1. The function calls the **ServiceNow Table API** to retrieve the incident's fields.
2. It selects the **required fields** (configurable) from that response.
3. It **transforms the selected fields into Adaptive Card schema** and returns it to the
   agent.

The set of fields surfaced is configurable, so the card can be tailored without code
changes to the transformation itself.

---

## Why this matters

- **Single source of truth.** The backend system (Pega or ServiceNow) owns the UI
  definition. The agent never holds a hardcoded copy of the form.
- **Self-syncing UI.** UI changes made in the backend appear in the agent automatically —
  no manual card rebuilding, no redeployment of the agent.
- **One transformation contract.** Both paths converge on the same step: backend metadata →
  Adaptive Card schema. New backends can be added by implementing a retrieval path that
  feeds the same transformation.

---

## Where it fits in the architecture

```
Copilot Studio (Incident Agent)
        │  request to render a card
        ▼
   Azure Function  ──►  Pega DX API        (creates case, returns UI metadata)
        │            └► ServiceNow Table API (returns incident fields)
        │
        ▼
  Transform metadata → Adaptive Card schema
        │
        ▼
  Adaptive Card returned to the agent and rendered in Microsoft Teams
```

See the top-level `fxDesk_Architecture.png` for the full solution architecture.
