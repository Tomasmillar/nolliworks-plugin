# The ontology, in brief

The database is the authority. Call `describe_ontology` for the exact props, vocabularies and ends; this page says how the pieces fit, as of ontology 1.4.0.

## Scope: where a node lives

| Scope | Means | Types |
|---|---|---|
| project | Belongs to one job. Create with `project_id`. | Action, Appointment, Assumption, BriefItem, Decision, Deliverable, Duty, Element, Enquiry, Fee, GateSignoff, Outcome, PlanningApplication, Project, Risk, StageInstance, Strategy, ValueEntry, WorkPackage, and every Source (Correspondence, Document, Drawing, Meeting, Transmittal, Utterance) |
| practice | Shared between one practice's jobs, never seen by another practice. Create with `organisation_id`, then tie it to a job with an edge. | Site, Person, Constraint, Buildup |
| shared | A practice's own row or a platform-wide one (policy, authorities). Practices can't create global rows. | Organisation, Requirement |

Abstract types (you can't create them, but edges use them as shorthand): **Source** (Correspondence, Document, Drawing, Meeting, Transmittal, Utterance), **Obligation** (Constraint, Requirement, BriefItem), **Party** (Person, Organisation).

A project's id is also its Project node's id and the `project_id` every node on it carries.

A **PlanningApplication** is one application to the planning authority: its references, dates, decision and appeal. Its status (preparing, submitted, under consideration, decided, withdrawn) is derived from its dates. Its decision is recorded on the job's Planning page, which files the decision notice and the Outcome with it; prefer that page to writing `decision` yourself. A planning condition is a Constraint of kind `planning_condition`, with `number`, `trigger` and `discharged_on`.

## The edges that carry the reasoning

| Edge | From → To | Reads as |
|---|---|---|
| caused_by | Decision, Strategy, Action, Duty, Document, Element → Decision, Source, Obligation, Assumption, Outcome, Action | "because of". **Needs a source.** |
| sourced_from | Obligation, Deliverable, Assumption, Strategy, Decision, ValueEntry, PlanningApplication → Source | "on the evidence of". **Needs a source.** An application's decision comes from its decision notice. |
| assumes | Decision, Document, Strategy → Assumption | "rests on" |
| addresses | Decision, Strategy, Deliverable, Document, PlanningApplication → Obligation | "deals with". A discharge of conditions, a section 73 variation or a non-material amendment addresses the planning conditions it deals with. |
| imposed_by | Constraint of kind planning_condition → PlanningApplication (one) | "was imposed by": the application whose decision, or whose appeal's, imposed the condition |
| discharged_by | Constraint of kind planning_condition → PlanningApplication (one) | "was discharged by": the discharge of conditions whose granted decision discharged it. Written and taken away by the Planning page with the decision; never write it yourself. |
| supersedes | new → old, same kind (Decision, Strategy, Assumption, Document, Requirement, Action, PlanningApplication) | "replaces". At most one each way. |
| depends_on | Action, Deliverable → Action, Deliverable, Decision | "waits on" |
| blocks | Deliverable, Action, Decision → StageInstance, GateSignoff, Decision, Deliverable | "holds up" |
| applies_to | Document, Deliverable, Strategy, Fee, Action, Duty, Risk, Correspondence, Meeting, Utterance, Transmittal → Site, Project, WorkPackage, StageInstance, Element, Appointment, Deliverable, BriefItem, Action, Duty, Risk, Enquiry, PlanningApplication | "about" |
| constrained_by | Site, Project, Element → Obligation | "bound by". **Needs a source.** Constraints are read from the Site. |
| satisfied_by | Deliverable → Document, Transmittal | "received as". **Needs a source.** At most one. |
| satisfies | Document, Element → Requirement | "meets". **Needs a source.** |
| resulted_in | Project, Decision, Strategy, PlanningApplication → Outcome | "led to". **Needs a source.** A decision on an application leads to the permission or refusal, from the job as well. |

## When and where

| Edge | From → To |
|---|---|
| made_during | Decision → StageInstance (one) |
| decided_in | Decision → Meeting (one) |
| produced_during | Document, Deliverable → StageInstance (one) |
| captured_during | Document, Utterance → Meeting (one) |
| minuted_in | Meeting → Document (one) |
| has_stage / follows | Project, WorkPackage → StageInstance / StageInstance → previous StageInstance |
| required_by | Deliverable → StageInstance |
| concerns | Project → Site |
| part_of | WorkPackage, Site, Deliverable, BriefItem, Action, Duty → parent of the same family (one) |

## People and responsibility

| Edge | From → To |
|---|---|
| engaged_on | Party → Project, WorkPackage (the role is a prop on the edge; director or manager only) |
| client_of | Person, Organisation → Project |
| member_of | Person → Organisation (one) |
| responsible_for | Party, Appointment → Deliverable, Action, Duty, Document, Site, Decision, Element, Risk |
| owes | Party, Appointment → Deliverable |
| signed_off_by | Decision, Document, Fee, GateSignoff, Deliverable, Strategy → Person (one; director or manager only) |
| attended | Person → Meeting |
| authored_by / addressed_to | Source → Party, Appointment |
| recorded_by | written by the database from the actor props (proposed_by, confirmed_by…); don't write it yourself |

## Design and specification

| Edge | From → To |
|---|---|
| specifies | Decision → Element (one) |
| depicts | Drawing → Element |
| issued_in | Drawing → Transmittal |
| based_on | Element → Buildup (one) |
| priced_in | Element → Document |
| reused_from | Decision, Strategy, Document, Assumption, Element → the same, or a Project (precedent) |

## Truth and time

- **status**: `confirmed` (a person vouched for it), `stated` (on someone's word), `inferred` (inbox only, never counts), `retired` (closed).
- **valid_from / valid_to**: world time, when the fact held. **recorded_at**: when it was typed in. `why` with `as_of` reads the record as at a date.
- **Cardinality "one"** (`at_most_one`): a second live edge isn't refused; it becomes a contradiction finding. Close the old edge first when replacing.
- Edges may not cross jobs unless the type allows it (concerns, constrained_by, engaged_on, client_of, member_of, attended, authored_by, addressed_to, addresses, satisfies, reused_from, candidate_for, proposed_by, administered_by, recorded_by).
