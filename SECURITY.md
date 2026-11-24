# Security Policy

## Reporting a vulnerability

Please don't report security problems in a public issue, pull request or
discussion. Use
[**Report a vulnerability**](https://github.com/sadayamuthu/agentlock/security/advisories/new)
on the repository's Security tab instead. Only the maintainers can see the
report.

Include what you can of:

- the AgentLock version (`agentlock version`) or commit, your Go version and OS
- how AgentLock was used, and its configuration with secrets removed
- steps to reproduce
- what happened and what you expected

You'll get a reply when the report has been read, and again when a fix is
ready. A fix ships as a patch release; the advisory names you unless you'd
rather it didn't.

## Supported versions

AgentLock is at the design stage and has no releases yet. Once it does,
fixes land in the latest release only, until 1.0.

## What counts

AgentLock sits between an agent and its MCP servers and allows, blocks or holds each tool call against the current task's envelope.

AgentLock is a security tool, so anything that breaks one of these promises
is a security issue:

- **Deny by default.** A call outside the envelope that is allowed is in scope.
- **Only the user widens an envelope.** A way for tool output or retrieved content to widen or replace an envelope is in scope.
- **Fail closed.** When an envelope can't be compiled or checked, the call is held, not allowed. A path that fails open is in scope.

These are by design, and not vulnerabilities:

- An envelope the user approves allows what it says.
- AgentLock limits which calls run, not what data they carry; that's AgentTaint.

A wrong result that isn't covered above is a bug, not a vulnerability. Report
it in a normal issue; see [CONTRIBUTING.md](CONTRIBUTING.md).
