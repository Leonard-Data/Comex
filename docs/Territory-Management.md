# Territory Management

**Territory management is the rule that decides which sales rep owns which customer.** Everything downstream — lead routing, pipeline, order scope, KPI — reads that decision. In ComEx it is [CRM module 5](CRM-Modules.md), *Customer & Territory*, and it is the module every other module depends on.

Interactive views:

- **Data model** — <https://leonard-data.github.io/Comex/#comex-territory-model.html> (source [docs/diagrams/comex-territory-model.html](diagrams/comex-territory-model.html))
- **Position in the ecosystem** — <https://leonard-data.github.io/Comex/#comex-territory-ecosystem.html> (source [docs/diagrams/comex-territory-ecosystem.html](diagrams/comex-territory-ecosystem.html))

## 1. What it is, in business terms

A territory is not a map region. It is an **economic zone**: a set of accounts assigned to a rep with a defined visit cadence and a defined revenue expectation. Drawing lines on state borders or splitting by equal surface area produces unbalanced books; territories are balanced on **account density, workload, travel feasibility and pipeline potential**.

Five things a working territory practice must do:

| Pillar | What it means | Fails as |
|---|---|---|
| **Segmentation** | Tier accounts by revenue potential, lifetime value, strategic fit | Tier-1 accounts get the same cadence as Tier-3 |
| **Balanced alignment** | Distribute potential evenly across reps, no overlap, no gap | One rep drowns, another starves |
| **Route / cadence planning** | Cluster visits geographically, cadence by tier (e.g. T1/30d, T2/60d, T3/120d) | Windshield time replaces selling time |
| **Point-of-contact capture** | Log the visit at the visit, not on the weekend | Forecast is built on stale records |
| **Review and realignment** | Periodic rebalance when accounts, headcount or market move | Imbalance is discovered at quarter end |

Measure the outcome, not the effort: revenue yield per territory and Tier-1 penetration, not visits logged or miles driven.

## 2. Data model — why it is data-driven

Salesforce's Enterprise Territory Management (ETM, "TM 2.0"; the original Territory Management was retired in Summer '21) is the reference model, and it is worth copying because it makes territory a **versioned data structure**, not a configuration setting.

| Object | Represents | Key fields |
|---|---|---|
| **Territory2Model** | A whole territory structure as one version | `State` = Planning / Activating / Active / Archived, `ActivatedDate`, `LastRunRulesEndDate` |
| **Territory2** | One territory | `ParentTerritory2Id` (hierarchy), `Territory2ModelId`, `Territory2TypeId`, `ForecastUserId`, `AccountAccessLevel`, `OpportunityAccessLevel`, `CaseAccessLevel`, `ContactAccessLevel` |
| **Territory2Type** | Category of territory (geographic, named account, overlay) | `Priority` (int, unique) |
| **ObjectTerritory2AssignmentRule** | The criteria that assign records automatically | `BooleanFilter` (e.g. "(1 AND 2) OR 3"), `IsActive` |
| **ObjectTerritory2Association** | A record ↔ territory link | `ObjectId`, `SobjectType` (Account, Lead), `AssociationCause` = `Territory2AssignmentRule` or `Territory2Manual` |
| **UserTerritory2Association** | A rep ↔ territory link | `UserId`, `RoleInTerritory2` = Owner / Administrator / Sales Rep, `IsActive` |

Five consequences that matter more than the field list:

1. **A model is a version.** You draft the next alignment in a `Planning` model while the `Active` one keeps running, then activate. Standard users never see a non-active model. Realignment stops being a destructive edit.
2. **Assignment is a rule, not a column.** Active rules evaluate on record create and edit. A record flagged `IsExcludedFromRealign` is skipped. So territory is *derived data* — reproducible from master data plus rules.
3. **Both sides are associations.** Records join territories through one table, people through another. Neither is a field on Account or User, which is why one rep can cover many territories and one account can sit in several.
4. **Territory drives record access.** The access levels live on `Territory2`, so changing the territory changes who can read and write the account. Territory is a *security* object, not only a reporting dimension.
5. **Type priority breaks ties.** For opportunity assignment, the account-assigned territory whose type `Priority` is highest wins; two territories of the same type on one account assign nothing. Priority must be designed, not defaulted.

## 3. How ComEx does it today

| Role | Reference model | ComEx today |
|---|---|---|
| Territory master | `Territory2` + `Territory2Model` | ComEx SQL Server, with Nexus arriving Jul-27 |
| Assignment rules | `ObjectTerritory2AssignmentRule` | Dagster mapping jobs |
| Record ↔ territory | `ObjectTerritory2Association` | Territory map output of Dagster |
| Rep ↔ territory | `UserTerritory2Association` | Nexus rep master + SQL Server |
| Model versioning | `State` Planning → Active → Archived | **nothing equivalent** |
| Access by territory | Access levels on `Territory2` | per-app, not territory-derived |

Flow: **Nexus + ComEx SQL Server → Dagster → territory map → ConnectPlus / Easy Order / Eagle Eye / Daily report.** See [App Ecosystem](App-Ecosystem.md).

### Position in the ecosystem

Territory mapping sits **between master data and every consumer**. It is not a feature of the CRM; the CRM is one of its readers. That is why the blast radius is large:

- wrong territory → module 2 routes the lead to the wrong rep
- wrong territory → modules 3 and 4 misstate coverage and pipeline
- wrong territory → Easy Order shows the wrong customer scope
- wrong territory → Eagle Eye and the Daily report are wrong **at the same time**, and agree with each other, so the error looks like truth

## 4. Gaps

1. **Dual master.** ComEx SQL Server and Nexus both hold territory-adjacent master data until Jul-27; which side is authoritative per entity (HCO, HCP, territory, rep) is undecided. Tracked as a `Solution` board item.
2. **No model versioning.** There is no Planning/Active/Archived equivalent, so a realignment is an in-place edit of live data with no preview and no rollback.
3. **Rules are code, not data.** Assignment logic lives in Dagster jobs rather than declarative, inspectable rules, so a business user cannot read or simulate why an account landed where it did.
4. **Access is not territory-derived.** Each app decides visibility itself, so "who may see this account" cannot be answered from the territory.
5. **No realignment cadence or approval path.** Nobody owns the quarterly territory health review.

## 5. Decisions to make before ConnectPlus closes

- Which entity is mastered where (SQL Server vs Nexus), per entity, written down.
- Whether ComEx adopts a versioned territory model before the first realignment, or accepts in-place edits.
- Territory **types** and their priority order, since that decides opportunity-level assignment.
- Visit cadence SLA per account tier, and who audits it.

> Sources: [What Is Territory Management: Definition, Process, and Best Practices](https://www.ytt-ai.com/post/what-is-territory-management-guide) (pillars, cadence, metrics) · Salesforce Object Reference for [Territory2](https://developer.salesforce.com/docs/atlas.en-us.object_reference.meta/object_reference/sforce_api_objects_territory2.htm), Territory2Model, Territory2Type, ObjectTerritory2Association, ObjectTerritory2AssignmentRule, UserTerritory2Association (data model and field semantics) · ETM overview corroborated by [erp-information](https://www.erp-information.com/salesforce-enterprise-territory-management) and [Calendly](https://calendly.com/blog/territory-management-salesforce). *Salesforce Ben, "Territory Management in Salesforce: 10 Things You Need to Know", and Salesforce Help article 000386766 were requested but are behind a bot challenge / login and could not be read; their content is not represented here.*
