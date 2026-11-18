# T3 Delegate

Send repository tasks from an external agent to T3 Code and follow progress in
the returned T3 thread. The originating agent coordinates the task and T3
runs it in the selected project and environment.

Tell OpenClaw, Rakazo, or another supported agent to read the
[instruction URL](https://raw.githubusercontent.com/whytobe/t3-delegate/main/INSTALL.md)
and install `t3-delegate` for the selected agent. See [INSTALL.md](INSTALL.md)
for host-specific setup and a fresh-session verification task.

The package follows the [Agent Skills specification](https://agentskills.io/specification).
Use [SKILL.md](SKILL.md) with its `references/` directory on hosts that support
companion files, or the [self-contained version](dist/rakazo.md) on hosts that
store a single skill body. T3's [outside-agent MCP interface](https://github.com/pingdotgg/t3code/blob/main/docs/user/outside-agents.md)
is the connection between the agent and T3.
