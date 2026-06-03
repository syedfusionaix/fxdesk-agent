# fxDesk — Integrated, Enterprise-Wide Service Management Agent

An AI-powered, multi-agent assistant built in **Microsoft Copilot Studio** that gives an
entire enterprise a single conversational front door for service management. Employees
describe what they need in plain language; fxDesk routes each request to the right
specialist agent and handles it across multiple backend systems — **ServiceNow, Pega, and
Microsoft Dataverse**.

Built for the **Microsoft Agent Academy Hackathon (Operative track)**.

---

## The Problem

Across an enterprise, service management is fragmented. IT, HR, and facilities each use
different systems and portals (ServiceNow, Pega, and more), and employees must know which
tool and form to use for every issue or request. Incidents live in one system, service
requests in another, with no single view and slow, manual triage.

**Target users:** employees across the enterprise who need to raise an issue, request
something, or check the status of an existing ticket — without needing to know which system
or form to use.

---

## What It Does — Three Capabilities

### 1. Troubleshoot & raise an incident (multi-system)
The employee describes a problem. fxDesk troubleshoots from a knowledge base first, and
raises an incident when escalation is needed — routing it to the right backend:
- A WiFi connectivity issue → raised in **ServiceNow**
- A payslip-download issue → routed to **Pega**

One conversation, the correct system of record behind each issue.

### 2. Create a service request
The employee asks for software, access, or hardware. The Service Request Agent logs a
structured service request on their behalf using the **out-of-the-box Dataverse MCP server**
(added as a tool on the connected agent), which creates the request as a record in
**Dataverse** — no portal, no form.

### 3. View my tickets (unified)
The employee asks for status. fxDesk retrieves their incidents and service requests from
**both ServiceNow and Pega** and presents them in one unified view.

---

## What Makes It Different — Dynamic, Metadata-Driven Adaptive Cards

fxDesk's Adaptive Cards are **not hardcoded**. An **Azure Function** sits between the agent
and the backend systems and builds each card at runtime from the backend's own UI metadata:

- **Pega:** the function calls **Pega's DX API**, which creates a case and returns its UI
  metadata; the function transforms that metadata into Adaptive Card schema.
- **ServiceNow:** the function calls the **ServiceNow Table API** to retrieve the incident
  fields, selects the required (configurable) fields, and transforms them into Adaptive
  Card schema.

The function backs create, submit, and view operations across both systems.

**The payoff:** any UI change made in ServiceNow or Pega — a new field, an updated form — is
reflected in the agent's card **automatically**, with no manual card rebuilding and no
developer rework. The backend system stays the single source of truth, and the agent's UI
stays in sync with it on its own. This applies to both the incident cards and the My Tickets
view.

> **Demo hosting note:** the function is designed as an Azure Function. For this demo it is
> run locally and exposed via an ngrok tunnel rather than deployed to an Azure subscription
> — no paid Azure resources are needed to reproduce the demo. The design is unchanged.

---

## Architecture

![fxDesk Architecture](fxDesk_Architecture.png)

**Entry point:** an employee chats with fxDesk in Microsoft Teams.

**Orchestrator:** the fxDesk agent in Copilot Studio, using generative orchestration,
interprets each request and routes it to the right connected agent.

**Connected agents:**
- **Incident Agent** — troubleshoots from a knowledge base, creates and escalates incidents
  to ServiceNow/Pega, and powers the My Tickets unified view. Renders results as dynamic
  Adaptive Cards.
- **Service Request Agent** — creates structured service-request records in Dataverse using
  the out-of-the-box Dataverse MCP server, added as a tool on the connected agent.

**Dynamic Adaptive Card engine:** an Azure Function reads ServiceNow/Pega UI metadata and
transforms it into Adaptive Card schema at runtime (Pega via DX API, ServiceNow via Table
API), backing create, submit, and view operations.

**Backend systems of record:** ServiceNow and Pega (incidents); Microsoft Dataverse
(service requests, written via the Dataverse MCP server).

**Data flow:**
`Employee (Teams) → fxDesk orchestrator (Copilot Studio) → connected agent. Incidents go to
ServiceNow / Pega via the Incident Agent (with the Azure Function rendering dynamic Adaptive
Cards from backend UI metadata); service requests go to Dataverse via the Service Request
Agent using the Dataverse MCP server.`

---

## Tech Stack

- **Microsoft Copilot Studio** — orchestrator + connected agents, generative orchestration
- **Connected agents** — multi-agent design (Incident Agent, Service Request Agent)
- **Model Context Protocol (MCP)** — the out-of-the-box Dataverse MCP server, used by the Service Request Agent to create request records in Dataverse
- **Azure Functions** — metadata-driven dynamic Adaptive Card generation
- **ServiceNow** — incidents (Table API)
- **Pega** — incidents (DX API)
- **Microsoft Dataverse** — stores service requests (written via the Dataverse MCP server)
- **Microsoft Teams** — deployment channel
- **Adaptive Cards** — rich, interactive responses rendered dynamically

---

## Agent Academy Concepts Applied (Operative track)

- **Authoring Agent Instructions (Mission 02)** — precise orchestrator and connected-agent instructions to control routing and behaviour (including result-display rules to avoid duplicate confirmations)
- **Multi-agent ready with connected agents (Mission 03)** — a generative orchestrator coordinating an Incident Agent and a Service Request Agent
- **Integrate with MCP Servers (Mission 10)** — the out-of-the-box Dataverse MCP server, added as a tool on the connected Service Request Agent, creates request records in Dataverse
- **Obtain User Feedback with Adaptive Cards (Mission 11)** — Adaptive Cards for rich responses, extended into dynamic, metadata-driven cards generated at runtime from ServiceNow and Pega UI metadata
- **Publishing to Microsoft Teams**

---

## Repository Contents

| Path | Description |
|---|---|
| `solution/` | Exported Copilot Studio solution (.zip) — orchestrator, connected agents, and flows |
| `azure-function/` | Description of the Azure Function that reads ServiceNow/Pega UI metadata and builds Adaptive Card schema (design only — see "External Dependencies") |
| `fxDesk_Architecture.png` | Solution architecture diagram |
| `docs/` | Demo deck (optional) |
| `screenshots/` | Screenshots of the agent in action (optional) |

---

## External Dependencies (not included)

fxDesk integrates with several external systems that are part of the running environment,
not artifacts of this repository. The following are intentionally not included here, as they
are either proprietary, environment-specific, or require their own licensed platforms:

- **Azure Function source code** — the dynamic Adaptive Card engine. Its design and
  behaviour are documented in `azure-function/README.md`, but the implementation is not
  published.
- **Pega application** — the Pega cases, classes, and DX API configuration live in a
  licensed Pega environment and are not part of this repo.
- **ServiceNow environment** — the ServiceNow instance, tables, and Table API configuration
  are part of a licensed ServiceNow environment and are not included.

To run fxDesk end to end, these dependencies must exist in your own environment, with their
endpoints configured via the solution's environment variables (below). This repository
contains the Copilot Studio solution (the orchestrator and connected agents) and full
documentation of the architecture and integrations.

---

## Setup / Import Notes

1. **Import the solution:** in the Power Platform maker portal (make.powerapps.com) →
   Solutions → Import solution → select the .zip in `solution/`. This includes the fxDesk
   orchestrator and the connected Incident and Service Request agents.

2. **Set up the dynamic Adaptive Card function:** the function that generates Adaptive Cards
   from ServiceNow/Pega UI metadata is designed as an Azure Function. Deploy it to your Azure
   subscription, or — for a quick demo without Azure — run it locally and expose it via an
   ngrok tunnel (as used in this project's demo).

3. **Set the environment variables:** provide the endpoint URLs for the dynamic-card function
   operations, so they can be updated without changing the agent or flows:
   - `CreatePegaCaseEndpointURL`, `SubmitPegaCaseEndpointURL`, `GetPegaTicketsEndpointURL`
   - `CreateServiceNowIncidentEndpointURL`, `SubmitServiceNowIncidentEndpointURL`,
     `GetServiceNowIncidentsEndpointURL`

4. **Configure connections:** reconnect/authenticate the connectors and references —
   ServiceNow, Pega, and the Dataverse MCP server.

5. **Set up the knowledge base** used by the Incident Agent for troubleshooting, and confirm
   the Dataverse table for service requests exists in the environment (the Service Request
   Agent writes to it via the Dataverse MCP server).

6. **Publish** the agent to Microsoft Teams.

---

## Demo

A walkthrough video (≤ 5 minutes) demonstrates all three capabilities — incident creation
across ServiceNow and Pega, a service request via the Dataverse MCP server, and the unified
My Tickets view — plus a **live demonstration** of the dynamic Adaptive Card: a new field
added to a Pega process appears in the agent's card instantly, with no code change or
redeployment.

*Demo video: https://youtu.be/PUpcxh2FgMk *
