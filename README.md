# CS3383 AI Practical Test Cheatsheet

Preparation handbook for Java, JUnit 5, specification-based testing, and
OpenCode CLI. This cheatsheet is for **pre-quiz preparation and reference**.
During the quiz, follow the official instructions and use this README only as
read-only reference material; do not edit it or treat its prompts as a
replacement for entering task instructions in OpenCode.

> This repository is preparation material, not an examination project. During
> an examination, the official instructions always override this cheatsheet.

## Examination Rules

Follow the official quiz specification first. Use only the authorized OpenCode CLI/model and permitted project files; keep work in its native English text conversation and submit through the official workflow.

## Quick Start

1. Open the quiz project and start OpenCode there.
2. Confirm the authorized model, then use the [Master Prompt](#master-opencode-examination-prompt).
3. Read the task before editing; verify allowed files and permitted test commands.
4. Review the diff and actual test results before submission.

## Master OpenCode Examination Prompt

Copy this entire prompt into **OpenCode CLI**:

The prompt, all follow-up conversation, code, and code comments must remain in
English.

```text
You are the authorized AI coding agent running inside OpenCode CLI for this examination.

Before changing anything:
1. Read the entire examination specification and all task-specific instructions.
2. Inspect the project structure and relevant existing code/tests using read-only actions.
3. State the tasks, acceptance criteria, explicitly allowed files, forbidden changes, test restrictions, build commands, and submission requirements.
4. Confirm that this session and every subagent use only the examination-authorized model. Do not call any external AI service.
5. Treat the real examination rules as authoritative if they differ from any mock, preparation note, prompt, or Skill.
6. Do not install dependencies or Skills, change build configuration, access an unavailable external service, or run downloaded scripts unless the specification explicitly permits it.
7. Conduct all conversation with the student in English. Write prompts, generated code, tests, and code comments in English.

Then give a brief requirement-to-test/implementation plan. Before each edit, inspect Git status and the current diff; after each edit, run `git status` and `git diff` to verify the exact change and confirm only explicitly permitted files changed. Use Git to track each completed change and create a commit/checkpoint only when the examination rules permit it. Perform the required work rather than only giving advice. Modify only explicitly permitted files. For test-only tasks, never edit production code. For repair tasks, make the smallest specification-correct fix and do not weaken tests to make them pass.

Run only permitted verification commands. Analyze failures by distinguishing incorrect expectations, implementation defects, compilation errors, and environment problems. At the end, report: files changed, requirements covered, exact commands run and actual results, unresolved issues, and any unverified assumptions. Preserve all native OpenCode and subagent session history; never alter, hide, export-import, fabricate, or delete it.
```

## Table of Contents

- [Examination Rules](#examination-rules)
- [Quick Start](#quick-start)
- [Master OpenCode Examination Prompt](#master-opencode-examination-prompt)
- [Mock Task Overview](#mock-task-overview)
- [Java and JUnit 5 Cheatsheet](#java-and-junit-5-cheatsheet)
- [Test Design Strategies](#test-design-strategies)
- [Test Coverage and JaCoCo](#test-coverage-and-jacoco)
- [Mutation Testing](#mutation-testing)
- [AI Agent Skills and GitHub Resources](#ai-agent-skills-and-github-resources)
- [Reusable OpenCode Prompts](#reusable-opencode-prompts)
- [Ultra-Short OpenCode Prompts](#ultra-short-opencode-prompts)
- [OpenCode Commands](#opencode-commands)
- [Quick Reminders](#quick-reminders)
- [References](#references)
## Mock Task Overview

| Task | Main focus | Important cases |
|---|---|---|
| `RoomSchedule` | Tests and permitted repair | overlap/adjacency, duplicates, move/cancel, failure preserves state |
| `AnnouncementFormatter` | Tests only | trimming, numbering, duplicates, blanks, exact formatting |
| `MiniTable` | Tests only | set/clear, sums, copy overlap, snapshots, independence |

Real quiz instructions may differ; verify file permissions and test restrictions in the actual specification.


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


## OpenCode Commands

Use `/help` in the quiz environment because commands vary by version. Common references: `/models` selects the authorized model, `/diff` reviews changes, and `/sessions` lists sessions; avoid commands that alter files or session history unless permitted.

## Quick Reminders

Use existing project tools and configuration only. If a command or report is unavailable, record that limitation; never change unauthorized files or weaken tests to get a passing result.

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
