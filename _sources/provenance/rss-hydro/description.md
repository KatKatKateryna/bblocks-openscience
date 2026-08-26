# OSPD Provenance Profile — RSS-Hydro implementation

A constrained profile of the OGC PROV building block (`ogc.ogc-utils.prov`) for
recording the processing steps of a geospatial workflow.

The baseline PROV building block is deliberately permissive: it accepts nested
objects or identifier references interchangeably, allows three alternative type
discriminators (`provType`, `prov:type`, `type`), and requires only that a node
carries *at least one* of a long list of properties. That flexibility is right
for a general-purpose datatype that other schemas mix in. It is not sufficient
for validation, automated discovery, or AI-assisted metadata generation, which
was the principal finding carried into OSPD 2026 from the previous phase.

This profile tightens the baseline along five axes. Each is a design decision
rather than a necessary consequence of PROV, and each is stated separately so
that divergence between the parallel implementations can be attributed to a
specific choice.

## Status within the pilot

This is one of the parallel implementations of the OSPD 2026 generic provenance
Building Block (Activity 1). It is deliberately *not* named as the canonical
profile: divergence between the parallel implementations is the point of the
exercise, and any consolidation is a matter for the Lead Architect at
convergence. Sibling implementations sit alongside this one under
`_sources/provenance/`.

## 1. Flat, identifier-linked graph only

A provenance record is an **array** of nodes. Cross-references between nodes are
identifiers, never inline objects.

*Rationale.* The nested form makes the same node representable in several
distinct ways, so two records describing identical provenance need not be
comparable. It also makes it impossible to state that an entity was consumed by
two activities without duplicating it. Flattening is the single change that most
improves the schema's usefulness for validation and for graph queries.

*Cost.* Records are less readable by eye, and a record fragment can no longer be
embedded directly as an annotation on a feature. Consumers needing the embedded
form must dereference.

## 2. Single canonical type discriminator

`provType` is required on every node and must be an unprefixed string, not an
array. `prov:type` and `type` are forbidden as discriminators.

*Rationale.* One discriminator means one code path. It also allows the three
node shapes to be distinguished by `oneOf` without ambiguity, which the baseline
cannot guarantee.

## 3. Mandatory process typing

Every Activity must carry `activityType`, an IRI or CURIE.

*Rationale.* This is the hook into the OSPD process-type register. Without it,
the provenance record says that *something* happened but not *what kind of
thing*, which is precisely the gap this phase exists to close. During Activity 4
these values are replaced by register URIs; until the register exists they
should use a local namespace.

## 4. Mandatory execution context

Every Activity must carry `startedAtTime`, `endedAtTime` and `wasAssociatedWith`.

*Rationale.* An activity with no responsible agent and no temporal extent cannot
be audited, reproduced, or ordered relative to its neighbours. Note that this
constrains *who* and *when*, not *where* — execution-environment description is
out of scope for this phase.

## 5. Single-direction edges

Generation is stated once, on the Entity (`wasGeneratedBy`). The inverse
property `generated` is forbidden on Activity.

*Rationale.* PROV permits both directions. Permitting both in a profile means
records can disagree with themselves, and consumers must reconcile. Choosing the
entity-centric direction keeps every entity self-describing: an entity node
alone answers "where did this come from".

*Open question for the pilot.* The activity-centric direction is equally
defensible and may suit workflow engines that emit per-step logs. If another
implementation chose the opposite direction, that divergence is a candidate for
resolution at profile level rather than a defect in either implementation.

## 6. Role disambiguation for multi-input activities

An Activity with two or more entries in `used` must also carry
`qualifiedUsage`, with `hadRole` distinguishing the inputs.

*Rationale.* Most real geospatial operations take heterogeneous inputs — a
field to be transformed, a reference grid, a parameter set. A bare `used` array
loses which was which, and the loss is not recoverable from the output. This is
the constraint most likely to expose limitations in the baseline building block,
because `qualifiedUsage` is comparatively under-exercised.

## Constraints expressed outside JSON Schema

Three classes of constraint are graph-level and are carried in `rules.shacl`
rather than the schema:

- an activity must not end before it started;
- an activity must not consume an entity generated after the activity began;
- every referenced identifier must resolve to a node in the same record.

JSON Schema can constrain the shape of a document but not relationships between
values in sibling array members. This split is itself an implementation finding:
a profile of PROV that is validated by schema alone will under-constrain, and any
adopter needs both artefacts.

## Entity identity

Entities represent *versioned* data objects. An identifier must denote a single
immutable state; a dataset that is periodically updated is a series of entities
related by `wasDerivedFrom` or `wasRevisionOf`, not one entity that changes. The
profile does not enforce this — it cannot — but conformance depends on it.

## Out of scope

Consistent with the scope discipline set out in the Call for Participation, this
profile addresses processing steps only. Data discovery, execution-environment
and platform description, and input/output schema description are not modelled
here. Where a workflow is executed through OGC API — Processes, the linkage
between a process description and the activities recorded here is a matter for
the OGC API Processes profile rather than this building block.
