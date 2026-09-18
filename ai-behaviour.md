# AI Behaviour Scripting

## Behaviour trees vs FSMs

Reforger uses behaviour trees, not finite state machines. Trees execute strict
left-to-right and therefore cannot fall into the infinite loops FSMs are prone
to. Logic is built from **sequence**, **selector** and **parallel** nodes.

Every node returns exactly one of three states: **success**, **fail**, or
**running**.

## Scripted nodes

- Custom task nodes inherit from **`AITaskScripted`** and override `OnInit`,
  `OnEnter`, `OnTaskSimulate`, `OnAbort`.
- Decorators — which monitor conditions continuously — inherit from
  **`DecoratorScripted`** or `DecoratorTest`.

**Performance rule:** heavy logic (spatial loot searches, wide queries) must be
split across frames by returning **running** from `OnTaskSimulate` rather than
computing the whole loop on one tick. Blocking the main thread here is the usual
cause of AI-related stutter.

**Debugging:** prefer script editor breakpoints over behaviour tree breakpoints
when chasing node failures — BT breakpoints obscure what actually returned.

## Agents

A character entity carries an **`AIControlComponent`**, which accesses an
**`AgentTemplate`** to spawn the invisible **AI Agent** that serves as the
entity's brain. Brain and body are separate objects and must be cross-referenced:

- `GetControlledEntity()` on the AI Agent → the body
- `GetControllingAgent()` on the `AIControlComponent` → the brain

## Waypoints and messages

- AI groups are driven by **Waypoints** (Timed, Entity, and others), which spawn
  **Activities**.
- Group Activities dispatch **AI Messages** to individual agents to coordinate
  tasks — moving, looting, engaging.

## Utility and reactions

- Agents process incoming AI Messages through their **`UtilityComponent`**.
- `PerformReaction` accepts or discards each message, then plans Behaviours by
  **priority number**.
- External commands can append large modifiers — **+1000 or +2000** — which
  forces the agent to override its base survival instincts. This is how Game
  Master orders beat self-preservation.

## Debug

Launch Workbench with `-define AIDebug` to expose the AI behaviour debug panel
during live testing.
