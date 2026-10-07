# Library Portal — LEAN playbook (2 hrs, token-aware)

3 prompts, paste in order, wait for each. Runs are fast (client-side), so don't over-iterate.

---

## PROMPT A — context + inspect + analysis + test cases

```
AI-assisted UI automation assessment, prompt-only — I can't open a browser or inspect anything.
WORK ECONOMICALLY: write all outputs straight to disk; never paste file contents, DOM, or test
cases back into chat — one short status line per step. Don't change the workspace structure;
keep it clean; run headless from the VS Code terminal; never fabricate IDs/behaviour/results.

APP: Library Management Portal, 3 client-side modules, NO login anywhere:
- Library Card https://webapps.tekstac.com/SeleniumApp2/Library/LibraryCard.html (US-01–12)
- Services     https://webapps.tekstac.com/SeleniumApp2/Library/Services.html     (US-13–22)
- Membership   https://webapps.tekstac.com/SeleniumApp2/Library/MemberShip.html   (US-23–28)
Membership is gated by a drag-and-drop (book -> drop zone); form unreachable until it succeeds.
Services has Email/Call/Chat paths (Call = no submit, no validation). Amount fields are
read-only/auto-filled (not inputs). No persistence across reload.

Do now, one turn, in order:
1. Read the workspace User Stories file (source of truth); confirm URLs + US ranges.
2. Inspect all 3 pages with a throwaway headless Playwright script (temp folder, delete after).
   Extract ONLY the control list (tag/id/name/type/placeholder), <select> options, and
   error/banner div ids — do NOT print full HTML. Capture: Library Card Role->School/Company
   reveal + Action->Amount autofill; Services medium reveals; Membership drag source + drop zone.
   Read the linked page JS to confirm real validation. Submit empty + invalid data to record the
   actual error text.
3. Write "Application Analysis & Requirements.txt": element IDs, field types, observed behaviour,
   acceptance criteria (Given/When/Then, US-01–28 by module), and a validation-rules table that
   includes rules NOT enforced (non-numeric Age, alphabetic Phone, no-format email on Services
   "From", symbol-only Card Number).
4. Write "testcases.csv", 40–45 cases, all US-01–28. Columns exactly: TC_ID, Module,
   User_Story_ID, Test_Scenario, Test_Type, Priority, Precondition, Test_Steps, Test_Data,
   Expected_Result, Actual_Result, Status. 60–70% negative (Critical/High), positive=Low,
   boundary/format-gap=Medium. MUST include negatives for: non-numeric Age, negative Age,
   alphabetic Phone, email no-TLD (Services From), symbol-only Card Number, Membership
   gate-bypass. Leave Actual_Result/Status = "Not Executed".

Reply with a 3-line summary only.
```

---

## PROMPT B — automation framework

```
Build a Playwright + TypeScript Page Object Model ON TOP of the existing workspace (match its
config; don't change the structure). One page object per module: Library Card, Services,
Membership. Helpers: a drag-and-drop helper for the Membership gate + a "fill-all-valid-except-X"
helper. NO login helper (there is no login). >=30 test methods with real assertions (~12/10/8),
negatives first. Don't submit the Services Call path; don't treat read-only Amount as an input;
assert on the success banner / ID-card text, not on persistence. Use the real IDs/rules from
Application Analysis & Requirements.txt — don't invent selectors. Reporters: built-in html + json
only (no Allure). Headless. Write files to disk, don't echo them. Don't run yet.
```

---

## PROMPT C — execute + refine + defects

```
Run the full suite headless from the terminal; show the command + summary result only (not full
logs). Iterate tightly:
- Classify each failure: locator / timing / test-data / assertion / genuine app defect.
- Fix automation issues; re-run. The Membership gate is the likely failure — if Playwright's
  dragTo doesn't trip it, dispatch native dragstart/dragover/drop and verify the form appears
  before asserting.
- NEVER weaken or delete a valid assertion just to pass. Cap at 2 corrective runs.
- Update Actual_Result/Status in testcases.csv from the real results.

Then, from the ACTUAL run results ONLY (not predictions), write defects/defect_report.json:
EXACTLY 5 distinct validation defects (format gaps are the likeliest source). JSON array; each
object: id, module, user_story_id, test_case_id, summary, input_entered, steps, actual_result,
expected_result, severity, priority, status. Each must trace to a failing TC_ID. Valid JSON only.
Don't fabricate to reach 5.
```

---

## 2-hour timeline

| Time | Do | Evaluate |
|---|---|---|
| 0:00–0:35 | Prompt A (analysis + testcases.csv) | **Evaluate #1** right after |
| 0:35–1:05 | Prompt B (framework) | — |
| 1:05–1:45 | Prompt C (run + fix + defects) | **Evaluate #2** when ~90% done |
| 1:45–2:00 | final fixes + check all files saved | **final Evaluate** |

## Token savers (what keeps the agent lean)
- Writes to disk, no file/DOM/CSV echoed back to chat.
- Inspector prints only the control list, never full page HTML.
- Edits files in place; doesn't regenerate what's already written.
- Max 2 corrective runs; log stubborn failures instead of looping.
- You get only **3 Evaluate attempts** — don't waste one early.
