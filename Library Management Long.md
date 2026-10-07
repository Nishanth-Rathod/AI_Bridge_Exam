# UI Automation Assessment — Prompt Playbook (VDI / prompt-only)

**Use this when the only thing you can do is prompt the agent inside VS Code.**
You cannot open a browser or inspect anything by hand — so every prompt below tells the
agent to do that work itself (write a script, run it, read its own output).

### How to run it
1. Paste **STEP 0 — PROFILE** first, as your very first message. Fill the 4 blanks from the
   on-screen **Tasks / Deliverables**. The agent keeps this in context for everything after.
2. Then paste **Prompt 1 → 7 in order**, waiting for each to finish before the next.
3. Don't edit prompts 1–7 — they read the values from the Profile.

---

## STEP 0 — PROFILE (fill once)

```
PROFILE — keep this in mind for every prompt that follows.

TECH_STACK: Playwright + TypeScript + Page Object Model.
  (If the workspace is already scaffolded for a different stack/language, MATCH the existing
   stack instead and tell me what you found — do not change the workspace structure.)

Fill these 3 from the on-screen Tasks / Deliverables:
  TEST_CASE_COUNT : ____   (e.g. "60+", "45–50", "40–45")
  METHOD_COUNT    : ____   (minimum automated test methods, e.g. "20+", "25+", "30")
  DEFECT_RULE     : ____   (e.g. "exactly 8, at least 2 per module" / "exactly 5" / "at least 8")

  DEFECT_PATH     : ____   (from Deliverables: "defect_report.json" OR "defects/defect_report.json")
  CSV_FILENAME    : testcases.csv   (use the EXACT name the Deliverables specify if different)

Everything else — module names, URLs, user-story ranges, element IDs, field types,
whether login is enforced, captcha / dialog / drag-and-drop mechanics — you will DISCOVER
in Prompt 1 by reading the workspace and inspecting the live pages. Do not ask me for them.

Hard rules for the whole task:
- Do NOT change, rename, or restructure the provided workspace. Build on top of it.
- Keep the workspace clean — no redundant files or folders.
- Run everything headless, from the integrated VS Code terminal.
- Never fabricate element IDs, behaviour, test results, or defects.
```

> **Shortcut:** if you can copy the on-screen question, paste the whole **Tasks + Deliverables +
> Minimum Passing** text under the Profile and add: *"Extract TEST_CASE_COUNT, METHOD_COUNT,
> DEFECT_RULE and DEFECT_PATH from this yourself."*

---

## PROMPT 1 — Inspect & analyse (the agent does it all)

```
You are a senior QA analyst. I am in a locked VDI: I CANNOT open a browser or inspect anything
myself. You must do all discovery programmatically and base everything on real evidence.

1. Read the workspace first. List the files, open the user-stories / requirements file(s), and
   extract: the module names, each module's URL, and the user-story ranges (US-xx to US-yy per
   module). Treat that file as the source of truth. Tell me what you found.

2. Inspect every module page yourself. Write a TEMPORARY Playwright script (put it in a temp
   folder, delete it when done — do not pollute the graded workspace) that, for each URL:
     - navigates to the page (log in first if a module requires it),
     - prints every form control: tag, id, name, type, placeholder,
     - prints every <select>'s option values,
     - prints the id/text of every error / message / banner container,
     - notes dynamic behaviour: conditional field reveals, auto-filled/read-only fields,
       captcha box + where its answer is displayed, native alert()/confirm() dialogs,
       drag-and-drop zones.
   Run it headless in the terminal and read the output.

3. Confirm the REAL validation logic. Fetch/read any linked page scripts (e.g. script.js,
   login.js, contact.js). Derive rules from the code, not from guessing at the UI.

4. Observe actual behaviour: programmatically submit empty forms and invalid values on each
   page and record the exact error text / behaviour that appears.

5. Explicitly list the rules the app does NOT enforce (e.g. email accepted with no "@",
   non-digit phone accepted, non-date text accepted in a date field, count allowed below zero,
   case-insensitive captcha). These are my highest-value defect candidates.

6. State clearly, per module: is login enforced to reach it? What is the exact auth/captcha
   mechanism (plain creds / arithmetic captcha shown in a result box / N-char case-sensitive
   captcha behind a Validate button / native alert+confirm / drag-and-drop gate / none)?

7. Create "Application Analysis & Requirements.txt" and write ALL of the above into it, organised
   by module. Save it to disk. Do not just print to chat.

Do not generate test cases or framework code yet.
```

---

## PROMPT 2 — Acceptance criteria

```
Act as a Business Analyst. For EVERY user story discovered in Prompt 1 (all modules, full US
range), write acceptance criteria in Given/When/Then form, grounded in the actual fields,
validation rules and behaviour you observed — not generic assumptions. Group them by module.

Also add a short end-to-end flow description for each module (how a user moves through it,
what valid input each field needs, what success looks like).

APPEND all of this to the existing "Application Analysis & Requirements.txt" — do NOT overwrite
Prompt 1's content. Save to disk.
```

---

## PROMPT 3 — Validation rules table

```
Using the analysis and acceptance criteria above as context, produce ONE consolidated
field-level validation-rules reference TABLE covering all modules, with columns:
  Module | Field | Rule | Enforced? (Yes/No) | Error message shown

Include the rules the app does NOT enforce (Enforced? = No) from Prompt 1 — they matter for the
defect report. This is a table, not Gherkin.

APPEND it to "Application Analysis & Requirements.txt". Save to disk.
```

---

## PROMPT 4 — Test case generation

```
Act as a senior test designer. Generate the number of test cases given by TEST_CASE_COUNT in the
Profile, covering EVERY user story across ALL modules (not a subset).

Write them to a CSV named exactly as CSV_FILENAME in the Profile, with EXACTLY these columns:
TC_ID, Module, User_Story_ID, Test_Scenario, Test_Type, Priority, Precondition, Test_Steps,
Test_Data, Expected_Result, Actual_Result, Status

Rules:
- Priority values: Critical, High, Medium, Low.
- 60–70% must be negative/validation tests, at Critical or High priority.
- Positive/success tests: Low priority.
- Boundary tests: Medium priority.
- Cover positive, negative, validation, boundary, mandatory-field and invalid-format scenarios.
- Base every case on the real element IDs and validation rules in the analysis file.
- Leave Actual_Result and Status as "Not Executed" (they get filled after the run).

Before finishing, verify every discovered US-ID appears in at least one row, and report any gaps.
Save to disk.
```

---

## PROMPT 5 — Automation framework

```
Build a complete automation solution for ALL modules using TECH_STACK from the Profile and the
Page Object Model.

- First, inspect the existing workspace config and MATCH it (framework, folders, config files).
  Build on top of the structure; do not change it. Scaffold fresh only if it's empty.
- Implement at least METHOD_COUNT (from the Profile) automated test methods, each with meaningful
  Playwright assertions. Prioritise critical/high-risk negative and validation scenarios over
  happy-path.
- Add helpers ONLY where this app needs them, based on Prompt 1's findings:
    * a reusable login helper IF (and only if) a module requires login — and it must handle the
      EXACT captcha/alert mechanism you discovered (read the displayed answer, type it, handle
      the success alert). Do NOT add login to modules that open directly.
    * a native alert()/confirm() dialog handler if the app uses them.
    * a drag-and-drop helper if a verification/upload drop zone exists.
- Use the real inspected element IDs and validation rules from the analysis file. Do not invent
  selectors.
- Reporting: use Playwright's BUILT-IN "html" + "json" reporters only. Do NOT add Allure or any
  dependency that needs an extra install (the VDI may block it).
- Run headless.

Save all files to disk. Do not execute yet.
```

---

## PROMPT 6 — Execute & refine

```
Run the full suite from the integrated VS Code terminal. Show me the exact command and its
actual output — do not assume anything passed.

For every failure, first classify it: locator / navigation / timing / test-data / assertion /
genuine application defect. Then:
- Fix automation issues (selectors, waits, navigation, test data).
- Correct an assertion ONLY when the app's actual behaviour proves the original assertion was
  wrong. NEVER delete or weaken a valid assertion just to make a test pass — that would hide the
  real defects I need to find.
- Re-run failed tests, then re-run the whole suite.
- Update Actual_Result and Status in the CSV from the real results.

Produce the html + json report as execution evidence. Save to disk.
```

---

## PROMPT 7 — Defect report

```
From YOUR OWN actual execution results — the terminal output and the json/html report, NOT
predictions and NOT anything I told you — produce a single consolidated defect report at
DEFECT_PATH (from the Profile).

Document the number and distribution given by DEFECT_RULE in the Profile. Output a JSON array of
defect objects, each with:
  id, module, user_story_id, test_case_id, summary, input_entered, steps,
  actual_result, expected_result, severity, priority, status

Every defect must trace to a specific failing TC_ID and name the assertion that failed. Do not
fabricate defects to reach the count — if genuine defects fall short, run more negative/boundary
tests first, then re-derive. Output valid JSON only, and save it to DEFECT_PATH.
```

---

## What changed from your version (and why)

| Your version | Problem under prompt-only VDI | Fix |
|---|---|---|
| P1 "you need to inspect" | you can't — no browser | agent writes+runs an inspector script, reads workspace, fetches JS |
| P7 "paste what you saw" | you have nothing to paste | defects come from the agent's own run + report |
| "headless so I can see" | contradictory; you can't watch anyway | headless everywhere; HTML+JSON report is the evidence |
| P4 "45–50" + "at least 50" + "all 20 US / 3 modules" | contradictory + hardcoded | single count from Profile; cover all discovered US |
| "Test Design.csv" | wrong name, has a space | `testcases.csv` (or exact Deliverables name) |
| login+captcha for every module | only true for some apps | login helper built conditionally on what P1 finds |
| "at least 20 methods", "exactly 8 defects" | hardcoded | Profile: METHOD_COUNT, DEFECT_RULE, DEFECT_PATH |
| Allure report | needs install the VDI may block | built-in Playwright html+json reporters |
| P7 "likely defects: email no @, phone letters…" | Train-only; invites fabrication | removed; each defect must cite a failing TC_ID |
| "Ctrl+S" | agent writes files directly | "save to disk" |
| Gherkin in P3 | wrong place (rules = a table) | Given/When/Then in P2; table in P3 |
