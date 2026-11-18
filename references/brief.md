# Repository task brief

Use this as the T3 task/message, filling only relevant fields. The worker receives
no assumed parent history. Keep secrets and unrelated conversation out.
Set the separate launch/delegation `title` argument to
`[platform/agent-id] owner/repo: Short task description` as defined in the skill.

```text
# Brief: [short task title]

Execution context:
- This is the T3 worker. Execute here; do not route the task back through
  t3-delegate or start additional agents unless explicitly authorized.
- Request ID: [stable submission identifier]
- Origin: [trusted platform and agent ID; task reference if available]
- Submitted thread title: [same title passed to the launch/delegation tool]
- Repository: [verified owner/repo and T3 project]
- Base: [branch/commit; explain required unpublished dependencies]
- Deliverable: [one PR, local changes, review findings, or investigation report]
- Caller authority: [allowed actions; approval-required actions and approval
  status; prohibitions copied from the originating agent for this task]

Goal:
[Observable outcome in 1-2 sentences.]

Why:
[Problem and reason, enough for judgment calls.]

Context:
- [Relevant entry points and known reproduction]
- [Existing data/APIs and relevant issue or artifact links]

Constraints:
- Follow repository instructions and nearby conventions.
- The caller's private approval rules are not inherited automatically. Preserve
  the explicit caller authority above regardless of the worker runtime mode.
- [Restate applicable production, DNS, visibility, push and deletion restrictions;
  omit inapplicable examples rather than treating them as new global policy.]
- Do not touch: [specific exclusions, if any].
- Preserve: [required behavior and preferences].
- Avoid new dependencies unless needed; justify additions.
- Keep code, comments, commits, and documentation in English.
- Run the relevant checks; report exact commands, outcomes, and omissions.

Done when:
- [Observable acceptance criteria]
- [Appropriate test/lint/build checks pass]
- [Proof: relevant test, log excerpt, or requested visual artifact]

Out of scope:
- [Explicit non-goals]

Questions and blockers:
- Resolve routine choices from the brief and repository.
- Ask for consequential missing decisions or access; do not guess.
- Make questions visible using the supported pending-question mechanism.
- Report what is blocked, what was tried, and exactly what is needed.

Deliver:
- [For an authorized PR: one PR with what changed, why, and how tested;
  return its URL, branch, head SHA, and check/review state.]
- [For review/investigation: findings with evidence and relevant locations.]
- Call out uncertainties and incomplete acceptance criteria.
- Do not merge or deploy.
```
