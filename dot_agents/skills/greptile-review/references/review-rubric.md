# Review Rubric

## Evidence before priority

Report defects introduced or worsened by the reviewed change that a maintainer can act on.
Compare base and head when origin is unclear. A previously missed defect introduced earlier in
this PR is in scope; an unrelated pre-existing defect is not a PR finding. Mention a material
pre-existing risk separately only when it affects understanding or validation of this change.

A finding must identify a reachable trigger, causal code path, and concrete consequence.
Try to disprove it before reporting. Conditional bugs are valid when the condition is supported
and stated explicitly. Unresolved speculation is a question, not a low-priority finding.

## Priority definitions

These are this skill's review priorities, not a claim about an external service's rubric.

- **P0 — Critical blocker:** urgent, broad failure such as unavoidable data corruption, a
  widespread outage, or a directly exploitable critical security failure. Requires demonstrated
  reachability and impact, not merely a theoretical worst case.
- **P1 — High:** a demonstrated serious regression, security gap, broken core path, or data loss
  under realistic conditions. Should be fixed before merge.
- **P2 — Medium:** a concrete, bounded correctness, reliability, performance, or contract defect
  with a supported trigger. Worth fixing, but less urgent than P1.
- **P3 — Low:** a concrete minor issue; suppress unless requested or no higher findings exist.

Severity and evidence strength are independent. Do not downgrade a guess into a finding.
The default posting policy requests changes only for P0; P1 can still say "needs changes before
merge" in a COMMENT review. Do not inflate severity to obtain a blocking GitHub event.

## Risk-directed checks

Use the checks relevant to the change; they are investigation prompts, not automatic findings.

- **Contracts:** callers and external consumers still handle signatures, return shapes,
  nullability, exceptions, units, ordering, defaults, renamed exports, routes and config keys.
  Trace dynamic registration and serialization where text search is insufficient.
- **Correctness:** boundary values, partial failure, cleanup, precision, timezone and encoding.
  Verify changed behavior is applied across the paths required by the stated feature.
- **State and concurrency:** transactions, uniqueness constraints, idempotency, retries,
  check-then-act races, stale caches and concurrent writers. Name a plausible interleaving.
- **Security:** authorization at the actual trust boundary, tenant isolation, injection,
  sensitive output, path/URL validation. Identify the attacker-controlled input and missing
  defense; do not demand controls without a relevant threat path.
- **Persistence and rollout:** migration compatibility, existing rows, null/default semantics,
  indexes on real query paths, mixed application versions and worker/message compatibility.
- **Performance:** supported input scale and query/loop behavior. Explain the cost increase;
  do not call every unindexed query or nested loop a defect.
- **Tests:** assertions that miss the claimed behavior, fakes that violate the relied-on
  production contract, and tests that pass with the implementation broken. Missing coverage
  alone is not a finding; name the specific regression or misleading test claim it exposes.
- **Errors and observability:** recovery or propagation required by callers, and failures that
  silently lose work. Logging preferences alone are not actionable defects.

## Recorded decisions and architecture

Intentional behavior changes are not automatically bugs. A stated intent can still be
implemented incorrectly or violate a separately established contract; report that distinction.

Treat explicit product, deployment and scope decisions as settled within their stated scope.
Record who decided and the remaining implication once. Reopen only with new evidence, a changed
requirement, or an unaddressed concrete consequence, explaining why the prior answer is no longer
sufficient. A reply saying "fixed" and a resolved thread both require code verification.

For costly-to-reverse architecture choices (public APIs, persistent data, dependency boundaries),
identify the existing contract and concrete downstream cost. Put unresolved choices in a decision
section. Use a defect priority only when there is a demonstrated failure or violated requirement.
An internal abstraction or layer preference is not sufficient.

## Exclude

Pure style, formatter issues, praise, diff narration, generic hardening, speculative refactors,
coverage percentages, missing tests without a concrete failure, and duplicated open findings.
Repository patterns inform investigation; divergence alone does not establish a defect.
