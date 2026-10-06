# Spec: Pluggable, self-upgrading agent

Status: ready for review. Synthesized from the design session; no issue tracker was connected, so this has not been published or labeled.

## Problem Statement

I want one agent that can take on any task I give it, but "any task" cannot be achieved by building features one at a time. Today, agents either cover a narrow job, or they are broad but monolithic: adding a capability means changing the agent itself, and a mistake in one capability can affect everything else.

I also want the agent to improve itself by writing its own extensions. That is only acceptable if it can never loosen its own safety rules, if every self-written change is checked by something it does not control, and if I stay in charge of the decisions that carry real risk. I do not want to approve every small action by hand, because that makes the agent tiresome to use, but I also do not want an agent that approves its own permissions.

## Solution

A small, fixed, trusted core plus pluggable everything else.

The core owns the agent loop, the plugin contracts, the permission gate, the session state, and the budgets. Only a human can change the core. Everything else is a plugin: tools, event sources, model providers, memory and compaction strategies, output sinks, and hooks. Tools and event sources run out of process, and anything the agent writes itself is treated as untrusted.

The agent runs as a daemon driven by a durable event queue. Each event starts an isolated session with its own context, budget, and permissions. A one-shot command-line mode is the same system, enqueueing one event and waiting for its session.

Every action passes through a deny-by-default permission gate based on capability manifests and three risk tiers. Reversible actions proceed and are logged. Irreversible or external actions need my approval. An optional delegate plugin can approve actions on my behalf, but only inside a rule envelope I author.

The agent can extend itself by changing prompts, configuration, memory, and plugins. It can never change the loop, the plugin contracts, or the permission layer. Each self-written change must pass static checks, contract tests, a behavioral evaluation suite that I own, and a canary run, and then it needs my approval before it is deployed as a versioned release with automatic rollback.

## User Stories

1. As the owner, I want every capability to be a plugin, so that I can add or remove abilities without touching the core.
2. As the owner, I want the core to be small and fixed, so that I can review and trust the part that enforces safety.
3. As the owner, I want only a human to be able to change the loop, the plugin contracts, and the permission layer, so that the agent cannot loosen its own rules.
4. As the owner, I want to start the agent as a long-running service, so that it can react to events without my starting it each time.
5. As the owner, I want to run a single task from the command line, so that I can use the agent without running a service.
6. As the owner, I want the command-line mode to use the same machinery as the service, so that behavior does not differ between the two.
7. As a plugin author, I want event-source plugins to only enqueue events, so that a faulty source cannot interfere with running sessions.
8. As the owner, I want events to be stored durably before processing, so that a crash never loses an event.
9. As the owner, I want a failed session to be resumable from its log, so that work is not repeated or lost.
10. As the owner, I want each event to run in its own isolated session, so that one runaway task cannot corrupt another.
11. As the owner, I want a low cap on concurrent sessions, so that cost and risk stay predictable while I build trust.
12. As the owner, I want to raise the concurrency cap deliberately, so that I decide when the agent is ready to do more at once.
13. As the owner, I want global limits on tokens, money, steps, and wall time, so that the agent can never run away.
14. As the owner, I want a session that hits a limit to write a handoff note instead of dying silently, so that the work can be resumed.
15. As the owner, I want the core, not the model, to decide when a session ends, so that a confused model cannot loop forever.
16. As the owner, I want to interrupt any running session, so that I can always stop the agent.
17. As the owner, I want long sessions to compact their context automatically at a safe threshold, so that they never fail at the context limit.
18. As the owner, I want the goal, constraints, granted permissions, open work, and key decisions to survive compaction, so that the agent does not forget what it is doing.
19. As the owner, I want bulky tool outputs compacted first, with pointers to the full record, so that detail can be re-fetched when needed.
20. As the owner, I want every compaction recorded as an event, so that I can replay what the agent actually saw.
21. As a plugin author, I want compaction strategies to be swappable, so that I can improve them without changing the core.
22. As the owner, I want every tool call to pass through one permission gate, so that no plugin can bypass the rules.
23. As the owner, I want the gate to deny anything not explicitly granted, so that new plugins start with no power.
24. As a plugin author, I want to declare the capabilities a plugin needs in a manifest, so that the host can grant exactly that and nothing more.
25. As the owner, I want actions classified as read-only, reversible, or irreversible, so that approval effort matches risk.
26. As the owner, I want read-only actions within granted scope to proceed automatically, so that the agent is not slowed by trivial approvals.
27. As the owner, I want reversible actions to proceed automatically but be logged and undoable, so that mistakes are cheap to fix.
28. As the owner, I want irreversible or external actions to require my approval, so that I stay in control of anything that cannot be undone.
29. As the owner, I want to approve each irreversible action individually by default, so that nothing slips through.
30. As the owner, I want to optionally grant time-limited scopes for repeated irreversible actions, so that I can reduce interruptions deliberately and temporarily.
31. As the owner, I want to receive approval requests in the terminal by default and through other channels by plugin, so that I can respond where I already work.
32. As the owner, I want every action recorded in an audit log the agent cannot write to, so that I can trust the record.
33. As the owner, I want the audit log to be tamper-evident, so that I can detect any alteration.
34. As the owner, I want no plugin to ever have access to the session or audit stores, so that plugins cannot rewrite history.
35. As the owner, I want tools and event sources to run out of process, so that a crashing or malicious plugin cannot take down the agent.
36. As the owner, I want anything the agent writes itself to be treated as untrusted, so that self-written code gets no special privileges.
37. As the owner, I want tool outputs marked as untrusted data inside the loop, so that content the agent reads cannot issue it instructions.
38. As the owner, I want the agent to be able to spawn a task force agent for a narrow job, so that work and verification can be delegated.
39. As the owner, I want a child session to receive a narrower permission set than its parent, so that delegation never widens authority.
40. As the owner, I want a child session to return only a short result to its parent, so that the parent's context stays small.
41. As the owner, I want a verifier agent to start with a fresh context and see only the task spec and the artifacts, so that it does not inherit the worker's blind spots.
42. As the owner, I want the verifier to be read-only, so that it judges work instead of changing it.
43. As the owner, I want the verifier to prefer objective evidence such as tests and builds before judgment, so that "done" means something checkable.
44. As the owner, I want the verifier to answer pass, fail, or uncertain with evidence, so that doubt is surfaced instead of hidden.
45. As the owner, I want an uncertain verdict to reach me, so that unclear cases get a human decision.
46. As the owner, I want task force agents never to approve irreversible actions, capability expansions, or upgrades, so that no agent grants authority to another agent.
47. As the owner, I want an optional delegate plugin that approves actions for me, so that I am interrupted less.
48. As the owner, I want the delegate to decide only inside a rule envelope I write, so that its behavior is predictable and auditable.
49. As the owner, I want the delegate to approve, deny, or escalate, with escalate as the answer for anything outside the envelope, so that it never guesses.
50. As the owner, I want the delegate to see only structured facts about an action, so that injected content cannot reach the approver.
51. As the owner, I want the delegate to propose new rules from my past approvals but never write them, so that its learning changes the envelope only with my consent.
52. As the owner, I want to opt in to silent learning for the lowest tier only, so that I can trade some transparency for convenience where it is safe.
53. As the owner, I want every delegated decision logged with the rule that justified it, so that I can audit and revoke.
54. As the owner, I want the agent to be able to change its own prompts, configuration, and memory, so that it improves with use.
55. As the owner, I want the agent to be able to write and update plugins, so that it can extend its own abilities.
56. As the owner, I want the agent to be unable to change the loop, the plugin contracts, or the permission layer, so that self-upgrade has hard limits.
57. As the owner, I want every self-written change to pass static checks and a declared-permissions review, so that obvious problems are caught early.
58. As the owner, I want contract tests to confirm a plugin handles errors and timeouts correctly, so that a new plugin cannot destabilize the agent.
59. As the owner, I want a behavioral evaluation suite built from my real workflows, so that "better" is measured against work I care about.
60. As the owner, I want the evaluation suite to be owned by me and not editable by the agent, so that it cannot lower the bar to pass.
61. As the owner, I want the agent to be able to propose new evaluation cases that take effect only after my review, so that the suite grows with each new failure.
62. As the owner, I want any regression on the baseline suite to block an upgrade automatically, so that new capability never costs existing quality.
63. As the owner, I want a canary run in shadow mode before a new version takes over, so that real-world behavior is compared before deployment.
64. As the owner, I want upgrades that stay within already-granted capabilities to be eligible for earned auto-approval, and any capability expansion to need my review, so that routine changes flow and risky ones do not.
65. As the owner, I want every deployed plugin version to be tracked and rollback to be a single switch, so that a bad upgrade is easy to undo.
66. As the owner, I want sessions to be replayable from their logs, so that I can debug failures and evaluate new versions against real history.
67. As the owner, I want storage to sit behind a swappable interface, so that I can move beyond a single local database if my needs grow.
68. As the owner, I want state backed up simply, so that losing a machine does not lose the agent's history.
69. As the owner, I want per-session and global spending caps from the first version, so that multiplied model calls from verifiers and canaries cannot surprise me.
70. As the owner, I want the system built in an order where each stage is usable on its own, so that I get value early and add risk late.

## Implementation Decisions

**Modules**

- Core: the agent loop, session runner, plugin host, permission gate, budget enforcer, compaction scheduler, and event bus. Changed only by a human.
- Event queue and session store: one embedded SQLite database in write-ahead mode, with a single writer inside the core. Enqueueing an event and creating its session happen in one transaction.
- Audit store: a separate SQLite file owned only by the core, with each entry including a hash of the previous one.
- Plugin host: loads plugins, enforces manifests, and mediates every call across the process boundary.
- Task force and verifier: exposed to the loop as a tool that spawns a child session with a narrower permission set.
- Delegate: an optional plugin on the approval hook.
- Upgrade pipeline: sandboxed build, static checks, contract tests, evaluations, canary, human approval, versioned deploy, rollback.

**Extension points**

- Tools and event sources: out of process, speaking an MCP-style protocol.
- Model providers, memory and compaction strategies, sinks (including the approval channel), and hooks: lightweight plugins with narrower contracts.
- Storage: behind an interface, with SQLite as the first implementation.

**Trust model**

- Anything the agent writes is untrusted and runs out of process or sandboxed.
- Only a small set of hand-written, signed plugins may run in-process.
- Plugins never receive filesystem access to the session or audit stores.
- Tool outputs and external content are treated as data, never instructions.

**Permissions**

- Deny by default. Plugins declare scoped capabilities in a manifest, and the host grants only those.
- Three tiers: read-only, reversible, and irreversible or external. Only the third requires approval.
- A new plugin version that requests broader capabilities than its predecessor always requires human review.

**Runtime**

- Daemon first. The command-line mode enqueues one event and waits.
- One isolated session per event, a concurrency cap starting at two to three, and global budgets for tokens, money, steps, and wall time.
- Single-user for now. Sessions carry an owner field so that multi-user isolation can be added without redesign.

**Loop and context**

- A single loop where the model calls tools until finished. Delegation happens through the spawn tool, not through a separate planner.
- The session log is the source of truth. The context window is a view of it.
- Compaction triggers at roughly 70 to 80 percent of the window, pins the goal, constraints, permissions, open work, and key decisions, compacts tool outputs first, and keeps pointers to full records.
- Stop conditions are enforced by the core: verified completion, budget exhaustion, or human interrupt. Exhaustion writes a handoff note.

**Verification and delegation**

- The verifier has fresh context, is read-only, prefers objective evidence, and returns pass, fail, or uncertain with evidence.
- Task force agents may decide whether a task is done and whether a reversible change is kept or retried. They may never decide irreversible actions, capability expansions, or upgrades.
- The delegate decides only inside a human-authored rule envelope, sees only structured action facts, and can propose but not write rules. Silent learning is opt-in, off by default, and limited to the lowest tier.

**Self-upgrade**

- The agent may change prompts, configuration, memory, and plugins. It may not change the loop, the plugin contracts, or the permission layer.
- Pipeline order is fixed: static checks and manifest review, contract tests, behavioral evaluations, canary, human approval, versioned deploy.
- The evaluation suite is owned by the human. The agent may propose cases, which take effect only after review. Any baseline regression blocks the upgrade.

**Build order**

1. Walking skeleton: loop, session log, permission gate, one out-of-process tool plugin, command-line driver, spending caps.
2. Daemon: event queue, worker pool, concurrency cap, global budgets, one event-source plugin.
3. Compaction and sub-agents.
4. Evaluation suite and verifier.
5. Delegate and the rule-proposal flow.
6. Self-upgrade pipeline.

## Testing Decisions

**What makes a good test.** Test external behavior only: what goes in as an event or a plugin call, and what comes out in the session log, the audit log, and approval requests. Do not test internal structure or the wording of model output.

**Proposed single seam, for confirmation.** The highest seam is the session runner interface: enqueue an event, then observe the resulting session log, audit log, approval requests, and final state. Almost every behavior in this spec can be exercised there with a scripted model provider and fake plugins. Please confirm this is the seam you want, or name a different one.

**Modules to test through that seam**

- Permission gate: undeclared capabilities are denied, each tier behaves as specified, and a capability expansion triggers review.
- Budgets and stop conditions: each limit ends the session and writes a handoff note.
- Crash recovery: killing the process mid-session and restarting resumes from the log without duplicating actions.
- Isolation: a failing or hostile plugin cannot affect another session or read the stores.
- Compaction: pinned items survive, and pointers resolve to the full record.
- Verifier independence: it receives no worker reasoning and cannot modify artifacts.
- Delegate: inside-envelope approvals, outside-envelope escalation, and rejection of attempts to write rules.
- Upgrade pipeline: a regression blocks deployment, a capability expansion requires review, and rollback restores the previous version.

**Prior art.** None assumed. Evaluation cases should be drawn from your real workflows and grow every time the agent fails in a new way.

## Out of Scope

- Multi-user operation and per-user isolation beyond the owner field.
- Hosting on multiple machines or replacing SQLite.
- Any change by the agent to the loop, plugin contracts, or permission layer.
- Write access to outside systems in the first versions beyond what plugins are explicitly granted and approved.
- Personal data sources and public-facing actions such as sending messages or making purchases, until the approval flow has proven itself.
- Automatic expansion of the evaluation suite without human review.

## Further Notes

Risks to watch:

1. **Evaluation quality.** The whole self-upgrade story depends on the baseline suite. A thin suite gives false confidence.
2. **Prompt injection.** Content the agent reads can try to steer it. Structured-only approval facts and untrusted-output marking reduce this but do not remove it.
3. **Cost runaway.** Verifiers, sub-agents, and canaries multiply model calls, so spending caps must exist from the first version.

Self-upgrade is built last on purpose: it depends on a trustworthy evaluation suite, permission model, and audit log, and it is the riskiest part to get wrong.
