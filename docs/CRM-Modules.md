# ComEx CRM Modules

ConnectPlus is the central hub, but the [app ecosystem](App-Ecosystem.md) describes *apps*, not the CRM *functions* they must cover. This page is the function-level source of truth: seven standard CRM modules, who owns each one, and what is still unowned.

Interactive relational view: **<https://leonard-data.github.io/Comex/#comex-crm-modules.html>** (source [docs/diagrams/comex-crm-modules.html](diagrams/comex-crm-modules.html)).

## Standard taxonomy

Modules 1–4 and 7 follow the [Odoo CRM](https://www.odoo.com/documentation/master/applications/sales/crm.html) function groups, which is the industry-standard breakdown:

| Odoo group | ComEx module |
|---|---|
| Acquire Leads (capture, convert, merge) | 1 · Lead Acquisition |
| Track Leads (scoring, assignment, resellers) | 2 · Qualification |
| Organize the pipeline (stages, lost reasons, teams) | 3 · Pipeline |
| Optimize (activities, enrichment, gamification) | 4 · Activity |
| Performance (win/loss, forecast, expected revenue) | 7 · Performance |

Modules 5 and 6 have no Odoo equivalent — they exist because ComEx operates in a regulated pharma market with HCO/HCP master data, and because ordering runs on a separate platform.

## Modules

| # | Module | Covers | Owning app | State | Pillar |
|---|---|---|---|---|---|
| 1 | **Lead Acquisition** | Capture from events, promo and webinars; dedupe/merge; convert to opportunity | ConnectPlus | Expecting Jan-27 | Leading |
| 2 | **Qualification** | Lead scoring; assignment to rep; routing to sales team | ConnectPlus | Expecting Jan-27 | Leading |
| 3 | **Pipeline** | Stages; forecast; won/lost reasons | ConnectPlus | Expecting Jan-27 | Managing |
| 4 | **Activity** | Visits, calls, webinar follow-up; engagement log | ConnectPlus | Expecting Jan-27 | Training |
| 5 | **[Customer & Territory](Territory-Management.md)** | HCO/HCP master; territory; sales team; rep master | Nexus + ComEx SQL Server, mapped by Dagster | SQL Server live; Nexus expecting Jul-27 | Control |
| 6 | **Quote & Order** | Quotation; order capture | Easy Order | Live | Orders |
| 7 | **Performance** | KPI monitoring; win/loss; forecast; client-level and daily reporting | Eagle Eye, Client reports, Daily report | Live | Analysis |

## Relations between modules

The modules are relational, not a flat list — each one consumes a defined output of another:

| From | To | Passes |
|---|---|---|
| 1 Lead Acquisition | 2 Qualification | Raw leads |
| 2 Qualification | 3 Pipeline | Qualified opportunities |
| 3 Pipeline | 6 Quote & Order | Won deals |
| 6 Quote & Order | 7 Performance | Orders |
| 4 Activity | 3 Pipeline | Logged touchpoints that move a stage |
| 5 Customer & Territory | 2 Qualification | Territory and account that decide the assignee |
| 7 Performance | 4 Activity | KPI feedback that drives the next rep action |
| SFA *(legacy)* | 5 Customer & Territory | Existing customer mapping |

Module 5 is the one every other module depends on: a wrong territory makes modules 2, 3 and 7 wrong at the same time.

## Gaps

1. **Module 2 has no owning app today.** SFA tracks specific teams only, so scoring and assignment rules must be defined before ConnectPlus build closes.
2. **Module 5 is dual-master** until Nexus lands in Jul-27. Which side is authoritative per entity (HCO, HCP, territory, rep) is undecided.
3. **No module owns quotation approval.** DocPromo approves promotional material and MedCompli approves fees — neither approves a deal.

Each gap is tracked on the board as a `Solution` item. Decisions land here, not in the issue.

> Source: Odoo CRM documentation (function taxonomy) · [docs/App-Ecosystem.md](App-Ecosystem.md) (app state and ownership).
