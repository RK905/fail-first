---
name: fail-first
license: MIT
description: >-
  Test-driven development for coding agents: write a test, watch it fail for
  the right reason, then write the minimum code to pass. Use when implementing
  any feature, bug fix, refactor, or behavior change, before writing
  implementation code.
---

# Fail First

**Rule: no production code without a test that failed first.** If you never saw the test fail, you don't know it can catch anything.

Applies to features, bug fixes, refactors, and behavior changes. Skip only for throwaway prototypes, generated code, or config files, and only when the user agrees.

Wrote code before the test? Delete it and start again from the test. Don't keep it "for reference": you will adapt it, and that is testing after.

## The cycle

Repeat for each behavior, one at a time:

1. **Red: write one failing test.** One behavior, a name that says what should happen, real code (mock only what is slow or external).
2. **Verify red: run it and read the failure.** It must *fail* (not error out), with the message you expected, because the feature is missing (not because of a typo).
   - Passes already? You're testing existing behavior. Change the test.
   - Errors? Fix the error and rerun until it fails correctly.
3. **Green: write the simplest code that passes.** No extra options, no "while I'm here" refactors, nothing the test doesn't demand.
4. **Verify green: run the whole suite**, not just your file (`npm test`, `pytest`, `cargo test`, whatever the repo uses). All tests pass, output clean (no new errors or warnings).
   - Your test fails? Fix the code, not the test.
   - Another test fails? Fix it now. If it was already broken, report it by name; never let a red test scroll past unmentioned.
5. **Refactor, only while green.** Remove duplication, improve names, extract helpers. Rerun. Add no behavior.

Bug fix = same cycle: first write a test that reproduces the bug and fails, then fix.

```ts
// Bug: empty email accepted
test('rejects empty email', async () => {
  expect((await submitForm({ email: '' })).error).toBe('Email required');
});
// red:   expected 'Email required', got undefined
// green: if (!data.email?.trim()) return { error: 'Email required' };
```

## Writing tests that can actually fail

Before writing the test body, **name the production change that would make it fail.** If you can't, the test proves nothing; test an observable behavior instead.

- **Hand-derive expected values.** Use literals or hand-checked fixtures. An expectation computed with the code under test (or its helpers) always passes.
  `expect(buildQuery({ tag: 'urgent' })).toBe('tag:"urgent"')`, not `toBe(buildQuery({ tag: 'urgent' }))`.
- **Test behavior, not decisions.** Not `expect(MAX_RETRIES).toBe(5)` but "a failing call is retried 5 times and never a 6th." Don't assert exact source text; run the script or code and check outputs, side effects, exit codes.
- **Test your code, not the framework.** Assert the contract at your boundary (the route you register, the query you emit, the payload you produce). Trivial getters, constructors, and forwarding need no test of their own.
- **One behavior per test.** "and" in the name means split it.

### Mocks

- **Never assert on a mock.** If the assertion would fail just because the mock was removed, it tests the mock. Assert the real behavior or unmock.
- **Mock at the right level.** Learn the real method's side effects first; mock only the slow or external part below what your test depends on.
- **Make mocks faithful and specific.** Mirror the full real data shape, not only the fields you read. Assert arguments, counts, or order when they're part of the contract. Give success, error, and malformed cases their own fixtures.
- **Keep test-only code out of production.** A cleanup method only tests call belongs in test utilities.
- **Mock setup bigger than the test?** Switch to an integration test with real components.

### Mutation check (before you finish)

Mentally break the code. At least one test should fail for each:

- wrong constant or argument
- wrong branch taken
- missing state change or side effect
- empty or default return
- missing validation for zero, empty, null, unauthorized, or malformed input

A mutation nothing catches is unprotected behavior.

## Stop signs

If you catch yourself thinking any of these, stop and restart the cycle:

| Thought | Reality |
|---|---|
| "Too simple to test" | Simple code breaks too. The test takes 30 seconds. |
| "I'll add tests after" | Tests written after pass immediately and prove nothing. They check what you built, not what you should have built. |
| "I already tested it manually" | Manual checks aren't recorded, repeatable, or complete. |
| "Deleting this work is wasteful" | Sunk cost. Untrusted code is the real waste. |
| "I need to explore first" | Fine: explore, throw it away, then start with a test. |
| "This is hard to test" | Hard to test means hard to use. Simplify the design or inject dependencies. |
| "This case is different" | It isn't. Ask the user if you truly want an exception. |

## Done means

- [ ] Every new function or behavior has a test
- [ ] You saw each test fail, for the expected reason, before implementing
- [ ] Code is the minimum needed to pass
- [ ] Full suite passes with clean output; any pre-existing failures are reported by name
- [ ] Edge cases and error paths covered; mutation check passes
- [ ] Mocks only where unavoidable, never asserted on

Can't tick every box? You skipped a step. Go back.
