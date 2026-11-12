# CS3383 AI Practical Test Cheatsheet（繁體中文版）

本文件是考前準備和參考資料，涵蓋 Java、JUnit 5、依規格設計測試及
OpenCode CLI 工作流程。**Quiz 期間只可觀看本 README，不得編輯或把其中
prompt 當作取代在 OpenCode 內輸入考試指令的方式。**

> 本倉庫是準備／參考資料，不是考試專案。Quiz 中以正式考試指示為準。

## 考試 AI 使用規則

### 必須遵守

- 所有 AI 輔助工作只可透過 **OpenCode CLI 文字對話**進行。
- 與 OpenCode 及獲准的 DeepSeek 模型對話時，**全程只用英文**，包括初始
  prompt、追問、除錯要求、程式修改要求和測試要求。
- 只使用考試指定的 AI 模型。Mock 指定 `deepseek-v4-flash`；正式考試可能
  不同，必須以當日指示為準。
- Subagent 也必須使用指定模型。
- 不可使用 ChatGPT、Claude、Gemini、GitHub Copilot、Codex 或其他 AI 介面；
  亦不可透過 API、腳本、擴充功能或背景程序呼叫其他 AI 服務。
- 從實際考試專案目錄啟動 OpenCode。所有實作和測試都留在獲准環境內。
- 保留所有原生 OpenCode 及 subagent session。不可刪除、修改、偽造、匯入、
  隱藏或選擇性省略對話紀錄。
- 所有 AI 指示、除錯、程式修改和測試生成要求都必須輸入 OpenCode 原生文字
  對話，保持可追溯。
- 只修改考試規格明確允許的檔案。
- 除非題目明確允許，不得更改依賴、建置設定、`pom.xml`、提供的測試或
  production code。
- 只使用指定的正式提交流程。

本 README 在 quiz 前準備，quiz 中只能作唯讀參考。準備資源不代表考試期間
可以安裝 Skill、連接 GitHub、改設定或執行下載腳本。

## OpenCode CLI 快速開始

1. Quiz 前閱讀本 README，練習 Java/JUnit 範例。
2. Quiz 中在核准環境開啟考試專案。
3. 在考試專案目錄啟動 `opencode`。
4. 使用可用的模型選擇器（通常是 `/models`），確認當日指定模型。
5. 將英文版 [Master OpenCode Examination Prompt](#master-opencode-examination-prompt)
   貼入 OpenCode。
6. 用英文要求 OpenCode 完整閱讀規格，並在修改前列出允許和禁止修改的檔案。
7. 用英文要求 OpenCode 按各題要求生成測試，或在獲准時修復 Java。
8. 用英文要求 OpenCode 執行允許的測試並分析失敗。
9. 用英文要求 OpenCode 檢查最後 diff、測試結果和未解決問題。
10. 按指定 OpenCode／考試流程提交。

### 30 秒檢查表

- [ ] 專案目錄正確，模型符合考試指定
- [ ] 已完整閱讀規格
- [ ] 已明確列出允許修改的檔案
- [ ] 已分清只寫測試、只改實作或兩者皆需
- [ ] 測試涵蓋需求、邊界及狀態轉換
- [ ] 已執行允許的測試，並調查失敗
- [ ] 最終 diff 只包含獲准檔案
- [ ] 保留原生 session，按正式流程提交

## Master OpenCode Examination Prompt

以下 prompt、後續對話、程式碼及程式註解都必須保持英文。請將此 prompt
貼入 **OpenCode CLI**：

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

Then give a brief requirement-to-test/implementation plan. Perform the required work rather than only giving advice. Modify only explicitly permitted files. For test-only tasks, never edit production code. For repair tasks, make the smallest specification-correct fix and do not weaken tests to make them pass.

Run only permitted verification commands. Analyze failures by distinguishing incorrect expectations, implementation defects, compilation errors, and environment problems. At the end, report: files changed, requirements covered, exact commands run and actual results, unresolved issues, and any unverified assumptions. Preserve all native OpenCode and subagent session history; never alter, hide, export-import, fabricate, or delete it.
```

## 目錄

- [考試 AI 使用規則](#考試-ai-使用規則)
- [OpenCode CLI 快速開始](#opencode-cli-快速開始)
- [Master OpenCode Examination Prompt](#master-opencode-examination-prompt)
- [考試規則與 Mock 概覽](#考試規則與-mock-概覽)
- [Java 及 JUnit 5 速查](#java-及-junit-5-速查)
- [測試設計策略](#測試設計策略)
- [測試覆蓋率及 JaCoCo](#測試覆蓋率及-jacoco)
- [Mutation Testing](#mutation-testing)
- [AI Agent Skills 及 GitHub 資源](#ai-agent-skills-及-github-資源)
- [可重用 OpenCode Prompts](#可重用-opencode-prompts)
- [超短 OpenCode Prompts](#超短-opencode-prompts)
- [OpenCode 指令參考](#opencode-指令參考)
- [疑難排解](#疑難排解)
- [常見錯誤及最後檢查](#常見錯誤及最後檢查)
- [參考資料](#參考資料)

## 考試規則與 Mock 概覽

### Mock 題目

| 題目 | 要求工作 | Production code | 高價值測試行為 |
|---|---|---|---|
| `RoomSchedule` | 編寫 JUnit 測試及修復 Java | 題目允許／要求修復 | 重疊與相鄰時段、重複名稱、移動／取消、失敗後狀態不變 |
| `AnnouncementFormatter` | 編寫 JUnit 測試 | 不可修改 | 去空白、編號、重複／空白公告、輸出格式、重複呼叫 |
| `MiniTable` | 編寫 JUnit 測試 | 不可修改 | 設定／清除／覆寫、列總和、複製重疊、快照、資料獨立 |

Mock 只代表練習題；正式考試的方法名稱、分數比重、檔案權限和測試限制都
可能不同。

### Mock 專屬規則與一般測試建議

| Mock 專屬規則（正式考試需重新確認） | 一般測試建議 |
|---|---|
| 只用指定模型，包括 subagent | 寫測試前先讀完整規格 |
| 保留原生 OpenCode session 紀錄 | 每項測試對應一項需求或故障類型 |
| 只用一般 JUnit 5 `@Test`；不可用 parameterized、nested、dynamic tests | 測試應獨立且結果穩定 |
| 測試不可讀 private fields、原始碼文字、檔案、網絡或 grader 內部 | 驗證公開可觀察行為和狀態 |
| 不可改建置設定或依賴 | 沿用專案現有建置及慣例 |
| 保留提供的測試，按指定流程提交 | 檢查最後 diff 及實際測試輸出 |

優先次序：

> **規格正確性 > 故障偵測 > 狀態驗證 > 覆蓋率百分比 > 測試數量**

## Java 及 JUnit 5 速查

範例使用 JUnit Jupiter（`org.junit.jupiter.api`）。請按專案現有 API、依賴及
Java/JUnit 版本調整；不要自行新增依賴。

### 常用 assertions

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

### 群組狀態檢查（只在題目允許時用 `assertAll`）

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

### 浮點數比較

```java
@Test
void average_threeValues_returnsExpectedValue() {
    assertEquals(2.0 / 3.0, Statistics.average(0, 1, 1), 1.0e-9);
}
```

除非規格保證數值可精確表示，否則比較計算所得的 `double` 時應使用合適誤差值。

### 檢查操作前後狀態及物件獨立性

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

### 常見 Java 錯誤

- 字串用 `expected.equals(actual)` 或 `assertEquals` 比較，不要用 `==`。
- 檢查整數邊界兩側（可表示時測 `min - 1`、`min`、`min + 1`），留意
  `Integer.MIN_VALUE`／`MAX_VALUE` 的溢位。
- 不要在測試間共用可變 fixture，除非每次都可靠重設。
- 不要假設 list、array、copy 或 snapshot 互相獨立；修改一邊，再檢查另一邊。
- 操作被拒絕後仍要檢查狀態；拋出 exception 前可能已部分修改資料。
- 驗證規格要求的 exception 類型；過於寬泛的類型會掩蓋錯誤路徑。
- 不要斷言規格沒有訂明的排序、格式或錯誤文字。

## 測試設計策略

### 策略速查

| 策略 | Java 例子 | 可發現的故障 |
|---|---|---|
| 等價類別 | 有效時間、負時間、開始晚於結束 | 漏掉某個驗證分支 |
| 邊界值分析 | 容量 `0`、`1`、上限；相鄰區間 `[9,10)`、`[10,11)` | 把 `<` 寫成 `<=` |
| 狀態轉換 | 空白 → 預約 → 移動 → 取消 | 狀態轉換錯誤或資料殘留 |
| 決策表 | 是否重疊？名稱是否重複？時段是否有效？ | 條件組合判斷錯誤 |
| 負向測試 | 未知 ID、規格定義的 null、越界 index | 漏掉拒絕邏輯或 exception 錯誤 |
| 邊界情況 | 空輸入、單一元素、重複呼叫、自我複製、重疊複製 | 特殊情況故障 |
| 回歸測試 | 重現曾經失敗的精確輸入 | 舊錯誤再次出現 |
| 規格測試 | 只按行為契約推導預期值 | 測試複製了現有錯誤行為 |
| Mutation testing | 將 `>` 換成 `>=`、刪除賦值、回傳常數 | assertion 太弱或缺漏 |

### Requirement-to-Test Matrix 範本

| ID | Requirement | 等價類別／邊界／狀態 | Test name | 預期可觀察結果 | 檔案是否允許 | 狀態 |
|---|---|---|---|---|---|---|
| R1 | _引用或概述行為契約_ | 有效值及下邊界 | `method_case_result` | 回傳值／狀態／exception | 是 | 計劃中 |
| R2 | _失敗操作不改變狀態_ | 被拒絕的轉換 | `method_invalid_stateUnchanged` | exception 且操作前後相同 | 是 | 計劃中 |

先建立矩陣，再寫程式。矩陣能比單純增加測試數量更容易指出漏測需求和重複測試。

### 提高故障偵測並避免重複

1. 每個測試都應有明確、獨特的失敗理由。
2. 選擇能區分可能錯誤實作的輸入；例如相鄰時段可區分
   `end <= nextStart` 和 `end < nextStart`。
3. 驗證完整的相關結果，不要只檢查「沒有拋出錯誤」。
4. 測試修改操作時，同時驗證預期變更及必須保留的其他狀態。
5. 只有在順序或歷史重要時才寫流程測試，否則保持各測試獨立。
6. 若測試沒有新增邊界、狀態轉換、assertion 或故障類型，考慮刪除重複案例。

### 簡單決策表

| 與現有資料重疊 | 名稱重複 | 時段有效 | 預期 |
|---:|---:|---:|---|
| 否 | 否 | 是 | 接受 |
| 是 | 否 | 是 | 拒絕；狀態不變 |
| 否 | 是 | 是 | 拒絕；狀態不變 |
| 否 | 否 | 否 | 拒絕；狀態不變 |

## 測試覆蓋率及 JaCoCo

- **Line coverage**：程式執行過哪些可執行程式行。
- **Branch coverage**：決策的各個結果是否都執行過，例如 `if` 的 true 和
  false 路徑。
- **覆蓋率不等於正確性**：測試可以執行某行，卻沒有驗證結果。即使 line／
  branch coverage 達 100%，仍可能漏掉錯誤常數、邊界運算子、狀態破壞或互動錯誤。
- **JaCoCo** 可記錄 classes、lines、branches 和 methods 覆蓋情況。用 HTML
  報告找未測路徑，再判斷該路徑是否對應規格行為。

### Maven 工作流程（只適用於現有設定）

```text
mvn test
mvn verify
```

只有當現有 `pom.xml` 在 lifecycle 綁定 JaCoCo 報告時，`mvn verify` 才會產生
報告。常見位置是 `target/site/jacoco/index.html`。只有在 plugin 已可用／已設定
時，`mvn jacoco:report` 才可執行。除非考試明確允許，不得新增或修改 plugin、
依賴、門檻或 `pom.xml`。

### VS Code 工作流程

如果核准環境已提供 Java Test Runner 和覆蓋率功能，可在 Test Explorer 執行相關
class／package，並用 coverage view 定位未執行路徑。編輯器標示只作診斷，不能
證明 assertion 有效。考試期間除非獲准，不得安裝擴充功能。

### 閱讀報告

1. 先檢查需求重要方法中未覆蓋的 branches。
2. 將每個未覆蓋路徑對應到規格行為。
3. 只在能表達有意義契約時新增測試。
4. 檢查已執行程式行的 assertion 是否過弱；執行過不代表驗證充分。
5. 先跑相關測試，再跑獲准的完整測試集。

## Mutation Testing

**Mutant** 是刻意修改的 production code，例如把 `<` 換成 `<=`、反轉條件、刪除
方法呼叫或改變回傳值。

- **Killed mutant**：至少一項測試在錯誤版本上失敗。
- **Surviving mutant**：所有測試仍通過；可能是輸入／assertion 不足，也可能是
  mutant 與原程式行為等價。
- **Mutation score**：常見計算為 `killed / 非等價 mutants 總數 × 100%`。

PIT/PITest 適合考前 Java 練習，因為隱藏錯誤實作在概念上類似 mutants。遇到
survivor 時，思考「哪個規格指定、可觀察的行為能分辨此變更？」不要用依賴內部
實作的 assertions。

PIT 通常需要建置設定和依賴。只在考前準備，或考試環境已提供且規則允許時使用。
不得只為安裝 PIT 而修改考試 `pom.xml`。不得檢查、探測、逆向分析或嘗試存取真正
的 hidden grader。

## AI Agent Skills 及 GitHub 資源

以下是已檢查的公開資源。原生 Skill 通常是包含 `SKILL.md` 的目錄；一般文件和
prompt 集合不會自動成為 Skill。

| Repository | 指定 Skill | 用途 | 關聯及限制 | 連結 |
|---|---|---|---|---|
| `github/awesome-copilot` | `java-junit` | JUnit、測試除錯、JaCoCo、可選 PIT、Maven/Gradle 指引 | Java/JUnit 綜合參考；部分功能可能違反 Mock 限制 | [SKILL.md](https://github.com/github/awesome-copilot/blob/main/skills/java-junit/SKILL.md) |
| `github/awesome-copilot` | `test-gap-audit` | 唯讀尋找缺漏或薄弱測試 | 有助找測試缺口；需先核對考試工具限制 | [SKILL.md](https://github.com/github/awesome-copilot/blob/main/skills/test-gap-audit/SKILL.md) |
| `mavka-ai/unit-tests-skills` | `generate-test-cases` | 分析程式並列出測試案例 | 適合需求及測試規劃 | [SKILL.md](https://github.com/mavka-ai/unit-tests-skills/blob/main/skills/generate-test-cases/SKILL.md) |
| `mavka-ai/unit-tests-skills` | `generate-tests` | 生成並執行 Java JUnit 測試 | 適合考前參考；預設規則可能與考試衝突 | [SKILL.md](https://github.com/mavka-ai/unit-tests-skills/blob/main/skills/generate-tests/SKILL.md) |
| `multica-ai/andrej-karpathy-skills` | `karpathy-guidelines` | 精簡修改、明確假設、避免過度設計 | 一般編程守則，非 Java 專用 | [SKILL.md](https://github.com/multica-ai/andrej-karpathy-skills/blob/main/skills/karpathy-guidelines/SKILL.md) |
| `robsonoliveiradacosta/agent-skills` | `add-mutation-testing` | PIT mutation testing 流程 | 面向 Quarkus/Maven，可能修改 `pom.xml`；只作考前參考 | [SKILL.md](https://github.com/robsonoliveiradacosta/agent-skills/blob/main/skills/add-mutation-testing/SKILL.md) |
| `robsonoliveiradacosta/agent-skills` | `flaky-test-triage` | 分析共享資料、時鐘、async、port 和 race | 除錯參考；小型考試題通常用不到 | [SKILL.md](https://github.com/robsonoliveiradacosta/agent-skills/blob/main/skills/flaky-test-triage/SKILL.md) |
| `robsonoliveiradacosta/agent-skills` | `spec-create` | 整理行為及驗收準則 | 規格優先參考；考試中不要因此新增檔案 | [SKILL.md](https://github.com/robsonoliveiradacosta/agent-skills/blob/main/skills/spec-create/SKILL.md) |

### OpenCode 中的 Skills

目前 OpenCode 文件描述的專案 Skill 位置是
`.opencode/skills/<skill-id>/SKILL.md`，亦支援從 `.agents/skills` 和
`.claude/skills` 搜尋。已發現的 Skill 可透過 `@skill-id` 提及載入。發現／載入
和安裝並不相同：

- **查看／載入**：使用已存在且考試允許的 Skill。
- **安裝／複製**：會寫入檔案，也可能需要網絡或 package 工具。
- **執行 helper**：可能以專案權限執行 Skill 附帶腳本。

考試期間先確認 Skills 是否已備妥且允許使用。不要因 README 有連結便安裝、複製、
更新或執行 Skill。使用前閱讀 `SKILL.md`；社群 Skill 可能和考試規則衝突，例如
要求修改 production code 或 `pom.xml`。

## 可重用 OpenCode Prompts

以下 prompt 都是供 **OpenCode CLI** 使用，請以英文輸入。替換尖括號佔位符，並以
正式規格為準。

### 1. Master examination prompt

請使用本文件開首的 [Master OpenCode Examination Prompt](#master-opencode-examination-prompt)。

### 2. 生成 JUnit 測試

```text
Inside this OpenCode session, read the full task specification, relevant production API, supplied tests, and neighboring test conventions before editing. First list the permitted test file(s) and build a compact requirement-to-test matrix. Then add independent ordinary JUnit 5 @Test methods only, unless the specification permits other forms. Prioritize boundary conditions, equivalence partitions, state transitions, repeated calls, failure-state preservation, copy independence, and inputs that distinguish likely faulty implementations. Assert observable specified behavior; do not inspect private fields, source text, files, networks, or grader internals. Do not modify production code, pom.xml, dependencies, or any unauthorized file. Run the permitted tests and report the exact command and actual result.
```

### 3. Java 除錯及修復

```text
Use only this OpenCode session and the authorized model. Read the specification and failing-test output, reproduce the failure with a permitted command, and distinguish a wrong expectation from an implementation, compilation, or environment defect. Confirm that production repair is explicitly allowed and name the permitted Java file before editing. Find the root cause, make the smallest specification-correct change, preserve unrelated behavior, and never weaken/delete tests to hide a defect. Rerun the focused test and then the permitted suite; report actual results and remaining risks.
```

### 4. 改善覆蓋缺口

```text
Inspect only the coverage report already available in this project and map missed lines/branches to the written specification and existing tests. Propose a short ranked list of meaningful missing behaviors, then add tests only in authorized files. Do not chase 100% mechanically, add assertion-free tests, install tools, or modify pom.xml. Run permitted tests and report coverage/test results actually observed; mark anything unavailable.
```

### 5. 改善 Mutation Testing

```text
Analyze the provided surviving-mutant report or suspected fault categories without accessing or reverse-engineering hidden grader internals. For each relevant survivor, identify the specification behavior, input, and observable assertion that would distinguish it. Strengthen only authorized test files, avoid implementation-coupled assertions and redundant cases, and do not install PIT or change build configuration unless explicitly permitted. Run permitted tests and report actual results.
```

### 6. 嚴格限制檔案修改

```text
Before editing, quote the specification's allowed-file list and show the exact files you intend to change. Modify only those files; do not change production code in a test-only task, supplied tests, pom.xml, dependencies, configuration, or session history. Stop and report the conflict if required work cannot be completed within that scope.
```

### 7. 最後驗證

```text
Perform final verification inside OpenCode. Re-read the requirements, inspect current diff/status, confirm every changed file is authorized, and run only permitted focused/full tests. Check that no test was weakened, disabled, or made order-dependent and that rejected operations preserve state where specified. Report requirements covered, changed files, exact commands with actual pass/fail counts, unresolved issues, and unverified assumptions. Do not alter or delete native session history; follow the official submission procedure.
```

### 8. 測試突然失敗

```text
Analyze this failure before editing. Read the specification, exact failing assertion/stack trace, relevant test, and implementation. Classify the cause as (a) incorrect expected result, (b) implementation defect, (c) compilation/build problem, or (d) environment issue, and cite evidence. Change only files permitted for this task; never change an assertion merely to match buggy behavior. Reproduce and rerun the smallest relevant test, then report actual results.
```

## 超短 OpenCode Prompts

### 分析規格

```text
Read the entire specification in this OpenCode project. List tasks, acceptance criteria, allowed/forbidden files, test restrictions, commands, and ambiguities before editing.
```

### 生成 JUnit 測試

```text
Generate high-fault-detection JUnit 5 tests from the specification in authorized test files only. Cover boundaries and state changes, then run permitted tests and report actual results.
```

### 改善測試品質

```text
Review existing tests for weak, redundant, or missing assertions against the specification. Strengthen only authorized test files and verify the suite.
```

### 尋找邊界案例

```text
Map the specification to existing tests and list untested boundaries, invalid inputs, state transitions, repeated calls, and independence cases. Rank them by likely fault detection.
```

### 除錯

```text
Reproduce this failure and classify it as expectation, implementation, compilation, or environment related. Cite evidence before making any permitted edit.
```

### 只修復指定 Java 檔案

```text
Repair only `<allowed-file.java>` with the smallest specification-correct change. Do not alter tests or other files; rerun permitted tests and report actual results.
```

### 檢查覆蓋缺口

```text
Map available JaCoCo gaps to specified behaviors and propose meaningful tests. Do not change build configuration or optimize the percentage blindly.
```

### 檢查修改檔案

```text
Inspect current diff/status and compare every changed path with the allowed-file list. Report unauthorized or suspicious changes without hiding them.
```

### 提交前驗證

```text
Recheck requirements, authorized diff, permitted test results, remaining failures, and submission instructions. Preserve all OpenCode session history and report evidence, not assumptions.
```

## OpenCode 指令參考

OpenCode 會更新，而考試環境可能鎖定舊版本。依賴指令前，先在當日環境執行
`/help` 或查看指令選單。除非考試指示要求，不要在 quiz 中更新 OpenCode。

| 指令 | 目前用途 | 考試注意事項 |
|---|---|---|
| `/init` | 文件所述版本可建立／更新 `AGENTS.md` | 會寫入檔案；檔案權限有限時不要執行 |
| `/models` | 列出／選擇當前 session 模型 | 用來確認指定模型，不可改選其他模型 |
| `/compact` | 壓縮／摘要當前 session 內容 | 可能有 `/summarize` 別名；不代表可刪除歷史 |
| `/diff` | 開啟目前 CLI 的 diff 檢視器 | 確認修改檔案均獲准 |
| `/mcps` | 目前文件記載的 TUI MCP 管理器 | 未獲明確允許時不要連接外部服務 |
| `/mcp` | 不是目前文件所述的 TUI 拼法；`opencode mcp ...` 是 CLI 指令族 | 先看 `/help`，不要自行設定 MCP |
| `/agents` | 依版本而異；文件也提供設定及 `opencode agent ...` | 啟用 subagent 前確認可用性及模型繼承 |
| `/skills` | 依版本而異；目前文件重點是自動發現和 `@skill-id` | 載入不等於安裝；未允許不要寫入 Skill 檔案 |
| `/review` | 常見為專案自訂 prompt，不保證內建 | 若不存在，直接用英文文字要求檢查 diff |
| `/status` | 不保證是 TUI 指令；`opencode service status` 回報伺服器狀態 | 伺服器狀態不等於 git 或考試進度 |

文件亦有 `/help`、`/sessions` 和 `/new`。除非明確允許，考試中避免使用
`/undo`：文件所述版本可能移除最近訊息並還原檔案修改，與保留對話紀錄的規則衝突。

### 純文字替代方式

```text
Inspect the current project diff and status. List every modified file and verify it against the specification's allowed-file list. Do not modify anything.
```

## 疑難排解

### 編譯失敗

1. 先讀第一個 compiler error，不要只看 Maven 最後摘要。
2. 對照現有程式檢查 imports、method signatures、package、Java/JUnit 版本及測試檔位置。
3. 不要安裝缺少的 library，只用專案現有依賴。
4. 用英文要求 OpenCode 作最小且獲准的修正，再跑相關編譯／測試指令。

### 測試結果意外

1. 重讀相關規格句子。
2. 縮小案例，檢查 actual 和 expected。
3. 檢查 fixture 洩漏、執行順序、共享 static state 和重複呼叫。
4. 修改前先判斷錯的是預期值還是實作。
5. 只測試的題目發現 production bug，不代表可以修改 production code。

### Maven／JaCoCo／PIT 無法使用

- 只執行專案已有的指令及 plugin。
- 未明確允許時不要新增依賴或修改 `pom.xml`。
- 如工具／報告不可用，據實報告，並繼續按規格分析及執行獲准測試。

### OpenCode 指令或 Skill 不存在

- 檢查 `/help` 和已安裝版本。
- 改用普通英文 prompt，不要依賴便利指令。
- 未明確允許時不要下載或安裝替代品。
- 準備用 Skill 和考試規則衝突時，以考試規則為準。

## 常見錯誤及最後檢查

### 常見 AI 編程錯誤

- 修改允許範圍以外的檔案。
- 在只要求測試的題目修改 production code。
- 使用禁止的 parameterized／nested／dynamic tests、private-field 存取、
  檔案／網絡存取或實作細節檢查。
- 只執行程式但沒有 assertion 驗證要求結果。
- 重複測同一類故障，卻漏掉邊界和狀態轉換。
- 忽略相鄰時段、重複值、無效範圍、重複呼叫、失敗後狀態、snapshot、重疊複製
  或物件獨立性。
- 把 100% 覆蓋率當成正確性的證明。
- 為了測試通過而修改正確預期值、刪除測試或停用測試。
- 安裝依賴、Skill、plugin 或 extension，或修改 `pom.xml`。
- 讓 subagent 使用非指定模型，或使用其他 AI 介面／API。
- 修改、刪除、偽造或選擇性省略 OpenCode session 紀錄。
- 未檢查最後 diff 或正式流程便提交。

### 最後驗證清單

- [ ] 正式考試規格優先於本 cheatsheet
- [ ] 主 agent 和 subagent 都使用獲准模型
- [ ] 所有 AI 工作均在 OpenCode 原生文字 session 中，且對話全英文
- [ ] 每項需求都有有意義的測試或實作項目
- [ ] 測試驗證可觀察行為及相關操作後狀態
- [ ] 沒有弱化、刪除或停用提供的測試
- [ ] 已執行獲准指令，記錄實際結果
- [ ] diff/status 只有明確允許的檔案
- [ ] 沒有未授權的依賴、設定或 `pom.xml` 修改
- [ ] 誠實報告限制和環境問題
- [ ] 保留 session 紀錄並按正式流程提交

## 參考資料

- [OpenCode 官方文件](https://opencode.ai/docs)
- [OpenCode Models](https://opencode.ai/docs/models)
- [OpenCode Skills](https://opencode.ai/docs/skills)
- [OpenCode Commands](https://opencode.ai/docs/commands)
- [OpenCode CLI Commands](https://opencode.ai/docs/cli/commands)
- [OpenCode MCP Servers](https://opencode.ai/docs/mcp-servers)
- [JUnit 5.14.4 User Guide](https://docs.junit.org/5.14.4/user-guide/)
- [JaCoCo 官方文件](https://www.jacoco.org/jacoco/trunk/doc/)
- [PIT Mutation Testing](https://pitest.org/)
- [Awesome Copilot Skills 索引](https://github.com/github/awesome-copilot/blob/main/docs/README.skills.md)

> 資料核對日期：2026-11-12。OpenCode 指令和 Skill 相容性會變動，請以考試
> 環境安裝版本為準。
