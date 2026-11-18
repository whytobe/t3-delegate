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
