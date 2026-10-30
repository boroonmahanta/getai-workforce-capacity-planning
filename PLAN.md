# Workforce Demand Forecast App: Plan v0.2

Date: 30 Oct 2026. Owner: Boroon Mahanta. Status: draft for the weekly review.

Changes since v0.1: labour share fixed at 100%, owner and hosting confirmed, contractors treated as fully flexible, and squads modelled from day one.

## 1. Purpose

Give the weekly utilisation and hiring review one forecast of the real people requirement, so practices stop hiring ad hoc and existing capacity is used first.

The one decision it must make easier: **hire, flex or wait**, per role family, per squad and per month. It advises and drafts. A person approves everything.

## 2. Decisions locked

| Topic | Decision |
|---|---|
| Core decision | Hire, flex or wait per role family and month |
| Platform | Standalone web app, built in this repo, hosted in Boroon's Azure tenant |
| Autonomy | Advise and draft. Requisition and candidate-pool drafts need human approval |
| Audience | Internal only. Data and logic kept separate so this can change later |
| Data feeds v1 | Weekly file upload: D365 opportunity export, headcount sheet, utilisation file |
| Owner | Boroon owns the weekly upload and the review |
| Labour share | 100% of contract value is this team's labour |
| Contractors | Casual, no end dates. Treated as flexible capacity: first to grow for peaks, first to release when demand falls |
| Pipeline trust | Calibrate from history. Leads can override a deal; every override is logged |
| Squads | Modelled from day one, alongside role families |
| Deadline | Usable by end of November, to inform the early-December decision |
| Capacity scope | AI Business Solutions and Cloud & AI Platforms, 22 people. Sandeep and Scott as backup for governance work only. The 8 Manila service and security staff are excluded |
| Utilisation target | 80% |
| Hire lead time | 4 to 6 weeks, planned at 6 |

## 3. What the app does

At each weekly review:

1. **Upload** the three files. The app checks them and lists data problems: placeholder durations, start dates in the past, missing fields.
2. **Calibrate** the pipeline. It replaces raw CRM likelihoods with likelihoods adjusted by historical win rates by practice and value band, and slips stale start dates.
3. **Demand engine.** Each open or won opportunity becomes labour days: contract value divided by the blended day rate, phased by month, split into role families using the SOW staffing templates, and run through a simulation to give P10, P50 and P90 demand. The same days are also shown as squad-months.
4. **Supply engine.** People, availability of 18 working days a month, 80% target, contractor flex flags, leave and ramp-up inputs, and the backup rule for Sandeep and Scott.
5. **Squad view.** Supply and demand in squads, with the role that limits the next squad.
6. **Gap and recommendation engine.** Gap by role family, squad and month, then the decision ladder below.
7. **Draft actions.** Requisition drafts, candidate-pool triggers and contractor ramp-up or release notes, each held for approval.
8. **What changed since last week.** Deals that moved, new gaps, closed gaps.
9. **What-if agent.** Questions like "what if Pepper slips 8 weeks" or "what if we lose VenME". The agent changes inputs and reruns the model. It never invents numbers.

## 4. Squad model

Default squad template, configurable:

| Seat | Count | Filled from |
|---|---|---|
| Architect | 1 | Principal and architect roles |
| Engineer | 2 | Developers and data engineers |
| Delivery and change | 1 | Project managers, analysts and strategy consultants |

A squad has 4 people and about 58 billable days a month at 80%.

Current supply from the headcount sheet:

| Seat | People |
|---|---|
| Architects | 5 |
| Engineers | 10 |
| Delivery and change | 7 |

That is 5 full squads, with 2 spare delivery and change people. Total capacity is about 5.5 squad-equivalents at 80%.

How demand looks in squads, at 100% labour share:

| Deal | Weighted view |
|---|---|
| Ventia VenME | About 2.7 squads for 4 months, half of all capacity |
| Pepper Money CCaaS | About 2.1 squads for 6 months |

At the January peak with history-adjusted likelihoods, demand is about 7.2 squad-months against 5.5 available. The shortfall is about 1.7 squads. Engineers limit squad formation, not delivery and change staff, so the hiring ask is "engineers to complete one or two squads", not heads in general.

The app also provides:

- A squad board that assigns named people to squads.
- A flag for governance squads that may draw on Sandeep and Scott.
- A flag for casual contractors, who can be added or released at each review.

## 5. Decision ladder and starting rules

Order of levers for any gap:

1. Redeploy bench and internal-project time.
2. Re-sequence start dates.
3. Use offshore capacity.
4. Pull a vetted candidate from the pool.
5. Use a casual contractor, for peaks only.

Starting thresholds, to be ratified at the first review:

| Signal | Recommendation |
|---|---|
| P50 load within capacity at 80% | Wait |
| Gap only at P90, or a peak shorter than 3 months | Flex |
| P50 gap of 1 FTE or more in a seat for 8 or more weeks, still there after peaks | Hire, with a decision date 6 weeks before the need date |
| P90 gap of 1 FTE or more within 16 weeks | Start or advance interviews so the pool is ready |
| Contractor load under 70% for 6 weeks | Recommend release |

The candidate pool is the buffer. It is held as interviewed candidates, not payroll.

## 6. Architecture

| Layer | Proposal |
|---|---|
| Front end | React web app, weekly review view first |
| Back end | Python API with a tested simulation module |
| Storage | Weekly snapshots of every upload, kept so trends and calibration are possible |
| Auth | Microsoft Entra single sign-on, in Boroon's Azure tenant |
| AI layer | Model on Azure AI Foundry, used only for what-if parsing, explanations and requisition drafts |
| Access | Leadership and practice heads only. Rates and contractor data visible to a finance role |

The model is deterministic code. The AI explains and parses; it does not calculate.

## 7. Data model

Opportunity snapshot, Person, Squad, Squad template, Demand template per service offering, Calibration set, Scenario, Recommendation, Decision log with overrides, Requisition draft.

## 8. Assumptions

| Assumption | Default | Source and risk |
|---|---|---|
| Labour share of contract value | 100% | Instructed. Overstates demand where forward-deployed engineers, partners or licences carry part of the value. The app shows the effect as a sensitivity, so the review sees how much it matters |
| Blended day rate | $1,500 | Averaged from the SOWs. Realised rates ranged from about $1,200 to $1,850 |
| Role mix, AI Business Solutions | PM 17%, BA 8%, architect 20%, developer 55% | SOW templates |
| Role mix, Cloud & AI Platforms | PM 6%, BA 4%, architect 22%, data 68% | SOW templates |
| Start slip | 0 to 8 weeks | Judgement. Replace with history |
| Working days a month | 18 | Net of leave and holidays |
| Win-rate haircut | 0.6 times CRM likelihood | Last 12 months: 18% and 24% by value |
| Contractor availability | On demand, confirmed at each review | No end dates, because they are casual |

## 9. First-cut result the app must reproduce

With history-adjusted likelihoods, demand exceeds capacity from December to March and falls below it from April. At the January peak the shortfall is about 4 developers and 5 data engineers, with architects 1.5 short. Project managers and analysts have spare capacity. Dropping Pepper Money alone cuts the January peak by about 5 people.

## 10. Roadmap to end of November

| Week | Deliverable |
|---|---|
| 2 to 6 Nov | Spec and schema. Importers and data checks. Existing model ported into a tested module. Squad template agreed with Matt and Greg |
| 9 to 13 Nov | Demand and supply engines, simulation, calibration. First forecast screen with the squad view |
| 16 to 20 Nov | Recommendation engine, ladder and triggers, squad board, weekly diff, override log, Entra sign-in |
| 23 to 27 Nov | What-if agent, requisition drafts, dry run of the review on real files, deploy |
| Early Dec | First live review informs the December decision |

If time slips, cut in this order: what-if agent, then requisition drafts. The forecast, the squad view and the recommendation are the minimum.

## 11. Exit-alignment check

- **Talent structure, strengthened.** Squads are the headline unit, and hiring is framed as completing squads.
- **Contractor share, a tension.** 20 of the 35 delivery staff are contractors, which cuts against an acquirer buying talent. Contractor share is on the dashboard with a target.
- **Track record, strengthened.** A pipeline mix view shows pilots and assessments against production work, deal-size bands, and the share of regulated Australian anchor clients.
- **IP portfolio, neutral.** An internal-only app does not build deployment IP. Data, rates and logic stay separate, so reversing the call later is cheap.
- **Ecosystem and regional, neutral.**

## 12. Risks and open items

- No Runn export. Utilisation relies on uploaded files and the Finance billability snapshot, which may be incomplete.
- The 100% labour share is the biggest swing factor in the forecast.
- A handful of deals drive the forecast. The tool must show the deals behind every gap.
- D365 hygiene: 151 of 437 rows have a placeholder duration and 7 open deals have past start dates.
- Rates and contractor data need tight access control.
- The utilisation file format is not yet defined.
