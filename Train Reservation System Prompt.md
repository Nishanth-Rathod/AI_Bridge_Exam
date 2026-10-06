# Train Reservation System — UI Automation Assessment
## Prompt Pack

The assessment scores two things: **what you discovered** (element IDs, field types, validation rules) and **how you directed your prompts** (research, generation, code, refinement). Good prompts carry your own findings into them.

Replace every `[BRACKET]` with values from your own inspection — that is the part the grader credits as *you driving*.

You get only **three Evaluate attempts**. Run them at ~50%, ~90%, and final — not for small checks. Feed earlier outputs back as context instead of re-pasting the whole app, to save tokens.

---

## 🔍 Prompt 1 — Research: inspect the three modules

You are a senior QA analyst. I'm inspecting a static client-side web app (Train Reservation System). I'll paste the page HTML and its JS file for one module at a time. For each module, produce a structured analysis section containing:

1. Every interactive element with: element ID (or name/selector if no ID), element type (text input / dropdown / checkbox / button), and its label.
2. Where each error renders: inline container (give the div id) versus native alert()/confirm() dialog.
3. The REAL validation logic, read from the JS — not guessed from the UI. For each field state the exact trigger condition and the exact message string.
4. Any behaviour quirks (redirects, auto-clearing messages, flags like secureCheck, functions that only fire on certain buttons).

Module: [LOGIN]. Here is login.html:
[PASTE HTML]
Here is login.js:
[PASTE JS]

Output as a clean text section I can paste into Application Analysis & Requirements.txt. Be precise with IDs and message strings. No filler.

*(Repeat for Ticket Booking: index.html + script.js, and Enquiry: contactus.html + contact.js.)*

---

## 📋 Prompt 2 — Generation: acceptance criteria (Step 2)

Act as a BA. Write acceptance criteria for user stories US-01 to US-26 of the Train Reservation System, grouped by module (Login US-01–09, Ticket Booking US-10–21, Enquiry US-22–26).

Use Given/When/Then format. Base every criterion on these validation rules:

LOGIN: Username must equal "admin" (blank → "Username cannot be empty", else → "Username is wrong"); Password must equal "admin" (blank → "Password cannot be empty", else → "Password is wrong"); Captcha field non-empty (blank → "Captcha code cannot be empty"); captcha Validate button must be clicked and typed text must match the 7-char generated code EXACTLY (case-sensitive) to set secureCheck=true; login success needs admin + admin + secureCheck → alert("Login Successful") → redirect to index.html; Remember-me confirm() OK → "Username and Password Saved Successfully!", Cancel → "Changes not saved!" (nothing persisted).

TICKET BOOKING: 8 mandatory fields (Travel From, Travel To, Departure, Class dropdown, Passenger Name, Email, Phone, No. of Passengers), each with its "can't be blank" message, and all failing messages joined with HTML line-break tags into the single error container div#errfn; Class default option invalid → "DropDown can't be blank"; passengers "0" → "Number of Passengers can't be Zero"; fare = passengers × class price (ACSleeper 2500 / Sleeper 1250 / Seating 750), VAT = round(subtotal × 0.02), total = subtotal + VAT, computed only by +/- buttons.

ENQUIRY: Full name ≥3 chars; Email must contain both "." and "@"; Message ≥15 chars; validation is sequential with early return (only the first failing field shows a message); success → "Thank you! We will get back to you as soon as possible." (auto-clears after 3s).

Also write explicit criteria for rules the app does NOT enforce: booking email not format-validated, phone accepts non-digits, Departure date format not validated, passenger count has no upper bound. Keep it tight.

---

## 🧾 Prompt 3 — Research: consolidated rules table (Step 3)

Using the analysis and acceptance criteria above as context, produce a single consolidated validation-rules reference table for all three modules. Columns: Module | Field | Enforced rule | Exact error message | NOT enforced (gaps). Explicitly list every gap (email format, numeric phone, date format, passenger upper bound, captcha case-sensitivity edge cases). Output as a plain-text table for the .txt file.

---

## 🧪 Prompt 4 — Generation: test cases → testcases.csv (Step 4)

You are a senior test designer. Generate 40–45 test cases covering ALL 26 user stories of the Train Reservation System. Use the validation rules and element IDs already established in this conversation as context.

Distribution: Login 13 (US-01–09), Ticket Booking 19 (US-10–21), Enquiry 10 (US-22–26) — totalling 40–45, every user story covered.

Priority mix:
- 60–70% must be Negative/Validation scenarios at Critical or High priority.
- Boundary cases = Medium: name of exactly 2 and 3 chars, message of exactly 14 and 15 chars, passenger count decremented below zero, class changed after count is set, case-altered 7-char security code.
- Positive/success cases = Low.

Include cases for the NON-enforced rules too (booking email with no @, non-numeric phone, bad date format, large passenger count).

Output ONLY a CSV with EXACTLY these columns in this order:
TC_ID, Module, User_Story_ID, Test_Scenario, Test_Type, Priority, Precondition, Test_Steps, Test_Data, Expected_Result, Actual_Result, Status

Rules: TC_ID sequential (TC_001…). Test_Steps numbered inside the cell. Expected_Result must quote the exact message string. Leave Actual_Result empty and set Status = "Not Executed". Quote any field containing commas. No commentary before or after the CSV.

---

## 💻 Prompt 5 — Code: automation framework (Step 5)

Stack: Selenium WebDriver + Java + TestNG + Maven, Page Object Model.

Build a complete automation framework for the Train Reservation System on top of my existing workspace (do NOT restructure it). URLs:
- Login: https://webapps.tekstac.com/SeleniumApp1/TrainReservation/login.html
- Ticket Booking: https://webapps.tekstac.com/SeleniumApp1/TrainReservation/index.html
- Enquiry: https://webapps.tekstac.com/SeleniumApp1/TrainReservation/contactus.html

Deliver:
1. Browser setup/teardown (base test class).
2. One page object per module: LoginPage, BookingPage, EnquiryPage. Use these locators: [PASTE YOUR ELEMENT IDs PER FIELD].
3. Reusable helpers:
   - Alert/confirm handler (accept/dismiss native dialogs, read alert text).
   - "fill all valid fields except X" helper for the booking form.
   - A login helper that: enters admin/admin, reads the LIVE captcha from element #code, types it, clicks Validate, accepts the "Valid input" alert, then clicks Login. The captcha regenerates on every load, so it must be read at runtime — never hard-coded.
4. At least 25 test methods with assertions (≥7 per module). Prioritise Critical/High negative validations over happy paths. Assert the exact message strings. For booking, remember all 8 error messages concatenate with HTML line-break tags into the error div with id errfn. For fare, trigger calculateTotal via the +/- buttons and assert subtotal/VAT/total against ACSleeper 2500 / Sleeper 1250 / Seating 750 with VAT = round(subtotal × 0.02).

Follow standard naming conventions. Produce the full file tree and the code for each file.

*(Swap the Stack line for Playwright/Cypress or another language if needed — keep the rest.)*

---

## 🔧 Prompt 6 — Refinement: fix first-run failures (Step 6)

I ran the tests. Here are the failures and stack traces:
[PASTE FAILURES]

Two known trouble spots to check first:
- Captcha alert flow: the Validate click raises alert("Valid input") which must be accepted BEFORE clicking Login; a stale read of #code or a dialog-timing race causes failures. Make the helper wait for the alert, accept it, then proceed.
- Fare recalculation timing: calculateTotal() fires only on +/- button clicks, not on direct input; assert the total only after clicking the button and waiting for the summary/total element to update.

Fix the failing tests and correct any wrong assertions (exact message strings, locator mismatches). Show only the changed methods/files.

---

## 🐞 Prompt 7 — Generation: defect report → defects/defect_report.json (Step 7)

From my ACTUAL test-run results below (not predictions), produce a single consolidated defect report at defects/defect_report.json. Document exactly 8 validation defects, at least 2 per module.

My observed results:
[PASTE WHAT YOU ACTUALLY SAW: input used, actual behaviour, which test, which module]

JSON: an array of defect objects, each with: id, module, user_story_id, summary, input_entered, actual_result, expected_result, severity, priority. Likely real defects to look for: booking accepts email with no @/domain, phone accepts letters, Departure accepts non-date text, passenger count decrements below zero, enquiry sequential early-return hides later errors, captcha case-sensitivity. Output valid JSON only.

---

## Files to submit (reference)

| File | What it contains |
| --- | --- |
| Application Analysis & Requirements.txt | Analysis of all three modules: element IDs, field types, acceptance criteria, validation rules (Prompts 1–3) |
| testcases.csv | 40–45 test cases across 26 user stories (Prompt 4) |
| Automation codebase | One page object per module + reusable helpers (login helper, alert/confirm handler) (Prompts 5–6) |
| defects/defect_report.json | 8 real defects, ≥2 per module (Prompt 7) |
