# CalendarMate — AI-Powered Productivity Assistant

**Capstone project — Applied Agentic AI for PMs/TPMs**
**Company context:** ServionIQ Solutions (SaaS platform for professional-services firms)
**Built with:** Langflow (agent orchestration) · OpenAI `gpt-4o-mini` · Google Calendar API ·
Gmail API · Langfuse (observability)

> Platform note: the assignment's reference tooling list suggests either n8n or Langflow.
> This submission uses **Langflow** throughout — all five flows below are Langflow exports.

---

## 1. What CalendarMate Does

| Capability | Type | Agent |
|---|---|---|
| Daily/weekly briefing with conflict detection | Core (required) | Briefing Agent |
| Natural-language scheduling with availability checks, HITL confirmation, Meet link | Core (required) | Scheduler Agent |
| Unread/recent email summarization, action-required vs. FYI | Core (required) | Email Agent |
| Meeting follow-up generation (action items + email) | **Extended (chosen)** | Follow-Up Agent |
| Intent classification and routing | Infrastructure | Orchestrator Agent |

**Orchestration pattern:** Router / Dispatcher — justification in
`docs/Q2_Design_Implementation_Evaluation.docx`, Section 1.3.

---

## 2. Repository Structure

```
CalendarMate/
├── README.md                                  ← you are here
├── diagrams/
│   ├── architecture_diagram.svg                ← editable source
│   └── architecture_diagram.png                ← rendered copy (also embedded in Q2 doc)
├── workflows/                                  ← Langflow flow exports (JSON)
│   ├── orchestrator_flow.json
│   ├── briefing_agent_flow.json
│   ├── scheduler_agent_flow.json
│   ├── email_agent_flow.json
│   └── followup_agent_flow.json
├── prompts/
│   └── system_prompts.md                       ← full system prompts + grounding-rule rationale
├── test-results/
│   ├── baseline_test_results.md                ← all 12 baseline tests + 6 edge cases + prompt experiments
│   └── mock_data.json                          ← fixture data used to produce documented outputs
└── docs/
    ├── Q1_Business_Case.docx
    ├── Q2_Design_Implementation_Evaluation.docx
    ├── Q3_Program_Charter.docx
    └── Q4_Reflection.docx
```

---

## 3. How to Run This in Langflow

1. Install Langflow: `pip install langflow` (or use the hosted Langflow Cloud).
2. In the Langflow UI, use **Import Flow** and load each JSON file from `/workflows`
   in this order: `briefing_agent_flow.json`, `scheduler_agent_flow.json`,
   `email_agent_flow.json`, `followup_agent_flow.json`, then `orchestrator_flow.json`
   last (it references the other four as sub-flows).
3. Set the following environment variables / Langflow global variables:
   - `OPENAI_API_KEY`
   - `GOOGLE_OAUTH_TOKEN` (per-user OAuth2 token, Calendar + Gmail scopes)
   - `LANGFUSE_PUBLIC_KEY`, `LANGFUSE_SECRET_KEY`
4. For local testing without a live Workspace tenant, point the HTTP Request nodes'
   base URL at a mock server seeded from `test-results/mock_data.json` — this is
   exactly how the baseline results in this repo were produced. Swapping that one
   URL is the only change needed to go from mock data to a real Google Workspace
   account.
5. Open the Orchestrator flow's chat panel and try any of the 12 baseline inputs
   from `test-results/baseline_test_results.md` to reproduce the documented outputs.

---

## 4. Design Highlights

- **Grounding is enforced at two layers, not one.** Every agent prompt has explicit
  "only reference what's in X, say so if X is empty" rules (see `prompts/system_prompts.md`),
  *and* several agents have a code-level guard upstream of the LLM (e.g., the Email
  Agent's Empty-Result Guard) so the system doesn't rely on the LLM alone to refuse
  to hallucinate.
- **Human-in-the-loop gates every consequential write.** No calendar event is created
  and no follow-up email is sent without an explicit confirmation step — this maps
  directly to 5 of the 12 baseline test cases (T3, T5, T6, T9, T11) that specifically
  check for this behavior.
- **The Orchestrator's confidence threshold is a cost control, not just a UX one.**
  See the "Prompt Modification Experiments" section of `test-results/baseline_test_results.md`
  for measured evidence of this.

---

## 5. Assignment Deliverable Map

| Deliverable | Location |
|---|---|
| Q1 — Business Case | `docs/Q1_Business_Case.docx` |
| Q2 — Architecture diagram | `diagrams/architecture_diagram.svg` / `.png` |
| Q2 — Orchestration pattern + tool rationale + agent design | `docs/Q2_Design_Implementation_Evaluation.docx` §1 |
| Q2 — Workflow JSON exports | `workflows/*.json` |
| Q2 — Observability integration | `docs/Q2_Design_Implementation_Evaluation.docx` §3.1 |
| Q2 — Baseline dataset + edge cases + prompt experiments | `test-results/baseline_test_results.md` |
| Q2 — Cost and evaluation strategy | `docs/Q2_Design_Implementation_Evaluation.docx` §4 |
| Q3 — Program Charter | `docs/Q3_Program_Charter.docx` |
| Q4 — Reflection | `docs/Q4_Reflection.docx` |
| Screenshots (workflow canvas, CalendarMate in action, observability traces) | To be captured from the live Langflow + Langfuse instance once imported per Section 3 above — see note below |

**Note on screenshots:** this repository was built and evaluated in a sandboxed
environment without a live Langflow UI session or a connected Langfuse project, so
canvas/trace screenshots are not included here. Once the flows are imported per
Section 3, screenshots of (a) each flow's canvas, (b) a live chat interaction for
each of the 3 core capabilities, and (c) the Langfuse trace view filtered to
`prompt_version=v3-final` should be added to a `/screenshots` folder before final
submission.

---

## 6. Known Limitations / v2 Candidates

See `docs/Q4_Reflection.docx` for full discussion. In brief: no event
cancellation/reschedule flow yet, Follow-Up Agent depends on user-provided notes
rather than live transcription, and the Router pattern doesn't yet handle
compound requests that span two agents in one turn.
