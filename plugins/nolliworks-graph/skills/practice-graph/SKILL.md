---
name: practice-graph
description: How to read and write a Nolliworks practice graph through the nolliworks-graph MCP tools. Use whenever the user asks about one of their architecture jobs (a project, stage, gate, decision, strategy, assumption, deliverable, action, risk, site, constraint, consultant or source), asks why something on a job is the way it is or what a change would affect, wants an email, letter, meeting note or report recorded against a job, or asks to add, connect, change or withdraw something on the record.
---

# Working on the practice graph

The practice graph holds an architectural practice's jobs as typed **nodes** joined by typed **edges**, each edge with provenance (who said so, on what evidence) and dates (when it became true, when it stopped). The tools act as the signed-in person: they see what that person can see, and everything written is recorded as theirs. The database enforces the ontology, scoping and permissions; when it refuses, the refusal comes back as the tool's error. Read it, fix the call, and don't work around it.

Three principles govern everything below:

1. **A model's reading is a proposal, not a fact.** What you read in a source or worked out yourself goes to the job's inbox (`add_source` → `propose`) for a person to confirm. Write directly only what the user has told you in terms.
2. **Nothing is deleted.** A fact that stops being true is *closed* (`close_edge`, with a reason) or *superseded* (a new node with a `supersedes` edge to the old one). The history is the point.
3. **The graph knows the record, not the job.** "Nothing on the job records a drainage survey" is right. "The drainage survey is missing" or "hasn't been done" is wrong: maybe it exists and nobody linked it. Never say *missing*, *not done*, *does not exist*, *no one has* or *failed to* about the world.

## Start every session the same way

1. `whoami`: confirms the connection and lists the person's practices. If they belong to several, pass `organisation_id` to practice-wide tools.
2. Find the job: `list_projects` (by name), or `search` for a node by words.
3. Before any write, `describe_ontology` with the `node_type` or `edge_type` you mean to use. It gives required props, vocabulary values, which edges may start or end on a type, cardinality, and whether an edge needs a source. **Don't guess prop names or enum values.**

If a tool says the connection must be confirmed past the second factor, give the user the link it names and wait. You can't do that step for them.

## What you can do

| The user wants… | Tools | Notes |
|---|---|---|
| To know what's on a job | `list_projects`, `list_nodes` (type + project_id), `search`, `get_node` | `get_node` is the working view: a node plus every live edge in and out, with the far end named. |
| **Why** something is so | `why` (project_id, node_id, optional `as_of`) | Walks back through caused_by, assumes, sourced_from… to the sources, with each step's quote and who confirmed it. `as_of` reads the record as it stood on a date. |
| **What a change affects** | `impact` | What rests on a node today. Run it before changing an assumption's figure or replacing a decision, and tell the user what it found. |
| Stage readiness / what stands before a gate | `stage_status` | Live stage(s), each gate check (blocking or advisory, passed or not) and the findings. Report findings as statements about the record. Blocking checks count only *confirmed* edges. |
| To record an email, letter, minutes, report, call or note | `add_source` | Use the source's own date, not today's. The same text twice is recognised and the existing one returned. Set `privileged` when the user says it must not be read by AI. |
| To pull decisions, actions, assumptions… out of a source | `add_source` → read it → `propose` | See *Reading a source* below. |
| To see or decide what waits | `list_inbox`, `confirm_proposal`, `reject_proposal` | Confirm only when the user has said *this one is right*. Confirming writes it as their fact. |
| To add something they've told you | `create_node`, then `create_edge` to tie it in | Project-scoped types need `project_id`. Practice-scoped ones (Site, Person, Constraint, Buildup) take `organisation_id` and must be tied to the job with an edge. |
| To connect two things | `create_edge` | `status: "confirmed"` only when the user has said in terms that it's so. Otherwise leave it `stated`. Give `source_node_id` + `snippet` whenever a source says so (required for `caused_by`, `sourced_from`, `constrained_by`, `satisfied_by`, `satisfies`, `resulted_in`). Set `valid_from` to when it became true. |
| To change a value | `update_node` (merge patch; `null` removes a key) | Run `impact` first if anything rests on it. For an assumption whose figure moved, consider proposing the change through the inbox instead. |
| To withdraw a fact or fix a wrong link | `close_edge` with a reason | Never delete a node to undo a link. |
| To replace a decision, strategy, assumption or document | `create_node` for the new one + `create_edge` `supersedes` (new → old) | The database moves the old one to superseded and closes its live edges, except the historical ones it carries forward. |
| To remove something entered in error | `delete_node` | Only when the user asks for that node to be removed. It's a soft delete. |

## Reading a source into the inbox

When the user gives you an email, minutes, a letter or a report about a job:

1. `add_source` with the right `kind`, the source's date and its full text. Keep the returned `source_node_id`.
2. Read the job's context before proposing, so you link to what's already there instead of duplicating it: `list_nodes` for Decision, Strategy, Assumption, Deliverable and Action on the job, and `stage_status` for the live stage. Use `search` for anything the source names.
3. `propose` up to 12 items, each with an **exact quote** from the source in `snippet` and your own confidence:
   - `node`: a new Decision, Assumption, Action, Deliverable, Strategy or Constraint, with its required props (check `describe_ontology`) and up to six links to existing nodes. The link back to the source is added for you.
   - `edge`: a connection between two existing nodes (caused_by, addresses, assumes, signed_off_by, supersedes, decided_in, made_during, depends_on, blocks, applies_to, constrained_by).
   - `prop_update`: a changed `amount`/`text_value`/`as_at`/`basis` on an Assumption, `expected_on` on a Deliverable, `due_on` on an Action.
   - `question`: when you can't tell who or what is meant. Ask rather than guess.
4. Calibrate confidence honestly: 0.8+ when the source says it plainly; 0.45–0.8 when it's implied; 0.2–0.45 when unsure. Below 0.2 isn't written. Confidence only orders the inbox; it never makes anything true.
5. Tell the user what you proposed and that it waits in the job's inbox. Don't confirm it yourself unless they ask you to, item by item or "confirm all of those".

If `propose` refuses a link because the ontology doesn't allow those ends, its error lists the edges that do run between those types. Pick the one that means what the source says, or drop the link.

## Modelling guidance

Read `references/ontology.md` for the types and edges that matter most and how they fit together. In short:

- **Decision**: a fork taken. Tie it to *why* (`caused_by` → a Source, Obligation, Assumption or earlier Decision), *when* (`made_during` → StageInstance, `decided_in` → Meeting) and *what it rests on* (`assumes` → Assumption). A live decision with no `caused_by` fails the blocking `decision-traced` check, and one that's costly or irreversible needs `signed_off_by` (`decision-signed-off`).
- **Assumption**: a figure not yet a fact (cost/m², GIA, U-value…), with a `basis` (evidenced, benchmarked, estimated, placeholder) and `as_at`. Each must say where it came from (`assumption-sourced`), and no placeholder may still carry the design or budget at the gate (`assumption-not-placeholder`). Both checks are blocking.
- **Strategy**: a position held over time (fire, drainage, planning, energy). It's superseded, not reversed.
- **Obligation** (abstract): a Constraint (about the land: practice-scoped, bound to a Site via `constrained_by`), a Requirement (policy or regulation, shared) or a BriefItem (from the client). `addresses` → Obligation says what deals with it.
- **Deliverable**: something owed. Its status is *derived*: it becomes received only through `satisfied_by` → a Document, and overdue only if it has an `expected_on`.
- **Source**: Correspondence, Meeting, Document (Drawing, Transmittal), Utterance. Every fact should trace back to one.

Some edges record authority (`signed_off_by`, `closed_by`, `started_by`, `has_stage`, `engaged_on`). Only the job's director or a manager may write them. If the database refuses, say so and suggest who can.

Contradictions are allowed: two people can confirm facts that can't both hold. Don't resolve one by choosing; tell the user, and close the wrong edge only on their word. Usually the fix is a missing `valid_to` on the older fact.

## Saying what you found

- Name things as the record does (the `label` the tools return), and cite the source and its quote when explaining *why*.
- Separate what is **confirmed**, what is **stated** (on someone's word) and what only **waits in the inbox**.
- Phrase gaps with the templates in `references/phrasing.md`.
- After writing, say exactly what you wrote (type, label, edges) and that it's recorded as the user.

For worked examples of common requests, see `references/recipes.md`.
