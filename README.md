# fail-first

An agent skill that makes your AI coding agent do real test-driven development: write a test, **watch it fail for the right reason**, then write the minimum code to pass.

Works with Claude Code and any agent harness that loads `SKILL.md` skills.

## Why fail first?

Agents love to write the code, then write tests that pass on the first run. Those tests prove nothing: they check what was built, not what should have been built, and many of them would pass even if the code were broken. This skill gives the agent a short, strict loop:

- no production code until a test has failed for the expected reason
- the simplest code that passes, nothing extra
- the whole suite run before "done", with pre-existing failures reported by name
- tests that can actually catch bugs: hand-derived expected values, no assertions on mocks, a mutation check at the end

## How it works

```
  feature / bug / refactor
          │
          ▼
  1. Red ──────────── one test, one behavior, real code
          │
          ▼
  2. Verify red ───── run it: fails (not errors), expected message,
          │           because the feature is missing
          ▼
  3. Green ────────── simplest code that passes
          │
          ▼
  4. Verify green ─── full suite passes, clean output
          │
          ▼
  5. Refactor ─────── only while green, no new behavior
          │
          └──► next behavior (back to 1)

  Before done: mutation check + "done means" checklist
```

Along the way it enforces a few test-quality rules:

| Trap | What the skill does |
|---|---|
| Expected value computed by the code under test | Uses literals or hand-checked fixtures |
| Test only checks that a mock was called / rendered | Asserts real behavior, or unmocks |
| Test pins a constant or exact text | Tests the behavior that depends on it |
| Mock setup bigger than the test | Switches to an integration test |
| Code written before the test | Deletes it and starts from the test |

## Install

**Claude Code (personal, all projects):**

```bash
git clone https://github.com/RK905/fail-first /tmp/ff
cp -r /tmp/ff/skills/fail-first ~/.claude/skills/
```

**One project only:** copy `skills/fail-first` into `<project>/.claude/skills/`.

**Other harnesses:** point your agent at `skills/fail-first/SKILL.md` (Codex, Cursor, Gemini CLI, etc. can load it as an instruction file or rule).

## Usage

The skill triggers on its own when you ask the agent to implement a feature, fix a bug, or change behavior. You can also call it directly:

```
/fail-first add retry with 3 attempts to fetchInvoice()
```

Give it what you have; it asks for the rest:

| Input | What it is | Example |
|---|---|---|
| `task` | Behavior to add or bug to fix | `reject empty email on signup` |
| `test command` | How the repo runs tests (if not obvious) | `pnpm vitest run` |
| `where` | Files or module involved | `src/forms/signup.ts` |
| `constraints` | Anything that can't be real in tests | `Stripe API must be mocked` |

### Example

```
/fail-first
  task: signup form accepts an empty email
  test command: npm test
  where: src/forms/
```

The agent writes the test, runs it, and shows the failure before touching the code:

```
RED    rejects empty email
       expected 'Email required', got undefined
GREEN  if (!data.email?.trim()) return { error: 'Email required' };
SUITE  npm test → 212 passed, 0 failed
```

## Tips

- **Tell it your test command.** It runs the whole suite, so it needs to know how.
- **Ask to see the red output.** No failing run shown means the step was skipped.
- **Let it delete code.** Code written before the test gets thrown away by design; that's the point.
- **Push back on mocks.** If a test is mostly mock setup, ask for an integration test.
- **Pair with [root-cause-first](https://github.com/RK905/root-cause-first)** for bugs whose cause isn't obvious yet.

## Credits

Derived from `test-driven-development` in [obra/superpowers](https://github.com/obra/superpowers) by Jesse Vincent, merged from two files into one and made self-contained. See [NOTICE](NOTICE).

## License

[MIT](LICENSE)
