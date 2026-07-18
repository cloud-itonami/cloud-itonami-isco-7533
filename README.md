# cloud-itonami-isco-7533

Open Occupation Blueprint for **ISCO-08 7533**: Sewing, Embroidery and Related Workers.

This repository designs a forkable OSS business for a sewing/embroidery workshop scheduling and logistics coordination practice: a workshop scheduling and supply-coordination robot manages crew/task records under a governor-gated actor, so a sewing/embroidery workshop crew keeps its own operating records instead of renting a closed workforce-management SaaS.

ISCO-08 7533 workers operate sewing and embroidery machines — a standard workshop hazard profile (needle-puncture injury, moving-machine-part injury) applies, so this actor's hazard reporting names that profile explicitly. This actor never operates a sewing/embroidery machine, never performs sewing or embroidery work, and never makes a safety-clearance decision — it coordinates workshop scheduling/logistics only.

**Maturity: `:implemented`.** `src/sewcoord/` implements the
`SewCoordActor` as a `langgraph.graph/state-graph`
(`sewcoord.actor`) wired to a `Sewing Coordination Advisor`
(`sewcoord.advisor`) and an independent `SewCoordGovernor`
(`sewcoord.governor`), following the itonami actor pattern
(ADR-2607121000): `:intake -> :advise -> :govern -> :decide -+-> :commit
(:ok?) +-> :request-approval (:escalate?, human-in-the-loop interrupt)
+-> :hold (:hard?)`. HARD invariants (always hold, never
overridable): worker provenance, workshop provenance, no-actuation
(`:effect` must be `:propose`), a closed op-allowlist
(`:log-work-record`, `:schedule-crew-operation`,
`:flag-safety-concern`, `:coordinate-supply-order` — nothing else may
ever be proposed), and a permanent, unconditional block on any
proposal that would directly finalize a sewing/embroidery-execution
decision (e.g. deciding a sewing or embroidery job is finished) or a
workshop-safety-clearance decision (e.g. declaring a workshop or item
safety-cleared), or override a shop safety officer's judgment.
Always-escalate paths (human sign-off regardless of confidence,
mapping this repo's Trust Controls in
[`docs/business-model.md`](docs/business-model.md)):
`:flag-safety-concern` (always) and `:coordinate-supply-order` above
the registered cost threshold.

## Robotics premise

All cloud-itonami verticals are designed on the premise that a **robot performs
the physical domain work**. Here a workshop scheduling/logistics coordination robot performs crew scheduling, job/commission/progress-record logging and thread/fabric-materials supply-order coordination for a sewing/embroidery workshop crew, under an actor that proposes actions and an independent **Sewing Coordination Governor** that gates them. The governor never
dispatches hardware itself, never operates a sewing/embroidery machine or performs sewing/embroidery work on the shop floor, and never finalizes a sewing/embroidery-execution decision or a workshop-safety-clearance decision, nor overrides a shop safety officer's judgment; `:high`/`:safety-critical` actions (such as a flagged needle-puncture/machine-part/equipment-condition concern, or an above-threshold supply order) require human sign-off. **This actor coordinates workshop scheduling/logistics only — it never performs sewing or embroidery work or makes safety-clearance decisions itself.**

## Core Contract

```text
crew roster + workshop registration + safety-reporting policy
        |
        v
Sewing Coordination Advisor -> SewCoordGovernor -> log/schedule/coordinate, or human sign-off
        |
        v
robot actions (gated) + operating records + audit ledger
```

No automated advice can dispatch a robot action the governor refuses, finalize
a sewing/embroidery-execution decision, declare a workshop safety-clearance,
override a shop safety officer's judgment, suppress an operating record, or
disclose sensitive data without governor approval and audit evidence.

## Capability layer

Resolves via [`kotoba-lang/occupation`](https://github.com/kotoba-lang/occupation)
(ISCO-08 `7533`). Required capabilities:

- :robotics
- :identity
- :audit-ledger

See [`docs/business-model.md`](docs/business-model.md) and
[`docs/operator-guide.md`](docs/operator-guide.md).

## License

AGPL-3.0-or-later.
