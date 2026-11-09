# Recipes

Worked sequences for the requests that come up most. Ids come from earlier calls; never invent one.

## "Why did we go with the blue roof on Kingsway?"

1. `list_projects` {query: "Kingsway"} → project id.
2. `search` {query: "blue roof"} → the Strategy or Decision.
3. `why` {project_id, node_id}.
4. Answer as a chain, nearest reason first: each step's label, the edge in words ("because of", "rests on", "on the evidence of"), the quote and its source, and whether it's confirmed or stated. End with any step that has no source ("…is recorded, but nothing on the job says where it came from").

## "If the build cost goes to £4,100/m², what does that touch?"

1. `list_nodes` {type: "Assumption", project_id} → find the cost assumption.
2. `impact` {project_id, node_id} → what rests on it.
3. Report what would be affected, grouped by type (decisions, documents, deliverables).
4. Only if the user then says to change it: `update_node` {id, patch: {amount: {amount: 4100, currency: "GBP"}, as_at: "<date>", basis: "<basis>"}}. Check `describe_ontology` {node_type: "Assumption"} for the money shape first. If the new figure came from a source rather than the user, `add_source` and `propose` a `prop_update` instead.

## "Here's the minutes from Tuesday's design team meeting, log them against Oakridge."

1. `add_source` {project_id, kind: "meeting", date: "<Tuesday's date>", text, title}.
2. Read the context: `list_nodes` for Decision, Action, Assumption, Deliverable and Strategy on the job; `stage_status` for the live stage.
3. `propose` with the decisions (kind from `decision_kind`), actions (with `due_on` when the minutes give one), changed dates or figures as `prop_update`, and a `question` for anything ambiguous. Each one quotes the minutes exactly.
4. Reply with a short list of what's waiting in the inbox and the confidence you gave each.

## "What's stopping Stage 4 sign-off?"

1. `stage_status` {project_id}.
2. List failing **blocking** checks first, then advisory ones, each with its findings in the record's words. Note which advisory checks rest on stated facts.
3. Offer the next steps the graph supports (link a document with `satisfied_by`, add a source for an unsourced assumption), but don't take them unasked.

## "Ben confirmed by phone that the CCTV survey came in, it's the PDF we filed yesterday."

1. Find the Deliverable (`list_nodes` {type: "Deliverable", project_id}) and the Document (`search`).
2. The user is telling you in terms, so write directly: `create_edge` {type: "satisfied_by", from_id: deliverable, to_id: document, source_node_id: document, snippet: "<what identifies it>", status: "confirmed", valid_from: "<date received>"}.
3. If you also want the call on the record: `add_source` {kind: "call", …} first and use it as the `source_node_id`.

## "That decision was replaced, D-041 now says timber frame."

1. `get_node` on D-041 → its kind, the stage, what it rests on.
2. `impact` on D-041 → tell the user what rests on it.
3. With the user's go-ahead: `create_node` {type: "Decision", project_id, props: {title, statement, kind, decided_on, state: "live"}}, then `create_edge` {type: "supersedes", from_id: new, to_id: D-041, status: "confirmed"} and `caused_by` to whatever led to the change (a source or the user's account of it).
4. The database marks D-041 superseded and closes its live edges except the historical ones. Say so.

## "Link the noise survey to the Kingsway site as a constraint."

Constraints are practice-scoped and bound to a Site:

1. `describe_ontology` {node_type: "Constraint"} → required props and `constraint_kind` values.
2. `create_node` {type: "Constraint", organisation_id, props: {...}}.
3. `create_edge` {type: "constrained_by", from_id: site, to_id: constraint, source_node_id: the survey document, snippet}. Gate checks read constraints from the Site, so tie it there, not to the project.
4. If you read the constraint out of the survey rather than being told, `propose` a Constraint node instead, with an edge `{type: "constrained_by", dir: "in", other_id: <site id>}` so a person confirms the binding too. The link to the source is added for you.

## A link was written wrongly

`close_edge` {id, reason: "Linked to the wrong stage; should be Stage 3"} and `create_edge` the right one. Don't delete either node.
