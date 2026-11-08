# Coordination channels

How work moves between agents in this factory, and where to look when it stops moving.

## Channel inventory

One row per channel this factory actually uses. Delete the ones you do not, and add the ones you built yourself.

| Channel | Primitive | Persists a restart | Timing | Addressed to | Owner |
| --- | --- | --- | --- | --- | --- |
| Tasks | `bd`, `gc sling` | yes | next eligible pull | a pool | |
| Mail | `gc mail send` | yes | recipient's next turn | one alias, or `--all` | |
| Orders | `gc order` | yes (definition on disk) | schedule or predicate | a pool, or the controller | |
| Nudges | `gc session nudge` | no | immediate, if the session is alive | one session | |
| Session attach | `gc session attach` | no | immediate | one session | |

Fill in the owner column. A channel with no owner is one nobody notices has broken.

## Preferences by handoff

One row per handoff in your pipeline. The primary is what carries it normally; the fallback is what you reach for when the primary has clearly not fired.

| Handoff | Primary | Fallback | Why |
| --- | --- | --- | --- |
| Human → front gate | Tasks (`gc sling` a bead I filed) | Mail the mayor | That is how I ran my first work item; The mayor is for when I don't want to write the bead myself |
| Front gate → implementer | Tasks (sling to the polecat pool) | Nudge a live polecat to check its hook | A sling persists, and wakes the pool; a nudge only helps the session along |
| Implementer → reviewer | Tasks (bead reassigned to refinery) | Mail the mayor | If the push or handoff fails, someone needs to hear about it |
| Reviewer → implementer (bounce) | Tasks (bead reopened with rejection_reason) | Nudge | Bounces are routine so they should not flood a human's inbox |
| Reviewer → architecture review | Tasks (routed to an architect) | Nudge the architect | Same pool routing as every other stage |
| Architecture review → reviewer | Tasks (architect_approved set, back to refinery) | Mail mayor | Architect can't reach a verdict => escalate |
| Reviewer → human (merge) | Tasks (PR opened, bead closed) | Mail mayor | I see the PR on GitHub; mail only if its heold back |
| Any agent → human (blocked) | Mail Mayor | Attach | A blocked status nobody looks at is invisible; mail isn't |
| Nothing → factory (scheduled sweeps) | Orders | `gc order run` by hand | factory-pulse / bead-closed-log write FACTORY_LOG.md |

## Protocols

Anything an agent has to do beyond picking the right channel. Two examples to replace with your own:

- **Blocked work always mails a human, never only sets a status.** A blocked bead is invisible unless somebody is looking at the board; a mail is not.
- **A nudge is never the only signal.** If a nudge is the mechanism, something durable carries the same intent, because a nudge to a dead session is gone with no trace.

## When the factory goes quiet

The diagnostic order that follows from the table above. Adjust it to match your own primaries.

1. `bd list --status open` — is there work, and does it have an assignee?
2. `gc session list` — is anything alive that could pick it up?
3. `gc mail inbox <alias>` — is a message sitting unread?
4. `gc order check` — should something have fired, and did it?
5. `gc session peek <id>` — is an agent alive but stuck mid-turn?
