# 🤖 Mira — Project Intelligence Assistant

> AI-powered multi-agent system that automates project planning, risk assessment, status reporting, milestone tracking, and stakeholder updates — built in n8n with gpt-4o-mini.

**Built by:** Praveen Kumar Jatta | Senior TPM | PMP® CSPO® SMC®
**Client Scenario:** Nexora Pvt. Ltd. → ABCDE Ltd. AI Adoption Project
**Stack:** n8n · OpenAI gpt-4o-mini · Slack · Python · DeepEval

---

## 🧩 The Problem It Solves

PMs and TPMs at Nexora spend **3–4 hours per project** manually creating plans, compiling risk assessments, and writing weekly status reports. Missed milestones are discovered only in retrospectives. Stakeholder updates are inconsistent.

Mira eliminates all of that.

One workflow. Five specialist agents. Two delivery channels. Fully automated.

---

## 🏗️ Architecture

![Mira Architecture](docs/mira_architecture.png)

```
Manual Trigger  ──→  Set Source (manual)  ──┐
                                             ├──→  Load Project Data  ──→  Trigger Router
Schedule Trigger ──→  Set Source (schedule) ──┘         │                       │
                                                         │               ┌───────┴────────┐
                                                         │          Manual branch    Schedule branch
                                                         │               │                │
                                                         │          Main Router      2pm Digest Agent
                                                         │               │                │
                                                         │     ┌─────────┼──────────┐    Slack
                                                         │     ↓         ↓          ↓  #daily-2pm-digest
                                                         │  Planner  Risk Assessor  Status Reporter
                                                         │  Milestone Tracker  Stakeholder Update
                                                         │                          │
                                                         └──────────────────────────┘
                                                                         │
                                                                    Slack #mira-reports
```

---

## 🤖 The Five Specialist Agents

| Agent | Trigger Keyword | What It Does |
|-------|----------------|--------------|
| 📋 **Status Reporter** | `status` | Reads task board, generates sprint status with blockers |
| ⚠️ **Risk Assessor** | `risk` | Analyzes project risks R01–R10, generates risk matrix |
| 🏁 **Milestone Tracker** | `milestone` | Tracks timeline progress, flags upcoming milestones |
| 🗓️ **Planner** | `plan` | Ingests project description, generates phased project plan |
| 📧 **Stakeholder Update** | `stakeholder` | Writes executive-friendly Sprint update email → Slack |
| 🤖 **2pm Digest** | Schedule (2pm daily) | Auto-generates full project digest → #daily-2pm-digest |

All agents use **gpt-4o-mini** and are grounded in the same project data loaded once at runtime.

---

## 📦 Project Data

The workflow ingests 4 data sources for **ABCDE Ltd. AI Adoption Project** (a logistics company adopting AI):

| Section | Contents |
|---------|----------|
| 📄 **Description** | Company overview, goals, focus areas |
| 📅 **Timeline** | 8 phases, 24 weeks (Initiation → Monitoring) |
| ⚠️ **Risks** | 10 risks (R01–R10) with impact & mitigation |
| 📋 **Task Board** | 25 tasks (T001–T025): 5 Done, 3 In Progress, 16 To Do, 1 Blocked |

> **Ground truth:** T024 (Security review of AI infrastructure) = BLOCKED, awaiting security team.

---

## 🔀 Routing Logic

```
userRequest.toLowerCase() contains...
  "status"      → Status Reporter
  "risk"        → Risk Assessor
  "milestone"   → Milestone Tracker
  "plan"        → Planner
  "stakeholder" → Stakeholder Update → Slack #mira-reports

$json.source === "schedule" → 2pm Digest → Slack #daily-2pm-digest
```

---

## 📊 Baseline Test Results (12 Tests)

| Test | Agent | Input | Result |
|------|-------|-------|--------|
| T1 | Planner | Project plan request | ✅ PASS |
| T2 | Risk Assessor | Top risks | ✅ PASS |
| T3 | Risk Assessor | Mitigation strategies | ✅ PASS |
| T4 | Planner | Phase 3 & 4 deliverables | ✅ PASS |
| T5 | Status Reporter | Sprint 3 status | ⚠️ PARTIAL → ✅ Fixed |
| T6 | Status Reporter | Task counts | ✅ PASS |
| T7 | Risk Assessor | High impact risks | ✅ PASS |
| T8 | Milestone Tracker | Completed milestones | ✅ PASS |
| T9 | Milestone Tracker | Current phase + next | ✅ PASS |
| T10 | Risk Assessor | Full risk report | ✅ PASS |
| T11 | Milestone Tracker | Next 2 weeks milestones | ✅ PASS |
| T12 | Stakeholder Update | Sprint 2 email | ⚠️ PARTIAL → ✅ Fixed |

**Final Score: 12/12 PASS ✅ | Overall: 0.99/1.00**

---

## 🔬 Phase 4 — LLM Judge Evaluation

Evaluated using **DeepEval + gpt-4o-mini** as LLM judge across 3 metrics:

| Metric | Description |
|--------|-------------|
| 🎯 Groundedness | Output grounded in actual project data |
| ✅ Completeness | Covers expected keywords & pass criteria |
| 🔍 Accuracy | Correct info, avoids must-not-contain phrases |

### Fix Ladder Applied

| Level | Approach | Used? |
|-------|----------|-------|
| **L1** | 🔧 Prompt Engineering | ✅ Solved both T5 + T12 |
| L2 | 🔗 Pipeline Restructuring | ⏭️ Not needed |
| L3 | 🔄 Model Swapping | ⏭️ Not needed |
| L4 | 📚 RAG / Better Grounding | ⏭️ Not needed |
| L5 | 🎯 Fine-Tuning | ⏭️ Not needed |

> See [mira-eval](https://github.com/praveenjatta/mira-eval) for the full evaluation framework.

---

## 💰 Cost Analysis

| Component | Model | Est. Cost/Run |
|-----------|-------|--------------|
| 5 specialist agents | gpt-4o-mini | ~$0.002/request |
| 2pm Digest (daily) | gpt-4o-mini | ~$0.004/day |
| **Monthly (1 user)** | | **~$0.20/user/month** |

> Cost scales linearly. At 50 users → ~$10/month total. Highly cost-effective for enterprise TPM workflows.

---

## 🛠️ Tech Stack

| Tool | Purpose |
|------|---------|
| **n8n** | Workflow automation & multi-agent orchestration |
| **OpenAI gpt-4o-mini** | All 6 AI agents |
| **Slack** | Output delivery (#mira-reports + #daily-2pm-digest) |
| **DeepEval** | LLM-judge evaluation framework |
| **Python 3.11** | Evaluation scripts |

---

## 📁 Repository Structure

```
mira-project-intelligence-agent/
├── workflows/
│   └── Mira_Project_Intelligence_Assistant.json   ← n8n workflow export
├── docs/
│   ├── mira_architecture.png                      ← Architecture diagram
│   ├── mira_architecture_light.svg                ← Architecture diagram (SVG)
│   ├── BRD.pdf                                    ← Business Requirements Doc
│   ├── PRD.pdf                                    ← Product Requirements Doc
│   ├── Iteration_Plan.pdf                         ← Sprint planning
│   └── Ground_Truth_Dataset.pdf                   ← 12 test case ground truth
├── tests/
│   ├── T1.txt – T12.txt                           ← All 12 test outputs
│   └── Baseline_Test_Results.pdf                  ← Baseline evaluation report
└── README.md
```

---

## 🚀 How to Run

**Prerequisites:**
- n8n (local via `nvm use 22 && npx n8n` or cloud at n8n.io)
- OpenAI API key
- Slack Bot Token (xoxb-)
- Slack channels: `#mira-reports` and `#daily-2pm-digest`

**Setup Steps:**

1. Import `workflows/Mira_Project_Intelligence_Assistant.json` into n8n
2. Connect credentials:
   - OpenAI API key → all 6 agent nodes
   - Slack Bot Token → Send a message nodes
3. In **Load Project Data** node, update `userRequest` with your query
4. Execute workflow → Manual Trigger for on-demand queries
5. Activate workflow for scheduled 2pm digest

**Sample Queries:**
```
"Generate a project plan for the AI Adoption Project"       → Planner
"What are the top risks we should escalate?"               → Risk Assessor
"Generate a weekly status report for Sprint 3"             → Status Reporter
"What milestones are coming up next?"                      → Milestone Tracker
"Generate a stakeholder update email for Sprint 2"         → Stakeholder Update
```

---

## 🔑 Key Design Decisions

**Single Load Project Data node** — All 4 data sections loaded once and referenced by every agent via `{{ $('Load Project Data').item.json.projectData }}`. Changing data in one place updates all agents automatically.

**Dual-trigger architecture** — Manual trigger for on-demand queries; Schedule trigger for automated 2pm digest. Both converge into the same Load Project Data node before splitting.

**Keyword-based routing** — Simple, transparent, and easy to extend. Adding a new agent = adding a new keyword rule in the Router Switch node.

**Prompt engineering as first fix** — Both T5 and T12 failures were resolved purely through prompt improvements. No pipeline changes, model upgrades, or fine-tuning needed.

**Grounding over generation** — Every agent is instructed to use ONLY the provided project data. Hallucination guardrails prevent fabricated tasks, risks, or milestones.

---

## 🚀 Stretch Goals (Production Enhancements)

- **Webhook trigger** — Accept queries from Slack slash commands or a web UI
- **RAG integration** — Add Pinecone vector search so agents retrieve only relevant data sections
- **Human-in-the-loop** — Add approval step before Stakeholder Update is sent to Slack
- **Multi-project support** — Parameterize project data so Mira can serve multiple projects
- **Model upgrade** — Swap gpt-4o-mini for gpt-4o on complex planning tasks for higher quality

---

## 📊 Related Projects

- 🔬 [mira-eval](https://github.com/praveenjatta/mira-eval) — LLM-judge evaluation framework for Mira

---

## 👤 Author

**Praveen Kumar Jatta** — Senior Technical Program Manager | AI Automation Consultant

- 🌐 [jattaai.com](https://jattaai.com)
- 💼 [linkedin.com/in/praveenjatta](https://linkedin.com/in/praveenjatta)
- 🐙 [github.com/praveenjatta](https://github.com/praveenjatta)
- 📅 [Book a free discovery call](https://calendly.com/praveenjatta/free-ai-automation-discovery-call)

---

## 📄 License

MIT License — free to use and modify with attribution.
