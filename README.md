# CS3383 AI Practical Test Cheatsheet

Preparation handbook for Java, JUnit 5, specification-based testing, and an
**OpenCode CLI-only** examination workflow.

> This repository is preparation material, not an examination project. During
> an examination, the official instructions always override this cheatsheet.

## Examination AI Usage Rules

### Non-negotiable examination rules

- Use **OpenCode CLI text conversations only** for all AI-assisted work.
- Use only the examination-authorized model. The mock specifies
  `deepseek-v4-flash`; do not assume the real examination uses the same model.
- The same model restriction applies to every subagent.
- Do not use ChatGPT, Claude, Gemini, GitHub Copilot, Codex, another AI client,
  or an external AI API, script, extension, or background process.
- Start OpenCode from the actual examination project directory. Keep
  implementation and testing inside the authorized environment.
- Preserve every native OpenCode session and subagent session. Never delete,
  edit, fabricate, import, hide, or selectively omit conversation history.
- Put every instruction, debugging request, code-change request, and
  test-generation request in the native OpenCode text conversation so the work
  remains traceable.
- Modify only files explicitly permitted by the examination specification.
- Do not change dependencies, build settings, `pom.xml`, supplied tests, or
  production code unless the task explicitly permits it.
- Use only the official submission workflow.

These restrictions apply to the examination. Preparation resources do not
grant permission to install a Skill, access GitHub, change configuration, or
run a downloaded script during the examination.

## OpenCode CLI Quick Start

1. Open the examination project in the approved environment.
2. Start `opencode` from the examination project directory.
3. Use the available model selector (normally `/models`) and confirm the exact
   authorized model shown by the examination environment.
4. Paste the [Master OpenCode Examination Prompt](#master-opencode-examination-prompt).
5. Ask OpenCode to read the complete specification and list allowed and
   forbidden files before editing anything.
6. Ask OpenCode to generate tests or repair Java only as required by each task.
7. Ask OpenCode to run only permitted tests and explain each failure.
8. Ask OpenCode to verify the final diff, test results, and remaining risks.
9. Submit through the prescribed OpenCode/examination workflow.

### 30-second checklist

- [ ] Correct project directory and authorized model
- [ ] Specification read completely
- [ ] Allowed files listed explicitly
- [ ] Test-only versus repair task identified
- [ ] Tests target requirements, boundaries, and state transitions
- [ ] Permitted tests run; failures investigated, not hidden
- [ ] Final diff contains only authorized files
- [ ] Native sessions preserved; official submission completed

## Master OpenCode Examination Prompt

Copy this entire prompt into **OpenCode CLI**:

```text
You are the authorized AI coding agent running inside OpenCode CLI for this examination.

Before changing anything:
1. Read the entire examination specification and all task-specific instructions.
2. Inspect the project structure and relevant existing code/tests using read-only actions.
3. State the tasks, acceptance criteria, explicitly allowed files, forbidden changes, test restrictions, build commands, and submission requirements.
4. Confirm that this session and every subagent use only the examination-authorized model. Do not call any external AI service.
5. Treat the real examination rules as authoritative if they differ from any mock, preparation note, prompt, or Skill.
6. Do not install dependencies or Skills, change build configuration, access an unavailable external service, or run downloaded scripts unless the specification explicitly permits it.

Then give a brief requirement-to-test/implementation plan. Perform the required work rather than only giving advice. Modify only explicitly permitted files. For test-only tasks, never edit production code. For repair tasks, make the smallest specification-correct fix and do not weaken tests to make them pass.

Run only permitted verification commands. Analyze failures by distinguishing incorrect expectations, implementation defects, compilation errors, and environment problems. At the end, report: files changed, requirements covered, exact commands run and actual results, unresolved issues, and any unverified assumptions. Preserve all native OpenCode and subagent session history; never alter, hide, export-import, fabricate, or delete it.
```

## Table of Contents

- [Examination AI Usage Rules](#examination-ai-usage-rules)
- [OpenCode CLI Quick Start](#opencode-cli-quick-start)
- [Master OpenCode Examination Prompt](#master-opencode-examination-prompt)
- [Examination Rules and Mock Overview](#examination-rules-and-mock-overview)
- [Java and JUnit 5 Cheatsheet](#java-and-junit-5-cheatsheet)
- [Test Design Strategies](#test-design-strategies)
- [Test Coverage and JaCoCo](#test-coverage-and-jacoco)
- [Mutation Testing](#mutation-testing)
- [AI Agent Skills and GitHub Resources](#ai-agent-skills-and-github-resources)
- [Reusable OpenCode Prompts](#reusable-opencode-prompts)
- [Ultra-Short OpenCode Prompts](#ultra-short-opencode-prompts)
- [OpenCode Command Reference](#opencode-command-reference)
- [Troubleshooting](#troubleshooting)
- [Common Mistakes and Final Checklist](#common-mistakes-and-final-checklist)
- [References](#references)

## Examination Rules and Mock Overview

### Mock task map

| Task | Required work | Production code | High-value behaviors |
|---|---|---|---|
| `RoomSchedule` | JUnit tests and Java repair | Repair is required/allowed | overlap versus adjacency, duplicate names, move/cancel, failure preserves state |
| `AnnouncementFormatter` | JUnit tests | Must not be modified | trimming, numbering, duplicates, blanks, exact formatting, repeated calls |
| `MiniTable` | JUnit tests | Must not be modified | set/clear/overwrite, sums, copy overlap, snapshots, independence |

The mock describes representative tasks only. Method names, grading weights,
file permissions, and test restrictions may differ in the real examination.

### Mock-specific restrictions versus general advice

| Mock-specific rule (verify again in the real exam) | General testing advice |
|---|---|
| Only the supplied model, including subagents | Read the specification before writing tests |
| Preserve native OpenCode session history | Trace every test to a requirement or fault class |
| Only ordinary JUnit 5 `@Test` methods; no parameterized, nested, or dynamic tests | Keep tests independent and deterministic |
| No private-field, source-text, filesystem, network, or grader-internal access from tests | Assert observable public behavior and state |
| Do not change build settings/dependencies | Use the project's existing build and conventions |
| Preserve supplied tests and submit through the named process | Review the final diff and actual test output |

Priority order:

> **Specification correctness > fault detection > state verification > coverage percentage > test quantity**

## Java and JUnit 5 Cheatsheet

Use the imports and APIs already available in the project. The examples assume
JUnit Jupiter (`org.junit.jupiter.api`). Replace the illustrative class/method
names with the real public API.

### Core assertions

```java
import static org.junit.jupiter.api.Assertions.*;
import org.junit.jupiter.api.Test;

class CalculatorTest {
    @Test
    void add_twoPositiveIntegers_returnsSum() {
        Calculator calculator = new Calculator();
        int actual = calculator.add(2, 3);
        assertEquals(5, actual);
    }

    @Test
    void isEmpty_newInstance_returnsTrue() {
        Calculator calculator = new Calculator();
        assertTrue(calculator.isEmpty());
        assertFalse(calculator.hasResult());
    }

    @Test
    void find_unknownKey_returnsNull() {
        Calculator calculator = new Calculator();
        assertNull(calculator.find("missing"));
        assertNotNull(calculator.getHistory());
    }

    @Test
    void divide_byZero_throwsAndPreservesState() {
        Calculator calculator = new Calculator();
        calculator.add(2, 3);
        int before = calculator.getLastResult();

        ArithmeticException error = assertThrows(
                ArithmeticException.class,
                () -> calculator.divide(5, 0));

        assertEquals(before, calculator.getLastResult());
        assertNotNull(error.getMessage()); // Assert exact text only if specified.
    }
}
```

### Grouped state checks (`assertAll`, only when permitted)

```java
@Test
void move_validRequest_updatesAllObservableState() {
    Booking booking = new Booking("A", 9, 10);
    booking.moveTo(11, 12);

    assertAll(
            () -> assertEquals("A", booking.getName()),
            () -> assertEquals(11, booking.getStart()),
            () -> assertEquals(12, booking.getEnd()));
}
```

### Floating point comparisons

```java
@Test
void average_threeValues_returnsExpectedValue() {
    assertEquals(2.0 / 3.0, Statistics.average(0, 1, 1), 1.0e-9);
}
```

Never compare computed `double` values without a suitable tolerance unless the
specification guarantees exact representation.

### Before/after state and independent instances

```java
@Test
void rejectedUpdate_doesNotPartiallyChangeState() {
    Schedule schedule = new Schedule();
    schedule.book("A", 9, 10);
    String before = schedule.toString();

    assertThrows(IllegalArgumentException.class,
            () -> schedule.book("B", 9, 10));

    assertEquals(before, schedule.toString());
}

@Test
void twoInstances_doNotShareMutableState() {
    MiniTable first = new MiniTable(2, 2);
    MiniTable second = new MiniTable(2, 2);
    first.set(0, 0, 7);

    assertEquals(7, first.get(0, 0));
    assertEquals(0, second.get(0, 0));
}
```

### Frequent Java mistakes

- Compare strings with `expected.equals(actual)` or `assertEquals`, never `==`.
- Check both sides of integer boundaries (`min - 1`, `min`, `min + 1`) when
  representable; watch overflow at `Integer.MIN_VALUE`/`MAX_VALUE`.
- Do not reuse mutable fixtures across tests unless reset reliably.
- Do not assume lists, arrays, copies, or snapshots are independent—mutate one
  side and assert the other remains unchanged.
- Test state after a rejected operation; exceptions can still leave mutations.
- Match the specified exception type; a broad type may hide the wrong path.
- Do not assert unspecified ordering, formatting, or error text.

## Test Design Strategies

### Compact strategy guide

| Strategy | Java example | Fault it can expose |
|---|---|---|
| Equivalence partitioning | valid time, invalid negative time, invalid reversed interval | one validation branch missing |
| Boundary value analysis | capacity `0`, `1`, maximum; adjacent intervals `[9,10)` and `[10,11)` | `<` used instead of `<=` |
| State transition testing | empty → booked → moved → cancelled | wrong transition or stale state |
| Decision table testing | overlap? duplicate name? valid interval? → accept/reject | combined-condition error |
| Negative testing | unknown ID, null (if specified), out-of-range index | missing rejection or wrong exception |
| Edge cases | empty input, one element, repeated call, self-copy, overlapping copy | special-case defect |
| Regression testing | reproduce the precise input that once failed | bug reintroduced later |
| Specification-based testing | derive expected output only from the contract | tests merely copy current bugs |
| Mutation testing | replace `>` with `>=`, delete assignment, return constant | weak/missing assertion |

### Requirement-to-test matrix template

| ID | Requirement | Partition/boundary/state | Test name | Expected observable result | File allowed? | Status |
|---|---|---|---|---|---|---|
| R1 | _quote/paraphrase contract_ | valid + lower boundary | `method_case_result` | return/state/exception | yes | planned |
| R2 | _failure must preserve state_ | rejected transition | `method_invalid_stateUnchanged` | exception + before=after | yes | planned |

Build this matrix before coding. A small matrix exposes missing requirements and
redundant tests more reliably than counting test methods.

### Fault detection without redundant tests

1. Give every test a distinct reason to fail.
2. Select inputs that separate likely faulty implementations—for example,
   adjacency distinguishes `end <= nextStart` from `end < nextStart`.
3. Assert the complete relevant result, not just “no exception”.
4. For mutations, assert the intended change and unrelated preserved state.
5. Add sequence tests only when order/history matters; otherwise isolate tests.
6. Remove cases that add no boundary, transition, assertion, or fault class.

### Small decision table example

| Existing overlap | Duplicate name | Interval valid | Expected |
|---:|---:|---:|---|
| No | No | Yes | Accept |
| Yes | No | Yes | Reject; state unchanged |
| No | Yes | Yes | Reject; state unchanged |
| No | No | No | Reject; state unchanged |

## Test Coverage and JaCoCo

- **Line coverage**: whether each executable line ran.
- **Branch coverage**: whether each decision outcome ran.
- **Coverage is not correctness**: a test can execute a line without asserting
  its effect. 100% coverage can miss wrong constants, boundary operators, state
  corruption, and interaction faults.
- **JaCoCo** records covered classes, lines, branches, and methods. Use its HTML
  report to find untested paths, then map each path to the specification.

### Maven workflow—only with existing project configuration

```text
mvn test
mvn verify
```

Use `mvn verify` when the existing `pom.xml` binds JaCoCo report generation to a
lifecycle phase. A report is commonly at `target/site/jacoco/index.html`.
`mvn jacoco:report` works only when the required plugin is already available.
Do **not** add/change a plugin, dependency, threshold, or `pom.xml` during an
examination unless explicitly authorized.

### VS Code workflow

If the approved environment already provides Java Test Runner and coverage
support, run the relevant class/package from Test Explorer and use its coverage
view to locate unexecuted paths. Highlights are diagnostic, not proof of good
assertions. Do not install extensions during the exam unless allowed.

### Reading a report

1. Start with missed branches in requirement-heavy methods.
2. Map each missed branch to a specified behavior.
3. Add a test only when it expresses a meaningful contract case.
4. Inspect covered lines for weak assertions; execution is insufficient.
5. Re-run the smallest relevant scope, then the permitted full suite.

## Mutation Testing

A **mutant** is a deliberately changed production version, such as replacing
`<` with `<=`, negating a condition, removing a call, or changing a return.

- **Killed mutant**: at least one test fails against the changed code.
- **Surviving mutant**: all tests still pass; the input/assertion may be weak,
  or the mutant may be equivalent to original behavior.
- **Mutation score**: commonly `killed / total non-equivalent mutants × 100%`.

PIT/PITest is useful in Java preparation because hidden faulty implementations
are conceptually similar to mutants. For a survivor ask: “Which specified,
observable behavior distinguishes this change?” Avoid internal assertions.

PIT commonly requires build configuration and dependencies. Use it only in
preparation or when supplied and authorized. Never modify an exam `pom.xml`
merely to add PIT. Never inspect, probe, reverse-engineer, or access the hidden
grader.

## AI Agent Skills and GitHub Resources

These links were checked as public resources. A native Skill is a directory
whose entry file is `SKILL.md`; documentation and prompts are not automatically
Skills.

| Repository | Specific Skill | Purpose | Relevance | Link |
|---|---|---|---|---|
| `github/awesome-copilot` | `java-junit` | JUnit, debugging, JaCoCo, optional PIT, Maven/Gradle guardrails | Broad Java/JUnit reference; some features may be forbidden in the mock | [SKILL.md](https://github.com/github/awesome-copilot/blob/main/skills/java-junit/SKILL.md) |
| `github/awesome-copilot` | `test-gap-audit` | Read-only search for missing or weak tests | Useful for gaps; verify exam tool rules | [SKILL.md](https://github.com/github/awesome-copilot/blob/main/skills/test-gap-audit/SKILL.md) |
| `mavka-ai/unit-tests-skills` | `generate-test-cases` | Analyze code and list test cases | Good requirement/test planning workflow | [SKILL.md](https://github.com/mavka-ai/unit-tests-skills/blob/main/skills/generate-test-cases/SKILL.md) |
| `mavka-ai/unit-tests-skills` | `generate-tests` | Generate and run Java JUnit tests | Useful preparation; defaults can conflict with an exam | [SKILL.md](https://github.com/mavka-ai/unit-tests-skills/blob/main/skills/generate-tests/SKILL.md) |
| `multica-ai/andrej-karpathy-skills` | `karpathy-guidelines` | Minimal, surgical, assumption-aware coding | General guardrail, not Java-specific | [SKILL.md](https://github.com/multica-ai/andrej-karpathy-skills/blob/main/skills/karpathy-guidelines/SKILL.md) |
| `robsonoliveiradacosta/agent-skills` | `add-mutation-testing` | PIT mutation workflow | Quarkus/Maven and may edit `pom.xml`; preparation only | [SKILL.md](https://github.com/robsonoliveiradacosta/agent-skills/blob/main/skills/add-mutation-testing/SKILL.md) |
| `robsonoliveiradacosta/agent-skills` | `flaky-test-triage` | Diagnose shared data, clocks, async, ports, races | Debugging reference; likely excessive for small exam units | [SKILL.md](https://github.com/robsonoliveiradacosta/agent-skills/blob/main/skills/flaky-test-triage/SKILL.md) |
| `robsonoliveiradacosta/agent-skills` | `spec-create` | Structure behavior and acceptance criteria | Specification-first reference; do not create extra exam files | [SKILL.md](https://github.com/robsonoliveiradacosta/agent-skills/blob/main/skills/spec-create/SKILL.md) |

### Skills in OpenCode

Current OpenCode documentation describes project Skills under
`.opencode/skills/<skill-id>/SKILL.md` and compatibility discovery under
`.agents/skills` and `.claude/skills`. A discovered Skill may be loaded by
mentioning `@skill-id`. Discovery/loading is different from installation:

- **View/load**: use an already available, permitted Skill.
- **Install/copy**: writes files and may need network/package tools.
- **Execute helpers**: may run bundled scripts with project permissions.

During an exam, first ask whether Skills are already available and permitted.
Do not install, copy, update, or execute a Skill because this README links to
it. Read `SKILL.md` before trusting it; community instructions can conflict with
exam rules (for example, changing production code or `pom.xml`).

## Reusable OpenCode Prompts

Every prompt is for direct use in **OpenCode CLI**. Replace placeholders and let
the actual specification override the prompt.

### 1. Master examination prompt

Use [Master OpenCode Examination Prompt](#master-opencode-examination-prompt).

### 2. JUnit test generation

```text
Inside this OpenCode session, read the full task specification, relevant production API, supplied tests, and neighboring test conventions before editing. First list the permitted test file(s) and build a compact requirement-to-test matrix. Then add independent ordinary JUnit 5 @Test methods only, unless the specification permits other forms. Prioritize boundary conditions, equivalence partitions, state transitions, repeated calls, failure-state preservation, copy independence, and inputs that distinguish likely faulty implementations. Assert observable specified behavior; do not inspect private fields, source text, files, networks, or grader internals. Do not modify production code, pom.xml, dependencies, or any unauthorized file. Run the permitted tests and report the exact command and actual result.
```

### 3. Java debug and repair

```text
Use only this OpenCode session and the authorized model. Read the specification and failing-test output, reproduce the failure with a permitted command, and distinguish a wrong expectation from an implementation, compilation, or environment defect. Confirm that production repair is explicitly allowed and name the permitted Java file before editing. Find the root cause, make the smallest specification-correct change, preserve unrelated behavior, and never weaken/delete tests to hide a defect. Rerun the focused test and then the permitted suite; report actual results and remaining risks.
```

### 4. Coverage improvement

```text
Inspect only the coverage report already available in this project and map missed lines/branches to the written specification and existing tests. Propose a short ranked list of meaningful missing behaviors, then add tests only in authorized files. Do not chase 100% mechanically, add assertion-free tests, install tools, or modify pom.xml. Run permitted tests and report coverage/test results actually observed; mark anything unavailable.
```

### 5. Mutation testing improvement

```text
Analyze the provided surviving-mutant report or suspected fault categories without accessing or reverse-engineering hidden grader internals. For each relevant survivor, identify the specification behavior, input, and observable assertion that would distinguish it. Strengthen only authorized test files, avoid implementation-coupled assertions and redundant cases, and do not install PIT or change build configuration unless explicitly permitted. Run permitted tests and report actual results.
```

### 6. Strict file modification

```text
Before editing, quote the specification's allowed-file list and show the exact files you intend to change. Modify only those files; do not change production code in a test-only task, supplied tests, pom.xml, dependencies, configuration, or session history. Stop and report the conflict if required work cannot be completed within that scope.
```

### 7. Final verification

```text
Perform final verification inside OpenCode. Re-read the requirements, inspect current diff/status, confirm every changed file is authorized, and run only permitted focused/full tests. Check that no test was weakened, disabled, or made order-dependent and that rejected operations preserve state where specified. Report requirements covered, changed files, exact commands with actual pass/fail counts, unresolved issues, and unverified assumptions. Do not alter or delete native session history; follow the official submission procedure.
```

### 8. Unexpected test failure

```text
Analyze this failure before editing. Read the specification, exact failing assertion/stack trace, relevant test, and implementation. Classify the cause as (a) incorrect expected result, (b) implementation defect, (c) compilation/build problem, or (d) environment issue, and cite evidence. Change only files permitted for this task; never change an assertion merely to match buggy behavior. Reproduce and rerun the smallest relevant test, then report actual results.
```

## Ultra-Short OpenCode Prompts

### Analyze this specification

```text
Read the entire specification in this OpenCode project. List tasks, acceptance criteria, allowed/forbidden files, test restrictions, commands, and ambiguities before editing.
```

### Generate JUnit tests

```text
Generate high-fault-detection JUnit 5 tests from the specification in authorized test files only. Cover boundaries and state changes, then run permitted tests and report actual results.
```

### Improve test quality

```text
Review existing tests for weak, redundant, or missing assertions against the specification. Strengthen only authorized test files and verify the suite.
```

### Find missing edge cases

```text
Map the specification to existing tests and list untested boundaries, invalid inputs, state transitions, repeated calls, and independence cases. Rank them by likely fault detection.
```

### Debug this failure

```text
Reproduce this failure and classify it as expectation, implementation, compilation, or environment related. Cite evidence before making any permitted edit.
```

### Repair only this Java file

```text
Repair only `<allowed-file.java>` with the smallest specification-correct change. Do not alter tests or other files; rerun permitted tests and report actual results.
```

### Review coverage gaps

```text
Map available JaCoCo gaps to specified behaviors and propose meaningful tests. Do not change build configuration or optimize the percentage blindly.
```

### Check modified files

```text
Inspect current diff/status and compare every changed path with the allowed-file list. Report unauthorized or suspicious changes without hiding them.
```

### Verify before submission

```text
Recheck requirements, authorized diff, permitted test results, remaining failures, and submission instructions. Preserve all OpenCode session history and report evidence, not assumptions.
```

## OpenCode Command Reference

OpenCode changes over time and examination installations may be pinned. Run
`/help` (or open the command palette) in the actual environment before relying
on a command. Do not update OpenCode during an exam unless instructed.

| Requested name | Current practical meaning | Examination-safe note |
|---|---|---|
| `/init` | Guided creation/update of `AGENTS.md` in documented versions | Writes a file; do not run when only task files may be modified |
| `/models` | Lists/selects models for the current session | Confirm the authorized model; do not select another model |
| `/compact` | Compacts/summarizes current session context | Alias may be `/summarize`; it does not authorize deleting history |
| `/diff` | Opens the current diff viewer in current CLI docs | Ensure only authorized files changed |
| `/mcps` | Current documented in-session MCP manager | Do not connect external services unless explicitly allowed |
| `/mcp` | Not the current documented in-session spelling; `opencode mcp ...` is a shell CLI family | Prefer `/help`; do not configure MCP opportunistically |
| `/agents` | Version-dependent; agents are documented via config and `opencode agent ...` | Confirm availability and model inheritance before a subagent |
| `/skills` | Version-dependent; current docs emphasize discovery and `@skill-id` | Loading is not installation; do not write Skill files unless allowed |
| `/review` | Often a project/custom prompt, not guaranteed built-in | If absent, request a review in plain text |
| `/status` | Not guaranteed in the TUI; `opencode service status` reports server health | Do not confuse server health with git/task status |

Useful documented commands include `/help`, `/sessions`, and `/new`. Avoid
`/undo` during an examination unless explicitly allowed: documented versions
can remove recent messages and revert file changes, conflicting with preserved
conversation history.

### Plain-text alternative

```text
Inspect the current project diff and status. List every modified file and verify it against the specification's allowed-file list. Do not modify anything.
```

## Troubleshooting

### Compilation failure

1. Read the first compiler error, not only the Maven summary.
2. Check imports, signatures, package, Java/JUnit version, and test location
   against existing code.
3. Do not install a missing library; use only project dependencies.
4. Ask OpenCode for the smallest authorized correction and rerun the focused
   compile/test command.

### Test fails unexpectedly

1. Re-read the exact specification clause.
2. Minimize the scenario and inspect actual versus expected values.
3. Check fixture leakage, order, shared static state, and repeated calls.
4. Decide whether the expectation or implementation is wrong before editing.
5. A production bug in a test-only task is not permission to repair it.

### Maven/JaCoCo/PIT unavailable

- Use only commands and plugins already supplied.
- Do not add dependencies or edit `pom.xml` without explicit permission.
- Report the tool/report as unavailable; continue with specification-based
  analysis and permitted tests.

### OpenCode command or Skill unavailable

- Check `/help` and the installed OpenCode version.
- Use a plain-text prompt instead of a convenience command.
- Do not fetch/install a replacement unless explicitly allowed.
- If a prepared Skill conflicts with the exam, follow the exam.

## Common Mistakes and Final Checklist

### Common failures

- Editing files outside the allowed scope.
- Repairing production code in a test-only task.
- Using forbidden parameterized/nested/dynamic tests, private-field access,
  filesystem/network access, or implementation introspection.
- Writing tests that execute code but do not assert required results.
- Repeating one fault class while missing boundaries and transitions.
- Ignoring adjacency, duplicates, invalid ranges, repeated calls, failure-state
  preservation, snapshots, overlap, or independence.
- Treating 100% coverage as proof of correctness.
- Changing correct expectations, deleting tests, or disabling tests for green.
- Installing dependencies, Skills, plugins, or extensions; changing `pom.xml`.
- Using a non-authorized model for a subagent or another AI interface/API.
- Altering, deleting, fabricating, or selectively omitting session history.
- Submitting without checking the final diff and official procedure.

### Final verification checklist

- [ ] Actual specification overrides this cheatsheet
- [ ] Authorized model confirmed for main agent and subagents
- [ ] All AI work exists in native OpenCode text sessions
- [ ] Every requirement has a meaningful test or implementation item
- [ ] Tests assert observable behavior and relevant post-state
- [ ] No supplied test was weakened, deleted, or disabled
- [ ] Exact permitted commands were run and actual results recorded
- [ ] Diff/status contains only explicitly allowed files
- [ ] No unauthorized dependency, configuration, or `pom.xml` change
- [ ] Limitations and environment failures reported honestly
- [ ] Session history preserved and official submission workflow followed

## References

- [OpenCode documentation](https://opencode.ai/docs)
- [OpenCode models](https://opencode.ai/docs/models)
- [OpenCode Skills](https://opencode.ai/docs/skills)
- [OpenCode commands](https://opencode.ai/docs/commands)
- [OpenCode CLI commands](https://opencode.ai/docs/cli/commands)
- [OpenCode MCP servers](https://opencode.ai/docs/mcp-servers)
- [JUnit 5.14.4 User Guide](https://docs.junit.org/5.14.4/user-guide/)
- [JaCoCo documentation](https://www.jacoco.org/jacoco/trunk/doc/)
- [PIT mutation testing](https://pitest.org/)
- [Awesome Copilot Skills index](https://github.com/github/awesome-copilot/blob/main/docs/README.skills.md)

> Last source check: 2026-11-12. OpenCode command names and Skill compatibility
> can change; verify against the examination's installed version.
