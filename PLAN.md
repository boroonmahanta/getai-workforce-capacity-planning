# Workforce Demand Forecast App: Plan v0.4

Date: 30 Oct 2026. Owner: Boroon Mahanta. Status: draft for the weekly review.

Changes since v0.1: labour share fixed at 100%, owner and hosting confirmed, contractors treated as fully flexible, squads modelled from day one, and the squad template assumed agreed by the practice leaders. v0.3 defines the utilisation file from the Runn timesheet snapshot already in hand and adds a measured baseline. v0.4 takes three outside contractors out of core capacity and removes Nirmani Ranchagoda until her return in January.

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
| Data feeds v1 | Weekly file upload: D365 opportunity export, headcount sheet, utilisation file in the Runn timesheet format below |
| Owner | Boroon owns the weekly upload and the review |
| Labour share | 100% of contract value is this team's labour |
| Contractors | Casual, no end dates. Treated as flexible capacity: first to grow for peaks, first to release when demand falls |
| Pipeline trust | Calibrate from history. Leads can override a deal; every override is logged |
| Squads | Modelled from day one, alongside role families. Template assumed agreed by the practice leaders |
| Deadline | Usable by end of November, to inform the early-December decision |
| Capacity scope | AI Business Solutions and Cloud & AI Platforms: 18 core people now, 19 from January. Sandeep and Scott as backup for governance work only. The 8 Manila service and security staff are excluded |
| External flex pool | David Gaviria, Juan Puerta and Paul Nash are outside contractors used on demand. They are not in core capacity or the utilisation base |
| Leave | Nirmani Ranchagoda is on parental leave and returns in January. She is excluded from supply until then |
| Utilisation target | 80% |
| Hire lead time | 4 to 6 weeks, planned at 6 |

## 3. What the app does

At each weekly review:

1. **Upload** the three files. The app checks them and lists data problems: placeholder durations, start dates in the past, missing fields.
2. **Calibrate** the pipeline. It replaces raw CRM likelihoods with likelihoods adjusted by historical win rates by practice and value band, and slips stale start dates.
3. **Demand engine.** Each open or won opportunity becomes labour days: contract value divided by the blended day rate, phased by month, split into role families using the SOW staffing templates, and run through a simulation to give P10, P50 and P90 demand. The same days are also shown as squad-months.
4. **Supply engine.** People, availability of 18 working days a month, 80% target, an external flex pool for outside contractors, leave and return dates, ramp-up inputs, and the backup rule for Sandeep and Scott.
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

Core supply, excluding the outside contractors and people on leave:

| Seat | Now | From January | Outside contractors, on demand |
|---|---|---|---|
| Architects | 4 | 4 | 1 |
| Engineers | 7 | 8 | 2 |
| Delivery and change | 7 | 7 | 0 |

That is 3 full squads now, limited by engineers, and 4 from January, with 3 spare delivery and change people. Total capacity is about 4.5 squad-equivalents at 80% now and 4.75 from January.

How demand looks in squads, at 100% labour share:

| Deal | Weighted view |
|---|---|
| Ventia VenME | About 2.7 squads for 4 months, more than half of core capacity |
| Pepper Money CCaaS | About 2.1 squads for 6 months |

At the January peak with history-adjusted likelihoods, demand is about 7.8 squad-months against 4.75 available. The shortfall is about 3 squads. Engineers limit squad formation, not delivery and change staff, so the ask is engineers to complete squads, not heads in general. The three outside contractors cover about 3 of the roughly 12 people short.

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
| Leave and returns | Entered as availability dates | Nirmani Ranchagoda returns in January, assumed first week |
| Realisation factor | Measured weekly from the utilisation file | About 0.72 on the first check. Default stays at 100% as instructed |

## 9. First-cut result the app must reproduce

With history-adjusted likelihoods and core supply only, demand exceeds capacity from December to March and falls below it from April. At the January peak the shortfall is about 6 developers, 5 data engineers and 2.5 architects, roughly 12 people in total. Project managers and analysts have spare capacity. Dropping Pepper Money alone cuts the January peak by about 5 people.

Monthly gap, history-adjusted:

| Month | Nov | Dec | Jan | Feb | Mar | Apr | May | Jun |
|---|---|---|---|---|---|---|---|---|
| Total gap in people | -2 | +6 | +12 | +6 | +1 | -3 | -7 | -11 |
| Developers | +3.5 | +4.0 | +6.3 | +4.0 | +2.3 | +2.0 | +1.0 | -0.4 |
| Data engineers | -1.0 | +3.8 | +4.6 | +3.4 | +2.2 | 0.0 | -1.8 | -2.6 |
| Architects | -0.7 | +1.1 | +2.5 | +1.3 | +0.3 | -0.6 | -1.5 | -2.3 |

## 9A. Utilisation file format

The file is the Runn timesheet extract that Finance already publishes in the billability dashboard on the Finance team site. The snapshot dated 7 Oct 2026 has 260 rows covering six weeks from 24 Aug to 28 Sep 2026. One row is one person, one project and one week.

| Column | Example |
|---|---|
| Project team | AI Business Solutions |
| Client | Brisbane Airport |
| Project | Operational Governance (T&M) |
| Person primary team | AI Business Solutions |
| Person | Alejandro Moran |
| Billable flag | Billable or Non-Billable |
| Pricing model | Time and Materials, Fixed Price or Non-Billable |
| Delivery status | Active |
| Week beginning | 24/08/2026 |
| Hours | 4 |

The app accepts a CSV with these columns. Until Finance publishes a CSV, it can read the dashboard's embedded data. Internal time is any row where the client is GET AI Pty Ltd.

The latest week in the snapshot is 28 Sep. The first live review needs an updated extract.

## 9B. Measured baseline from that file

For the 18 core people over the six weeks to 28 Sep, at 40 hours a week each:

| Measure | Value |
|---|---|
| Billable hours | 1,385 |
| Internal hours | 958 |
| Pre-sales and other non-billable hours | 178 |
| Billable utilisation | 32% |
| Target | 80% |
| Billable days a month | About 125, against about 259 capacity at 80% |

The three outside contractors and Nirmani Ranchagoda are excluded from this base. The outside contractors do log time in Runn. "David Puerta" is Juan Puerta, confirmed. The flex pool billed only 77 hours in six weeks, about 10 days, and David Gaviria and Paul Nash logged nothing and are confirmed idle, so the pool is almost entirely unused. The three casuals also log fewer hours than 40 a week, which understates their utilisation.

Calibration check: the CRM-derived model gives about 239 days for the same period from won deals. Actual billable days were about 173. So at 100% labour share the model runs about 40% high. The app will compute this realisation factor from the file every week and show it as a sensitivity. The default stays at 100% as instructed.

## 10. Roadmap to end of November

| Week | Deliverable |
|---|---|
| 2 to 6 Nov | Spec and schema. Importers, an alias table for names that differ between Runn and the headcount sheet, and data checks. Existing model ported into a tested module. Squad template taken as agreed |
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

- No direct Runn export. Utilisation relies on the Finance billability extract, which ends at 28 Sep and may be incomplete.
- The 100% labour share is the biggest swing factor in the forecast.
- A handful of deals drive the forecast. The tool must show the deals behind every gap.
- D365 hygiene: 151 of 437 rows have a placeholder duration and 7 open deals have past start dates.
- Rates and contractor data need tight access control.
- Runn names do not always match the headcount sheet. "David Puerta" in Runn logged 77 billable hours on Brisbane Airport and Graebel but is not in the sheet. He is Juan Puerta, confirmed. "Matt Noble" in Runn is Matthew Noble in the sheet. The importer needs an alias table.
- The squad template agreement is an assumption. Confirm it in writing before the first live review.
