# Getting the Most from Cortex Code

Think of Cortex Code (CoCo) as a fast, knowledgeable pair-programming partner — great at execution, but you still own the design, the review, and the judgment.

## Do vs Avoid

| Do | Avoid |
|---|---|
| Use `/plan` before building anything multi-step. Let CoCo propose, then approve. | Asking CoCo to build everything in a single prompt. |
| Review every change before accepting — use the diff view. | Auto-accepting without reading what changed. |
| Keep sessions focused on code and data tasks. | Mixing unrelated topics into a coding session. |
| Start fresh sessions when things feel off — this is normal, like saving your work. | Assuming one session should last forever. |
| Write an `AGENTS.md` file with your project context so CoCo starts every session informed. | Expecting CoCo to "remember" your codebase across sessions — it's stateless by design. |
| Start with the basics — plan, execute, review, verify — before exploring advanced features. | Jumping straight to sub-agents or custom skills before the core workflow is comfortable. |

## Pre-Session Checklist

- [ ] I can state what I want in one sentence.
- [ ] I know which account, role, warehouse, and schema we'll use.
- [ ] I know what must not change — production objects, roles, cost-sensitive resources.
- [ ] I'll use `/plan` before asking CoCo to build anything complex.
- [ ] I'll review every diff and SQL statement before accepting.
- [ ] If CoCo contradicts itself or produces unexpected output, I'll save my progress and start a fresh session.


For full details, see the [Cortex Code CLI documentation](https://docs.snowflake.com/en/user-guide/cortex-code/cortex-code-cli), [CLI reference](https://docs.snowflake.com/en/user-guide/cortex-code/cli-reference), and [extensibility guide](https://docs.snowflake.com/en/user-guide/cortex-code/extensibility) and the [best practices HoL](https://www.snowflake.com/en/developers/guides/best-practices-cortex-code-cli/)



## Keep in Mind

- **CoCo is stateless.** Each session starts fresh. Use an `AGENTS.md` file and reference docs to give it the context it needs.
- **LLMs are not deterministic.** You'll get consistently correct results, but not identical output every time. This is normal for all AI coding agents.
- **Sessions have a natural lifespan.** As conversations grow, older context gets compressed. Restarting with a handoff summary keeps quality high.
- **The core loop is your foundation:** Goal → `/plan` → execute one phase → review diffs → verify → commit. Master this before exploring advanced features.
