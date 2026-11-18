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
follow [the agent installation guide](#appendix-a-installation-guide). Install the skill,
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
3. Set the title using the format above and fill [the brief](#appendix-b-repository-task-brief).
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

## Appendix A. Installation guide

# Install T3 delegation for an external agent

Use this guide when asked to install or update `t3-delegate`, connect T3 MCP, or
enable automatic repository routing. Reuse the existing connection and policy.
Do not create another skill, scheduler, bridge service, or duplicate MCP server.

## 1. Identify the target

Discover the host platform, agent ID, workspace, instruction entry point, skill
loader, and existing MCP configuration through its supported CLI/API. For
OpenClaw, inspect the actual profile wrapper and `agents list`; do not assume the
agent is named `main`. Change only the requested agent/profile.

On connector hosts such as Rakazo, discover the connected app's tools through
the host's catalog first. If the connection answers a read-only capability or
project call, reuse its credentials: no separate account login or CLI is needed.
Use native skill and bot-instruction tools when exposed. Rakazo's observed tools
include `skill_create`, `skill_update`, `skill_read`, and `update_bot`; inspect
the running schemas rather than assuming these names or permissions are universal.
Account/GitHub import is an alternative when the native route is unavailable,
not a prerequisite. Do not ask for sign-in solely because no local CLI or
account-management surface is available. Filesystem links still do not install
a skill into a remote account.

If a working connector hides its URL or routing metadata, record its connector
reference and leave unexposed fields unknown; do not demand a URL to recreate it.
For direct clients, get the MCP URL from the existing connection or T3's
[sharing setup](https://github.com/pingdotgg/t3code/blob/main/docs/user/outside-agents.md).
For a hosted client, use the reachable T3 Connect HTTPS address. Loopback
points to the client's own machine. Use a URL intended for that client.

Preserve the authenticated-owner rule, authorized projects, provider/model
selection, and maximum runtime mode. Discover missing facts before asking the
user. Ask for a consequential policy choice only when no existing authorization
settles it; installation does not authorize widening access.

## 2. Connect and authenticate MCP

Inspect the current tool schemas or CLI help before changing configuration. If
`t3` already exists, merge the required changes instead of replacing its settings.
On connector hosts, test the existing connection before attempting OAuth setup.
Do not request raw OAuth scopes through a host exposing only a managed connector.
Do not expand its grant; record the caller mode, supported modes, and any explicit
ceiling reported by `orchestrator_capabilities`. Record an unexposed ceiling as
unknown, and inspect the worker's actual runtime mode separately.

For a new direct connection, register the verified URL with streamable HTTP and
OAuth using the host's supported setup flow. Request only the permissions needed for
the authorized workflow; the read/operate scope names used here are
`orchestration:read` and `orchestration:operate`. The T3 approval screen determines
the actual grant and execution ceiling; preserve the intended existing level.
The scope flags and OpenClaw commands below apply to direct MCP clients, not
connector hosts that already manage authentication.

For the standalone-thread route used by external OpenClaw agents, expose these
tool names through any host filter, using the current server's actual catalog:

```text
t3_project_list, t3_project_read, orchestrator_capabilities,
t3_thread_launch, t3_thread_list, t3_thread_read, t3_thread_wait,
t3_thread_send, t3_pending_request, t3_thread_interrupt
```

If the host uses actual delegated child tasks instead, discover and expose its
supported `delegate_task`/`task_status`/`task_cancel` route. Do not grant unrelated
administrative tools just to make discovery easier. Host restrictions take
precedence over this suggested list; report a required capability that cannot be
exposed rather than bypassing the restriction.

OpenClaw command families are documented in the
[MCP CLI guide](https://docs.openclaw.ai/cli/mcp). Inspect the installed
version's help before making changes.

```bash
openclaw --profile PROFILE mcp add --help
openclaw --profile PROFILE mcp configure --help
openclaw --profile PROFILE mcp tools --help
openclaw --profile PROFILE mcp login t3
openclaw --profile PROFILE mcp reload
```

Use the discovered profile or its verified wrapper, or omit the profile
flag for the default installation. `mcp add` accepts `--url`, `--transport`, `--auth`,
`--oauth-scope`, and `--include`; inspect existing state and help to build the
specific command. `mcp configure` updates an existing entry; `mcp tools` updates
filters. Preserve existing filter entries unless removal is requested.

Skip these direct-client OAuth steps for a working managed connector. Only when
authentication actually needs setup or renewal, complete OAuth using the
supported browser/CLI flow. Keep credentials out of
chat, prompts, Git, and diagnostic output; use the host's protected secret flow
when user input is required. If OpenClaw explicitly requires offline state
ownership, stop only the selected gateway through its verified service owner,
run login, and restore that gateway even if login fails. Do not work around the
ownership gate by editing its credential database or copying another profile's
tokens. Coordinate a conflicting active task before taking it offline.

Apply configuration changes through the target host's supported publication
or reload path. OpenClaw's CLI `mcp reload` refreshes runtimes within that
CLI process. Verify tool discovery and a read-only call such as project
listing from the target agent. A configuration
status saying `authorized` does not prove the credentials are still usable.

Before selecting a smoke-test task, call `orchestrator_capabilities` and page
through `t3_project_list`. Match the intended repository using verified
`repositoryIdentity`/origin remote metadata, normalizing equivalent SSH/HTTPS
Git URLs; never select by display name alone. Record the matched remote and
`projectId`. A successful capability call proves access to that call, not that
launch permissions or a particular repository are available.

If the intended repo has no T3 project, do not create one without authorization.
For a general installation smoke test, use another existing authorized project
and record both repositories, the substitution, and the missing project as an
open item. If acceptance requires the specific missing repo, that acceptance
stays blocked even if the alternate smoke test passes. Never substitute repos
for an actual user coding task.

## 3. Install the complete skill

Install this directory with its `references/` files through the host's supported
loader. For a selected OpenClaw profile, install the local skill directory as follows.

```bash
openclaw --profile PROFILE skills install /absolute/path/to/t3-delegate --agent AGENT_ID
openclaw --profile PROFILE skills info t3-delegate --agent AGENT_ID --json
```

For updates, compare the installed copy with its source and preserve separate
local edits. Use `--force` only after establishing that replacing the installed
copy will not discard unrelated work. Read back the installed entry and check
that it is eligible and model-visible. On Rakazo, prefer the native skill tool
when available; use GitHub/library import only when that is the supported route
for this installation. Read existing entries before updating them.

### Hosts with a single skill body

Some creation tools store only `content`/`body` even when the host's imported
skills can have companion files. Inspect the actual create/update/read schemas.
When companion-file writes are unavailable, install the main `SKILL.md` body
with Appendix A containing this `install.md` and Appendix B containing `brief.md`.
Rewrite the main body's relative links to those appendix headings; do not leave
instructions pointing to absent files. Copy each reference once, without
recursively embedding the combined body into Appendix A.

Prefer all source files at one immutable commit. Record that commit and the Git
blob SHA for each original file in the receipt. For an explicitly supplied local
draft or a source with only file bytes available, record `commit: null`, identify
it as a byte snapshot, and record per-file SHA-256 digests. Never attribute dirty
working-tree bytes to HEAD or call a SHA-256 digest a Git blob SHA. Also record
the installed combined-body digest. Check the live host's size limit and
whether it counts characters or bytes before writing. If the installed host
does not expose a limit, use its documented validation and verify readback.
If the body does not fit, report the
limit and use a supported companion-file/import route; do not silently truncate.
Read back the complete entry, paging if necessary, and verify both appendices
and their contents survived. Missing references fail installation acceptance.

## 4. Install the always-loaded routing rule

Merge this block near the start of the host's `AGENTS.md`, system instructions,
or equivalent bot instructions. Replace the skill reference with its real
installed path/name. Preserve unrelated instructions and existing access/model
policies. Amend conflicting generic advice about optional subagents so it does
not exempt small repository tasks from the required execution route.

```text
For actionable repository coding, testing, reviews, and investigations, load
t3-delegate before substantive repository tools and execute the work through
T3 MCP. This includes small and read-only tasks without a T3 mention. Other
engineering skills belong in the worker brief and do not replace routing.
Honor an explicit local/no-T3 request; file names, read-only scope, or brevity
are not opt-outs. If already running inside T3, execute there without recursive
delegation. If T3 is unavailable, report the blocker instead of doing the work
locally. Follow the skill to set the origin/repository title, wait for the result,
handle questions and user takeover, and verify completion before reporting it.
Copy the caller's task-relevant prohibitions and approval requirements into every
worker brief; the worker's execution mode does not grant additional authority.
```

Check the instructions actually injected into a new session. On OpenClaw,
inspect `systemPromptReport.bootstrapTruncation`; if needed, adjust only the
selected profile's bootstrap limit so its instructions fit within the total
budget. Check the deployed version's configuration reference for the applicable
bootstrap limit. A file on
disk is not evidence that the model received it in full.

On hosts without an injected-prompt report, read back the persisted bot
instructions using the supported tool, including every page. Keep routing status
`pending_first_fresh_session` until a fresh repository task demonstrably submits
to T3 before local implementation inspection. That trajectory is behavioral
evidence of routing, not proof of the exact hidden system prompt. Do not claim
prompt injection was directly observed when the host cannot expose it.

### Worker authority on every task

The worker does not automatically receive the originating bot's private
instructions or human-approval rules, regardless of `full-access` or another
runtime mode. Every brief must explicitly carry the applicable allowed actions,
actions requiring approval, and prohibitions. For example, if the caller forbids
production, DNS, repository-visibility changes, pushes, or deletions without
approval, restate those restrictions and whether the required approval exists.
These are examples to copy when applicable, not universal new bans or permission
to push. A requested PR still needs the caller's actual publication authority.

Use the worker's supported approval controls where real approval is required;
text in a brief is not an enforced access boundary. Never select a broader mode
to bypass the caller's gate. If mandatory approval cannot be enforced by this
route, do not launch that action; report the missing approval mechanism. Keep
secrets and unrelated private instructions out of the brief.

## 5. Prove the installation works

Start a fresh target-agent session with a small, read-only investigation of an
authorized repository. Do not mention T3, delegation, the skill, or the title
format in the test task. Request a bounded answer and forbid repository edits,
tests/installers, PRs, and external messages. Use the host's result surface, not
LINE/Slack/email, unless delivery there was explicitly requested.

Inspect the actual trajectory and remote thread. Acceptance requires:

- The agent loads routing instructions and submits the work before inspecting
  implementation files or running substantive repository commands locally.
- The submitted and actual title follow
  `[platform/agent-id] owner/repo: Short task description`, with trusted identity.
- T3 completes, and the originating agent uses the supported wait/read flow to
  retrieve the final result and return the real thread link.
- Independently verify at least one concrete worker claim against the authorized
  source (a raw file at a recorded commit, an API response, or local source).
  Record the claim, source revision/location, and observed result. A worker's
  self-report is not acceptance. Verify that no forbidden changes or outbound
  messages occurred, and assess instruction loading using the host's available
  evidence rather than claiming hidden prompt visibility.

For thread links, prefer the `link` returned by launch/wait/read, whether an app
HTTPS URL or a supported clickable deep link. If launch lacks one, read the
existing thread to recover its link. Store the raw thread ID and returned link;
store MCP session/environment routing identifiers separately only if explicitly
returned. Do not confuse those identifiers or fabricate a URL from a raw
`mcp:launch:...` ID. If no supported clickable link can be obtained, record the
limitation and leave that acceptance item incomplete. Use a documented host link
builder only when its required origin/routing metadata is actually available.

A standalone `t3_thread_wait` timeout leaves the same job running. Read that
thread and any pending question/approval, then issue another bounded wait when
the tool permits it. Never launch a replacement merely because a wait timed out.
Follow any `next` instruction to yield for completion instead; the delegated-task
event/wake route must not be polled. Keep questions, permission requests, and
terminal failures distinct from a timeout.

Repeat a representative second task if the first exposed a routing issue. If
short tasks finish in one originating turn, report that evidence precisely; it
does not prove that completion can wake a closed turn/session.

Record every lifecycle row below as `passed`, `failed`, `unsupported`, or
`not_run`, with evidence or a concrete limitation. A happy-path result alone sets
`coreStatus: core_ready`, scoped to `smokeTest.repository`; keep
`installationStatus: partial`. Use `installationStatus: fully_verified` only
when all applicable lifecycle rows and any required target-repository acceptance
pass. Channel delivery is applicable only when requested. Unsupported or
untested required behavior leaves the installation partial. Use bounded, harmless
fixtures and the host's permitted test surfaces. Do not send external messages,
change production, or incur extra permissions to manufacture a pass.

| Capability | Required observation |
| --- | --- |
| Automatic routing/title/result | Fresh task with no T3 hint submits before local inspection; actual title matches; result is read and independently checked. |
| Question round trip | Worker asks a harmless structured question; caller lists/reads it with `t3_pending_request`, relays it when needed, responds with the actual schema, and resumes the same job. |
| Follow-up/steering | An authorized update uses `t3_thread_send` on the same thread, and readback shows it was applied; respect rules requiring a new delegation for a new child review. |
| User takeover/handback | Explicit takeover stops automated mutations and answers; no cancellation merely for takeover; handback reads current state before coordination resumes. |
| Explicit interruption | An authorized disposable top-level run is stopped with `t3_thread_interrupt`, or a delegate with `task_cancel`; readback confirms termination and no automatic relaunch. |
| Wait timeout/no duplicate | Timeout is followed by read/wait or the tool's completion wake on the same identifiers; no duplicate launch occurs. |
| Work across turns | Originating turn ends; a real supported event wakes it or another documented continuation resumes the saved receipt, reads completion, and reports through the requested surface. An operator manually checking status is not an automatic wake. |
| Channel delivery, if requested | Actual delivery receipt on the authorized channel; no pass inferred from task completion or a queued message. |

If a check fails, report the exact failed step and repair the narrow cause before
claiming the corresponding acceptance complete. Return the installed
agent/profile, source revision, instruction destination, connection/auth result,
test thread links, and lifecycle coverage using the receipt below.

## 6. Receipt

Keep this in the host's existing task/install store, outside the target coding
repo. Fill observed values; keep unexposed values `null` instead of inferring
them. Add one verification/lifecycle entry per check. Never store credentials,
full private prompts, or an OAuth callback code here.

```json
{
  "installationStatus": "partial",
  "coreStatus": "not_run",
  "host": {"platform": null, "agentId": null, "instructionDestination": null},
  "source": {"repository": null, "revisionKind": null, "commit": null, "blobs": {}},
  "installedSkill": {"reference": null, "layout": null, "digestAlgorithm": null, "digest": null, "bodyLimit": {"value": null, "unit": null}},
  "connection": {"reference": null, "reused": null, "authChanged": null, "capabilityEvidence": null, "reportedRuntimeMode": null, "explicitCeiling": null},
  "target": {"requestedRepository": null, "projectStatus": null},
  "smokeTest": {"repository": null, "matchedRemote": null, "projectId": null, "threadId": null, "runId": null, "link": null, "mcpSessionId": null, "environmentId": null, "provider": null, "requestedModel": null, "executedModel": null, "runtimeMode": null},
  "routingStatus": "pending_first_fresh_session",
  "verification": [{"claim": null, "source": null, "revision": null, "observed": null, "status": "not_run"}],
  "lifecycle": [{"capability": "across_turns", "status": "not_run", "evidence": null}],
  "rollback": {"instructionsBackup": null, "previousSkillSource": null, "connectionChanges": null},
  "open_notes": []
}
```

## 7. Rollback

Before changes, preserve the prior instructions through the host's supported
version/history or a protected local backup. Record their version/reference,
not the entire private body in the shared receipt. Preserve the previous skill
source commit and blob SHAs (or its existing user-edited version) so it can be
restored exactly. For a combined-body installation, keep its prior digest and
supported skill version as well.

Record only the connection settings changed by this installation. If an active
connector was reused and credentials were untouched, explicitly record
`authChanged: false`: there is no auth rollback to perform. Do not revoke or
replace that credential during rollback. If authentication or filters actually
changed, use the host's supported restoration/revocation flow for those changes
only; never restore token values from a report or affect another agent's grant.

## Appendix B. Repository task brief

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
