# Workforce Demand Forecast App: Plan v0.1

Date: 30 Oct 2026. Owner: Boroon Mahanta. Status: draft for the weekly review.

## 1. Purpose

Give the weekly utilisation and hiring review one forecast of the real people requirement, so practices stop hiring ad hoc and existing capacity is used first.

The one decision it must make easier: **hire, flex or wait**, per role family and per month. It advises and drafts. A person approves everything.

## 2. Decisions locked in the grilling

| Topic | Decision |
|---|---|
| Core decision | Hire, flex or wait per role family and month |
| Platform | Standalone web app, built in this repo |
| Autonomy | Advise and draft. Requisition and candidate-pool drafts need human approval |
| Audience | Internal only. Data and logic kept separate so this can change later |
| Data feeds v1 | Weekly file upload: D365 opportunity export, headcount sheet, utilisation file |
| Contractors | Flex capacity. First to grow for peaks, first to release when demand falls |
| Pipeline trust | Calibrate from history. Leads can override a deal; every override is logged |
| Deadline | Usable by end of November, to inform the early-December decision |
| Capacity scope | AI Business Solutions and Cloud & AI Platforms, 22 people. Sandeep and Scott as backup for governance work only. The 8 Manila service and security staff are excluded |
| Utilisation target | 80% |
| Hire lead time | 4 to 6 weeks, planned at 6 |

## 3. What the app does

At each weekly review:

1. **Upload** the three files. The app checks them and lists data problems: placeholder durations, start dates in the past, missing fields.
2. **Calibrate** the pipeline. It replaces raw CRM likelihoods with likelihoods adjusted by historical win rates by practice and value band, and slips stale start dates.
3. **Demand engine.** Each open or won opportunity becomes labour days: contract value, times the labour share for its service offering, divided by the blended day rate. Days are phased by month, split into role families using the SOW staffing templates, and run through a simulation to give P10, P50 and P90 demand.
4. **Supply engine.** People, availability of 18 working days a month, 80% target, contractor flex flags, leave and ramp-up inputs, and the backup rule for Sandeep and Scott.
5. **Gap and recommendation engine.** Gap by role family and month, then the decision ladder below.
6. **Draft actions.** Requisition drafts, candidate-pool triggers and contractor extension or release notes, each held for approval.
7. **What changed since last week.** Deals that moved, new gaps, closed gaps.
8. **What-if agent.** Questions like "what if Pepper slips 8 weeks" or "what if we lose VenME". The agent changes inputs and reruns the model. It never invents numbers.

## 4. Decision ladder and starting rules

Order of levers for any gap:

1. Redeploy bench and internal-project time.
2. Re-sequence start dates.
3. Use offshore capacity.
4. Pull a vetted candidate from the pool.
5. Use a contractor, for peaks only.

Starting thresholds, to be ratified at the first review:

| Signal | Recommendation |
|---|---|
| P50 load within capacity at 80% | Wait |
| Gap only at P90, or a peak shorter than 3 months | Flex |
| P50 gap of 1 FTE or more in a role family for 8 or more weeks, still there after peaks | Hire, with a decision date 6 weeks before the need date |
| P90 gap of 1 FTE or more within 16 weeks | Start or advance interviews so the pool is ready |
| Contractor load under 70% for 6 weeks | Recommend release |

The candidate pool is the buffer. It is held as interviewed candidates, not payroll.

## 5. Architecture

| Layer | Proposal |
|---|---|
| Front end | React web app, weekly review view first |
| Back end | Python API with a tested simulation module |
| Storage | Weekly snapshots of every upload, kept so trends and calibration are possible |
| Auth | Microsoft Entra single sign-on, hosted in Azure |
| AI layer | Model on Azure AI Foundry, used only for what-if parsing, explanations and requisition drafts |
| Access | Leadership and practice heads only. Rates and contractor data visible to a finance role |

The model is deterministic code. The AI explains and parses; it does not calculate.

## 6. Data model

Opportunity snapshot, Person, Demand template per service offering, Calibration set, Scenario, Recommendation, Decision log with overrides, Requisition draft.

## 7. Assumptions to calibrate

| Assumption | Default | Source and risk |
|---|---|---|
| Blended day rate | $1,500 | Averaged from the SOWs. Realised rates ranged from about $1,200 to $1,850 |
| Labour share of contract value | 100% | Too high for forward-deployed engineering, contact centre, licensing and partner-delivered work |
| Role mix, AI Business Solutions | PM 17%, BA 8%, architect 20%, developer 55% | SOW templates |
| Role mix, Cloud & AI Platforms | PM 6%, BA 4%, architect 22%, data 68% | SOW templates |
| Start slip | 0 to 8 weeks | Judgement. Replace with history |
| Working days a month | 18 | Net of leave and holidays |
| Win-rate haircut | 0.6 times CRM likelihood | Last 12 months: 18% and 24% by value |

## 8. First-cut result the app must reproduce

With history-adjusted likelihoods, demand exceeds capacity from December to March and falls below it from April. At the January peak the shortfall is about 4 developers and 5 data engineers, with architects 1.5 short. Project managers and analysts have spare capacity. Dropping Pepper Money alone cuts the January peak by about 5 people.

## 9. Roadmap to end of November

| Week | Deliverable |
|---|---|
| 2 to 6 Nov | Spec and schema. Importers and data checks. Existing model ported into a tested module. Labour-share workshop with Matt and Greg |
| 9 to 13 Nov | Demand and supply engines, simulation, calibration. First forecast screen |
| 16 to 20 Nov | Recommendation engine, ladder and triggers, weekly diff, override log, Entra sign-in |
| 23 to 27 Nov | What-if agent, requisition drafts, dry run of the review on real files, deploy |
| Early Dec | First live review informs the December decision |

If time slips, cut in this order: what-if agent, then requisition drafts. The forecast and the recommendation are the minimum.

## 10. Exit-alignment check

- **Talent structure, strengthened if done right.** Show supply and demand a second way, in forward-deployed squads of one architect, two engineers and one change specialist. Frame hiring as squads, not heads.
- **Contractor share, a tension.** 20 of the 35 delivery staff are contractors, which cuts against an acquirer buying talent. Put contractor share on the dashboard with a target.
- **Track record, strengthened.** Add a pipeline mix view: pilots and assessments against production work, deal-size bands, and share of regulated Australian anchor clients.
- **IP portfolio, neutral.** An internal-only app does not build deployment IP. Data, rates and logic stay separate, so reversing the call later is cheap.
- **Ecosystem and regional, neutral.**

## 11. Risks and open items

- No Runn export. Utilisation relies on uploaded files and the Finance billability snapshot, which may be incomplete.
- Labour share per service offering is unknown and moves the answer a lot.
- A handful of deals drive the forecast. The tool must show the deals behind every gap.
- D365 hygiene: 151 of 437 rows have a placeholder duration and 7 open deals have past start dates.
- Rates and contractor data need tight access control.
- Someone must own the weekly upload and the review.

## 12. Inputs needed

1. Labour share of contract value by service offering, from Matt Noble and Greg Nash.
2. Named owner of the weekly upload and the review.
3. Azure subscription and Entra tenant for hosting.
4. Contract end dates for contractors.
5. Whether to build in squad units from day one.
