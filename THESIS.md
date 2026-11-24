# AgentLock: thesis

**In one line:** a prompt injection doesn't need new permissions, only
standing ones, so scope each task's permissions to what the user actually
asked for.

## The problem

Agents get broad standing permissions (refund any order, email anyone)
because some task might need them. An injection only has to redirect those
permissions. Static allowlists can't help, because the right scope depends on
the request in front of the agent right now.

## The bet

1. **Most tasks compile to a narrow envelope.** For the common, concrete
   requests ("refund order 4411 and tell the customer"), rules can derive the
   allowed tools and argument values, and that covers most production
   traffic.
2. **Provenance of authority is the defence.** If only the user's words can
   set or widen scope, and content the agent reads never can, injected
   instructions have nothing to work with.
3. **Ask, don't guess.** Showing ambiguous envelopes to the user to confirm
   is acceptable friction if it's rare.
4. **Enforcement at an MCP proxy** works with existing tool servers
   unchanged.

## Why now

Agents with write access to customer, payment and messaging systems are
going into production, prompt injection remains unsolved at the model level,
and MCP gives a single place to enforce.

## What success looks like

- On an AgentDojo-style injection benchmark, AgentLock blocks most attacks
  that redirect existing permissions, while completing most benign tasks
  without extra prompts.
- Confirmation prompts appear on a small minority of tasks in realistic
  workloads.
- Runs on the shared OpenAstra proxy core alongside AgentTaint.

## What would prove us wrong

- Real requests are too open-ended to compile, so envelopes are either too
  wide to help or need confirmation so often that users approve blindly.
- Agents legitimately need to widen scope mid-task so often that the flow is
  unusable.
- Model-assisted compilation is required for most tasks, reintroducing the
  injection risk the design is meant to avoid.

## Not a bet on

Detecting injections in content, or replacing organisation-wide policy
(that's OpenGPL, which sets ceilings).

## Where it fits

Area VIII, AI Runtime Security. Pairs with
[AgentTaint](https://github.com/sadayamuthu/agenttaint) (which data drives a
call) and AgentID (who the envelope is issued to).
