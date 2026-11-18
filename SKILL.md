---
name: t3-delegate
description: Automatically route repository coding tasks from external agents such as OpenClaw or Rakazo to T3 over MCP, even when the user does not mention T3. Use for implementing, fixing, refactoring, testing, reviewing, or investigating a repository, and for resuming delegated work through questions and completion. Do not recursively route work already running inside T3, ordinary coding advice, or skill installation itself.
---

# T3 delegate

Keep repository execution visible in T3 so the user can inspect activity and take
control. The external agent owns the brief, questions, continuity, and final
report; T3 owns repository execution. Enable automatic routing for an agent
when its user explicitly requests it. Once enabled, apply it to each
authorized task in scope.

## Installation and host setup

When installing, connecting, or enabling automatic routing for an external agent,
follow [the agent installation guide](references/install.md). Install the skill,
connect T3 MCP, and merge the routing rule into the host's always-loaded
instructions. Skill discovery alone is not a complete installation. Setup itself
runs in the originating host; do not delegate installation through the connection
being installed. Ordinary task execution can skip this guide.

## Routing and boundaries

- Apply to actionable repository work, including read-only investigations and
  reviews, regardless of task size. Load this routing skill before choosing a
  specialized engineering skill or inspecting implementation files. Put relevant
  specialized workflows in the worker brief. Do preliminary discovery locally
  only as needed to identify the repo and prepare the brief; let T3 perform the
  substantive work. A generic preference against unnecessary subagents does not
  override this user-requested execution route.
- Honor an explicit request to work directly, use a different executor, or stop.
  Direct execution means an explicit local/no-T3 instruction; naming files,
  requesting read-only inspection, or asking for a short answer is not an opt-out.
  Advice without requested execution and non-repository tasks stay with the
  originating agent unless the user asks otherwise.
- If already executing inside T3, do the assigned work there. This skill is not
  permission for recursive delegation, extra reviewers, or new top-level threads.
  Installing or maintaining this skill does not require handing itself to T3.
- Preserve the host's authenticated-user, project allowlist, model, and approval
  rules. Routing grants no additional repository access, publication permission,
  or runtime privileges. Do not silently switch to local execution if T3 fails.
- Copy task-relevant caller prohibitions and approval requirements into every
  worker brief. Private bot instructions are not inherited automatically, and
  `full-access` does not authorize actions the caller must not perform. Use real
  approval controls where required; a prompt is not an enforced access boundary.

## Thread titles

Set the launch/delegation `title` to:

```text
[platform/agent-id] owner/repo: Short task description
```

Examples:

- `[openclaw/dev] example-org/site: Inspect build configuration`
- `[rakazo/coding] example-org/site: Fix mobile navigation`

Use the originating platform and agent ID from trusted runtime context, and the
verified repository identity from project discovery. The example agent IDs are
illustrative, not configured defaults. If the host exposes only the platform,
use `[platform]` rather than inventing an agent ID. Keep the task description
short and specific, in English. If shortening is needed, shorten the task text
and preserve the origin and repository prefix. Keep run IDs, timestamps, status,
and model/provider details in the receipt rather than the title.

Honor an explicitly requested title; otherwise preserve this prefix when a title
changes. Follow-ups reuse the existing thread and title. Record the same origin,
repository, and submitted title in the brief/receipt so attribution survives a
later user rename. Never infer authority or repository access from a title.

## Prepare and submit

1. Discover the connected T3 MCP tools and read their current schemas. Tool names
   below are suffixes; use the actual exposed names. Never invent an endpoint,
   option, callback, or provider. Call `orchestrator_capabilities` before the first
   `delegate_task`, and otherwise when needed to resolve supported execution.
2. Resolve the authorized project with `t3_project_list`, paging if necessary.
   Match repository identity, not just its title. Exclude archived/deleted or
   host-disallowed projects. If the repo is absent or ambiguous, report the exact
   missing project/access or ask which target; do not substitute another repo.
   Match verified origin/remote identity and retain `projectId` plus matched
   remote in the receipt. The install guide's alternate smoke-test project is
   an installation-only exception, not permission to substitute a coding target.
3. Set the title using the format above and fill [the brief](references/brief.md).
   Infer available facts from trusted task context and repository metadata.
   Ask only about consequential missing choices.
   Choose the deliverable from the request: investigation findings, reviewed
   changes, or one PR. Do not turn a review into an edit or publish a local-only task.
4. Choose the host-supported execution route:
   - **External agent without a T3 parent:** launch one visible task thread with
     `t3_thread_launch` when that is the connected API for this authorized T3 job.
     Pass an explicit verified `projectId`. For repository changes, bind
     `workspaceStrategy` to a worktree before starting. Respect repo-specific
     requirements for root/local execution instead of forcing a worktree.
   - **Actual child-task context:** use `delegate_task` when supported and
     authorized by the host. Use `isolation: "worktree"` for independent changes.
     Do not create a top-level thread merely as a substitute for a child.
   - Honor the host's restrictions if a route is unavailable. Report the missing
     capability instead of bypassing it with shell agents or a second executor.
5. Honor the requested base. Independent thread worktrees start from the remote
   default branch with `startFromOrigin: true`, or the local default when no
   remote exists. Dependent/unpublished work uses its required local base with
   `startFromOrigin: false`. A delegated worktree starts at the parent commit;
   do not claim this selects the default branch. Uncommitted edits are not copied.
6. Use the user's selected provider/model or established host default. Do not
   silently substitute an unavailable selection. Inherit permission modes unless
   an authorized host policy specifies otherwise; never escalate for convenience.
7. Generate a stable `clientRequestId` and preserve it before submission. On a
   transport error or lost reply, retry with the same ID and unchanged request.
   Reconcile with existing work before any new launch. A wait timeout is not a
   failed submission and must not create a second task.
8. Save the returned identifiers and share the returned T3 `link` verbatim
   immediately. Include the target repo and what T3 is doing. Do not fabricate a
   link or claim every internal reasoning step is exposed by the activity view.

## Continuity and completion delivery

Keep a compact task receipt in the host's existing task/session store, outside
the target repository. Store request ID, origin reference, environment/project,
task/thread/run IDs, returned link, origin platform/agent ID, submitted title,
repo/base/branch, last read cursor, pending question ID, ownership (`agent` or
`user`), and the observed status. Never include
credentials. On restart, read the receipt and reconcile remote state before
acting; reuse the existing task.

Use the completion mechanism actually supported by this connection:

- When `delegate_task` returns `next` saying completion wakes the parent, follow
  it and end the turn. Do not poll, sleep-loop, or call `task_status` on a running
  task before its completion wake. Read the result on wake. A parent turn ending
  with live child work is not completion.
- For standalone threads, follow submission with `t3_thread_wait` or
  `t3_thread_read` as the tool requires. Prefer a supported asynchronous wait or
  subscription. A bounded wait may return a question, approval, or timeout;
  timeout leaves the task running. Honor returned `next` instructions.
  On a wait timeout, read the same thread and pending requests before the next
  bounded wait when permitted; never create a duplicate job to resume waiting.
- If the external host has no demonstrated resume/delivery mechanism after its
  turn ends, state that automatic notification is unavailable, preserve the
  receipt, and give the T3 link. Resume on the next supported wake or user check.
  Do not promise unattended follow-through, build a poller, or create recurring
  schedules merely because the skill asks for tracking.
- Use incremental `t3_thread_read` with small limits and saved `nextPosition`;
  use `view: "activity"` only when execution details are needed. Report meaningful
  state changes, not invented percentages. Failed, cancelled, interrupted,
  waiting, and idle without an accepted deliverable are not successful completion.

## Questions, approvals, and user control

- When waiting for a question, use `t3_pending_request` to list/read the exact
  request. Answer with `respond` and its actual answer schema only when the brief
  already supplies the answer or it is a routine choice within scope. Relay
  consequential questions to the user with choices, consequences, and the T3 link.
  Re-read before responding so a question already answered in T3 is not answered
  twice. Do not use an ordinary message as a substitute for a pending response.
- Question responses cannot approve permissions. Surface approval requests to
  the authorized user in the supported approval UI; never fabricate approval or
  increase access modes to unblock the worker. Use the host's protected secret
  entry flow when needed; never ask for credentials in chat or put them in briefs.
- On an explicit user takeover, mark ownership `user`. Stop automated sends,
  question answers, retries, and corrective launches. Leave the existing worker
  available to the user; takeover alone does not cancel it. Read-only observation
  is fine. If a read reveals direct user steering that conflicts with the stored
  brief, defer to that steering rather than overwriting it.
- Resume active coordination only when the user hands it back. First read the
  latest thread, runs, questions, and branch state; do not replay the old brief
  over intervening user changes. Do not infer takeover merely from a page being
  opened, and do not claim to detect it when the host exposes no such event.
- For an authorized correction, read first, then steer the same running thread
  with `t3_thread_send`; avoid `restart` unless needed and authorized. Respect
  child-task rules that require a fresh delegation for a new review/re-check.
  Explicit cancellation uses the owning API (`task_cancel` for delegated tasks).
  Do not relaunch cancelled work without a new instruction.

## Verify and report

Read the final result before reporting completion. Check acceptance criteria
against the returned evidence, actual head SHA, and relevant test outcomes.
Use accessible PR/check tools for actionable verification; clearly label anything
unverified. A worker's claim alone is not evidence of a passing check.

Return the T3 link, outcome, PR or artifact/findings as requested, branch/head when
relevant, verification commands and results, and specific remaining uncertainty
or blocker. Follow the host's PR-linking and monitoring rules; creating a PR is
distinct from its CI or review passing. Never merge or deploy under this skill's
default authority. Missing access or a failed check must identify the concrete
next action and who can supply it.
