# Karpathy-Inspired Coding Agent Guidelines

A compact set of behavioral guidelines for coding agents, based on [Andrej Karpathy's observations](https://x.com/karpathy/status/2015883857489522876) about common LLM coding failures.

This fork keeps the original principles, removes integration-specific bloat, and adds explicit guidance for **subagent orchestration, model selection, context handoff, verification, and commits**.

## Why This Fork

The original project primarily targeted Claude Code and duplicated the same guidance across plugin metadata, Cursor rules, documentation, examples, and translations.

This fork focuses on the instructions themselves.

It removes the Claude/Cursor-specific packaging and expands the areas that matter for modern coding agents:

| Guideline                 | Purpose                                                      |
| ------------------------- | ------------------------------------------------------------ |
| **Think Before Coding**   | Avoid assumptions and surface meaningful ambiguity           |
| **Simplicity First**      | Prevent overengineering and speculative abstractions         |
| **Surgical Changes**      | Keep diffs scoped to the requested change                    |
| **Goal-Driven Execution** | Turn work into verifiable outcomes                           |
| **Subagents**             | Delegate only independent work and use the appropriate model |
| **Commits**               | Keep changes cohesive and Git operations safe                |

## Why Add Subagent Rules?

Subagents introduce a new class of failure that the original guidelines did not address.

OpenAI's current model guidance notes that models may **delegate less often than desired** unless explicitly told when and how to use subagents. GPT-5.6 supports multi-agent workflows, but good orchestration still depends on the instructions given to the parent agent.

I also noticed the opposite problem in practice: a **GPT-5.6 Sol parent spawning another GPT-5.6 Sol at high reasoning effort just to explore a codebase**.

That works, but it wastes expensive model capacity on a bounded task that a smaller model can handle.

The added rules therefore make delegation explicit:

* Keep **linear tasks** in the main agent: if `B` depends on `A`, do `A → B` instead of spawning both.
* Delegate only genuinely independent work.
* Do not assume a subagent should use the same model as its parent.
* Give every subagent the exact **goal, relevant context, constraints, expected output, and verification criteria**.
* Use the cheapest model that can reliably complete the task.
* Keep the parent responsible for reviewing and integrating subagent results.

### GPT-5.6 Model Routing

For bounded coding subtasks, this fork favors:

```text
Luna high/xhigh
    ↓ escalate if needed
Terra high/xhigh
    ↓ escalate if needed
Sol
```

**GPT-5.6 Luna** is the default for focused work such as:

* codebase exploration;
* locating definitions and call sites;
* reading logs and tests;
* documentation lookup;
* mechanical analysis;
* narrowly scoped reviews.

Luna is substantially cheaper than Sol while still supporting high reasoning effort, making it a better default for well-specified subagents.

**GPT-5.6 Terra** is an optional middle tier when Luna is insufficient but Sol is unnecessary.

**GPT-5.6 Sol** should be reserved for work that actually benefits from it:

* architecture;
* ambiguous debugging;
* difficult implementation decisions;
* cross-cutting changes;
* synthesis of conflicting findings;
* high-consequence work.

GPT-6 Astra is more naturally suited to long-horizon orchestration, but GPT-5.6 agents should not need Astra-style defaults to use subagents effectively. The instructions make the delegation strategy explicit.

## Context Matters

A cheaper subagent is only useful when it receives enough context to solve the task.

Delegation should look like:

```text
Goal: [exact outcome]

Context:
- [relevant files/modules]
- [known behavior]
- [decisions already made]

Constraints:
- [scope boundaries]
- [project rules]
- [what must not change]

Output:
- [exact result expected]

Verify:
- [how success is checked]
```

Do not make subagents rediscover information the parent already knows, and do not dump the entire parent conversation into them when only a few facts matter.

The goal is **minimum sufficient context**.

## Core Principle

The common theme is simple:

> Give coding agents a precise goal, constrain unnecessary behavior, and make success verifiable.

The guidelines intentionally bias toward correctness, minimal changes, and verification over raw speed.

For trivial tasks, use judgment.

## Upstream Changes

Compared with the original repository, this fork removes:

* Claude Code plugin marketplace metadata;
* Claude-specific project instructions;
* Cursor-specific rules and documentation;
* duplicated examples;
* the translated README.

The repository is now focused on the behavioral guidelines rather than maintaining multiple wrappers around the same content.

## License

MIT
