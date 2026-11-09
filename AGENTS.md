# Working agreement — ComEx source of truth

Read by **agents** (this file is auto-loaded by Claude Code / DSH) and by **people**. Same rules for both.

Repo: `Leonard-Data/Comex` (private) · Board: **ComEx Ecosystem** `https://github.com/users/Leonard-Data/projects/6`

## 1. Where a thing goes

| The thing is… | Goes to | How |
|---|---|---|
| Why / what it means / how an app works | `docs/` | PR editing a page in [docs/](docs/) |
| Work to be done or tracked | Board (project 6) | item with **Category** set |
| Something broken, blocked or requested | Issue | issue form + `type:*` label |
| Code | this repo | PR |

One rule: **decisions in `docs/`, work on the board, problems in issues.** Never duplicate across two.

## 2. Category — the kanban axis

Every board item carries exactly one **Category**:

| Category | Means | Matching issue label |
|---|---|---|
| `Solution` | end-to-end delivery / integration / migration | `type:solution` |
| `Wiki` | a `docs/` page to write or revise | `type:wiki` |
| `Business` | process, policy, ownership, vendor | `type:business` |
| `Feature` | new capability in one app | `type:feature` |
| `Ticket` | request routed to another team | `type:ticket` |
| `Document` | SOP, spec, deck, contract | `type:document` |
| `Bug` | defect in a live app | `type:bug` |
| `Access` | permission request or failure | `access` |
| `Data` | master data, mapping, pipeline correctness | `type:data` |
| `Chore` | housekeeping, no business outcome | `type:chore` |

Other fields: **App**, **Phase** (Idea→Scoping→Build→UAT→Live→Legacy), **Pillar** (Orders/Training/Leading/Analysis/Control/Managing), **Owner**, **Start date**, **Target date**, **Status** (Todo/In Progress/Done).

## 3. Board views

Projects v2 views cannot be created by API — the three below are **set up once in the UI**, then never touched again.

| View | Layout | Config |
|---|---|---|
| **Kanban** | Board | Group by **Category**, filter `-status:Done` |
| **Projects** | Table | Columns: Title, Category, App, Phase, Pillar, Owner, Target date |
| **Roadmap** | Roadmap | Group by **App**, dates **Start date** → **Target date**, zoom Quarter |

## 4. Commands (agent and user, same syntax)

```bash
# shells here have empty $HOME
export GH="HOME=/root gh"

# add an item and categorise it
HOME=/root gh project item-create 6 --owner Leonard-Data --title "Eagle Eye: regional KPI drill-down" --body "…"
HOME=/root gh project item-list 6 --owner Leonard-Data --format json --jq '.items[]|{id,title,category}'

# field + option ids needed for item-edit
HOME=/root gh project field-list 6 --owner Leonard-Data --format json \
  --jq '.fields[]|select(.options)|{name,id,opts:[.options[]|{name,id}]}'
HOME=/root gh project item-edit --project-id <PROJECT_ID> --id <ITEM_ID> \
  --field-id <CATEGORY_FIELD_ID> --single-select-option-id <OPTION_ID>

# issues
HOME=/root gh issue create -R Leonard-Data/Comex --label type:bug --title "…" --body "…"
HOME=/root gh issue list -R Leonard-Data/Comex --label type:solution
```

## 5. Rules for an agent working here

1. **Search before writing.** `docs/` and the board already hold the answer more often than not. Read [docs/App-Ecosystem.md](docs/App-Ecosystem.md) before touching anything about an app.
2. **Never invent ecosystem facts.** App states and owners come from [docs/App-Ecosystem.md](docs/App-Ecosystem.md). If reality differs, change that page in the same PR — it is the source of truth, not a comment.
3. **Every board item gets a Category** at creation. An uncategorised item is invisible on the Kanban view.
4. **Issues close with the trace filled in.** A `type:bug` or `type:solution` issue closes only when root cause and resolution are written in it — that is the trace, and the next incident is diagnosed from it.
5. **Don't open a second place for the same thing.** No `TODO.md`, no roadmap table in the README, no status spreadsheet. Link to the board.
6. **Owner is a person, not a team**, except `Client reports` (AIS team).

## 6. Moving to an organisation

The board lives under the user `Leonard-Data`. When ComEx gets a GitHub org:

```bash
# 1. transfer the repo (keeps issues, labels, PRs, history)
HOME=/root gh repo transfer Leonard-Data/Comex <ORG>   # or Settings → Transfer ownership

# 2. the project does NOT move. Recreate it at org level and relink:
HOME=/root gh project copy 6 --source-owner Leonard-Data --target-owner <ORG> --title "ComEx Ecosystem"
HOME=/root gh project link <NEW_NUMBER> --owner <ORG> --repo <ORG>/Comex
```

`project copy` carries fields and views but **not items** — re-add them, or export first with
`gh project item-list 6 --owner Leonard-Data --format json > items.json`.

After the move: enable the real **Wiki** (available on org repos; a *private user* repo needs GitHub Pro — that is why business context sits in `docs/` today) and mirror `docs/` into it. Set org teams to `write` on the repo and `admin` on the project, then update every `--owner Leonard-Data` in this file.
