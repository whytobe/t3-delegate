# T3 Delegate

Send repository tasks from an external agent to T3 Code and follow progress in
the returned T3 thread. The originating agent coordinates the task and T3
runs it in the selected project and environment.

Send [t3.whyto.be](https://t3.whyto.be) to OpenClaw, Rakazo, or another
supported agent to start setup. The page links to [INSTALL.md](INSTALL.md),
which covers host-specific setup and a fresh-session verification task.

The package follows the [Agent Skills specification](https://agentskills.io/specification).
Use [SKILL.md](SKILL.md) with its `references/` directory on hosts that support
companion files, or the [self-contained version](dist/rakazo.md) on hosts that
store a single skill body. T3's [outside-agent MCP interface](https://github.com/pingdotgg/t3code/blob/main/docs/user/outside-agents.md)
is the connection between the agent and T3.

The instruction page is served as plain text from `site/` on Cloudflare Pages.
Deploy changes with `wrangler pages deploy site --project-name t3-delegate --branch main`.
