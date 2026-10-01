# GitHub Copilot — AI Augmented QA — Corrected Prompts

These are the three assessment prompts, with the issues fixed. Each is a single, self-contained,
autonomous prompt — paste the whole block as one message and let it run end-to-end.

**Summary of changes**
- **ParaBank** — fully rewritten to match the stronger structure of the other two: added inspect-first, do-not-fabricate, and the execute → investigate → fix → re-run loop; removed the "wait for user confirmation" ending and the conversational "if you want I can give mini prompts" line; added a note about the shared `fromAccountId` id and the register→login flow needed to reach modules 2 and 3.
- **Smart University** — changed all "US-01 through US-20 / all 20 user stories" to cover **US-01 through US-21** (the scope lists Add Professor as US-14 to US-21); reconciled the "strictly Playwright" vs "you may use any tool" contradiction; added an explicit autonomous-run directive.
- **College Social Network** — already solid; added only a stop-and-report safeguard for environments that can't actually execute, plus an explicit autonomous-run line.


---
---


# 1. ParaBank — Corrected Prompt

ROLE

Act as a senior QA automation engineer, SDET, test architect, requirements analyst, and debugging specialist working directly inside the current VS Code workspace.

You are completing an AI-assisted UI automation assessment for the ParaBank (Tekstac-hosted) web application.

Your objective is NOT merely to generate code. Your objective is to:
inspect → analyze → document → derive requirements → identify validation rules → generate comprehensive test cases → implement automation → execute tests → investigate failures → fix automation issues → re-run → identify genuine application defects → document actual defects → perform final regression verification.

The final workspace must contain a complete, executable, maintainable solution satisfying every assessment requirement.

You are assisting me with the ParaBank UI Automation assignment. Follow the assignment exactly and work with the existing workspace. Analyse the existing project workspace first — do NOT change, delete, rename, or restructure it. Implement the solution on top of the existing structure only. Avoid redundant files/folders.

TECHNOLOGY
- Inspect the existing workspace and use whatever framework/language it already contains (Selenium, Playwright, or Cypress in Java, JavaScript, TypeScript, Python, or C#/.NET).
- If the workspace has no existing framework, use Playwright with TypeScript and the Page Object Model.
- Use reusable fixtures/helpers/utilities, meaningful assertions, reliable locators, and proper waits/synchronization.
- Use the integrated VS Code terminal for installation, execution, debugging, and validation.

The assessment evaluates both:
1. What was analysed/discovered and shared with AI: application analysis, element IDs, field types, acceptance criteria, and validation rules.
2. How AI prompts were structured: research, generation, code-generation, and refinement/follow-up prompts.

Perform the following steps in order.

APPLICATION UNDER TEST

System: ParaBank (Tekstac hosted)
Registration URL: http://webapps.tekstac.com:9090/tekbank/register.htm
Open New Account and Fund Transfer: accessed by navigating the application after login — no separate entry URLs.

Flow note: Registration creates a user. To test Open New Account and Fund Transfer you must first register a user (or reuse one) and ensure you are logged in, then navigate to those modules. This assignment does not mention a captcha (unlike some other Tekstac apps) — verify on the live page, and if a captcha or other gate appears, handle it in the helper.

SCOPE
- Module 1 — Registration: 11 fields, US-01 to US-07
- Module 2 — Open New Account: Account Type and Source Account, US-08 to US-10
- Module 3 — Fund Transfer: Amount and Account Selection, US-11 to US-13

Read any user stories/use cases already available in the workspace and treat them as the source of truth.

ELEMENT IDs (verify against the live DOM before use — do not trust blindly)

Registration
- customer.firstName
- customer.lastName
- customer.address.street
- customer.address.city
- customer.address.state
- customer.address.zipCode
- customer.phoneNumber
- customer.ssn
- customer.username
- customer.password
- repeatedPassword
- Register button: input[type="submit"][value="Register"]

Open New Account
- type (Account Type: CHECKING / SAVINGS)
- fromAccountId (Source Account — dynamically loaded dropdown)
- "$100.00" is a static display label — do not test or interact with it
- Open Account button: input[type="submit"][value="Open New Account"]

Fund Transfer
- amount
- fromAccountId (From Account — dynamically loaded dropdown)
- toAccountId (To Account — dynamically loaded dropdown)
- Transfer button: input[type="submit"][value="Transfer"]

Note: both Open New Account and Fund Transfer use id="fromAccountId". Scope every locator to its own page/page-object; do not share a single global locator across modules.

VALIDATION RULES (these are the EXPECTED behaviour — the oracle)

Registration
- First Name / Last Name / City: letters only; reject digits and special characters
- State: exactly 2 alphabetic characters
- Zip Code: exactly 5 numeric digits
- Phone: exactly 10 numeric digits
- SSN: exactly 9 numeric digits
- All 11 fields mandatory; empty submission must show a validation error

Open New Account
- Account Type: required (CHECKING or SAVINGS)
- Source Account: required, selected from the dynamically loaded dropdown

Fund Transfer
- Amount: required, numeric, positive (> 0)
- From Account: required
- To Account: required

Treat these rules as the expected result for assertions. The goal is to find where the live application FAILS to enforce them — compare expected (these rules) against actual (observed behaviour) and log every gap.

STEP 1 — INSPECT ALL THREE MODULES
Open the Registration page, then register/log in to reach the other two modules. For each form, inspect and document: HTML element IDs, labels and field types, buttons/navigation, default values, success/error messages, mandatory fields, client/application validation, empty-form behaviour, and invalid-value behaviour. Actually submit empty forms and invalid values to observe real behaviour. Do not assume behaviour.

STEP 2 — DEFINE ACCEPTANCE CRITERIA
Create acceptance criteria for every user story US-01 through US-13 using the existing user stories plus actual observed behaviour.
Create: Application Analysis & Requirements.txt
It MUST contain: application overview, navigation flow, module analysis, US-01 through US-13, acceptance criteria, element IDs, field types, actual observed behaviour, validation rules, success/error messages, and any discrepancies discovered.
Use this file as context for subsequent test-case and code generation.

STEP 3 — RESEARCH VALIDATION RULES
Document all field-level validation rules for all three modules in Application Analysis & Requirements.txt (the rules above plus any additional rules discovered during inspection).

STEP 4 — GENERATE COMPREHENSIVE TEST CASES
Generate AT LEAST 60 detailed test cases covering ALL 13 user stories across all 3 modules.
Create a CSV containing exactly:
TC_ID, Module, User_Story_ID, Test_Scenario, Test_Type, Priority, Precondition, Test_Steps, Test_Data, Expected_Result, Actual_Result, Status
Requirements:
- Every US-01 through US-13 covered.
- Include positive, negative, validation, boundary, mandatory-field, invalid-format and functional scenarios.
- 60–70% MUST be negative/validation tests.
- Negative/validation tests: Critical or High priority.
- Positive/success tests: Low priority.
- Boundary tests: Medium priority.
- Allowed priorities: Critical, High, Medium, Low.
- Use the actual validation rules, element IDs and observed behaviour. Do not invent behaviour.
- Update Actual_Result and Status after execution where applicable.

STEP 5 — AUTOMATION IMPLEMENTATION
Create a complete automation solution using the Page Object Model.
Required: browser setup/configuration; a reusable registration+login helper/flow to reach the post-login modules; page objects for Registration, Open New Account, and Fund Transfer; test methods; reusable utilities where needed; meaningful assertions; reliable locators; proper waits.
Implement AT LEAST 20 automated test methods across all 3 modules.
Prioritize: (1) critical validations, (2) high-risk negative tests, (3) boundary tests, (4) positive scenarios.
Use the actual element IDs and validation rules discovered during inspection. Do not guess selectors.

STEP 6 — EXECUTE AND REFINE
Run the tests using the integrated VS Code terminal. Do not stop at the first execution. For every failure:
1. Inspect the actual error.
2. Determine whether it is a locator, navigation, timing, test-data, assertion, application-behaviour, or genuine application-defect issue.
3. Fix automation issues.
4. Correct assertions ONLY when actual application behaviour proves the original assertion was wrong.
5. Re-run failed tests, then re-run the full suite where practical.
6. Improve stability and coverage.
Do NOT remove assertions or weaken valid validations just to make tests pass. Provide execution evidence (reports/artifacts) where supported.

STEP 7 — DOCUMENT DEFECTS
Document defects based ONLY on actual observed execution. Create exactly ONE consolidated file: defect_report.json
It must contain AT LEAST 8 documented validation defects. Each defect must include at minimum: Defect ID, Module, User Story ID, Test Case ID, Test Data / what was entered, Steps/scenario, Expected Result, Actual Result, Severity, Priority, Status.
Defects MUST come from actual test execution — not prediction or assumption. Execute enough negative/boundary/validation tests to obtain genuine observations across all three modules. Do not fabricate defects merely to reach eight.

FINAL DELIVERABLES
1. Application Analysis & Requirements.txt — analysis of all 3 modules, navigation flow, element IDs, field types, US-01 to US-13, acceptance criteria, validation rules, actual observed behaviour.
2. Test case CSV — 60+ cases, all 13 user stories, required columns, 60–70% negative/validation, correct priorities.
3. Automation codebase — POM, registration+login helper, page objects for Registration, Open New Account, Fund Transfer, at least 20 test methods, meaningful assertions.
4. defect_report.json — one consolidated JSON file, at least 8 real defects, each with expected result, actual result, severity, priority, and execution details.

MINIMUM PASSING EXPECTATIONS (verify every item before finishing)
- Application Analysis & Requirements.txt exists with documented findings.
- 60+ test cases covering all 13 user stories.
- Risk-based coverage with 60–70% negative/validation scenarios.
- At least 20 automated tests with meaningful assertions.
- Page objects for Registration, Open New Account, and Fund Transfer.
- All three modules automated.
- Tests have actually been executed; failed tests investigated and refined.
- defect_report.json exists with at least 8 execution-based defects, each including expected result, actual result, severity, and priority.
- Existing workspace structure unchanged; no redundant files/folders; all files saved.

IMPORTANT CONSTRAINTS
- Do NOT change the provided workspace structure.
- Do not create unnecessary/redundant files or folders.
- Use the VS Code integrated terminal for execution.
- Do not fabricate element IDs, validation behaviour, test results, or defects.
- Inspect the live application first and base everything on actual behaviour.
- Use AI assistance, but ensure the analysis, requirements, and prompts demonstrate that I am directing the process.
- The final solution must satisfy the assessment's evaluation criteria, not merely produce sample code.
- If the environment genuinely cannot launch a browser or reach the application, STOP and report this clearly in Application Analysis & Requirements.txt and to me — do NOT invent element IDs, results, or defects to fill the gap.

EXECUTION MODE
Work autonomously through all seven steps in order without pausing for confirmation between steps. Start by inspecting the existing workspace and the live application; do not immediately generate code. Then proceed through every step and leave the completed, executed solution in the workspace.


---
---


# 2. Smart University — Corrected Prompt

ROLE

Act as a senior QA automation engineer, SDET, test architect, requirements analyst, and debugging specialist working directly inside the current VS Code workspace.

You are completing an AI-assisted UI automation assessment for the SmartUniversity / Rocky Global University web application.

Primary implementation stack:
- Playwright + TypeScript + Page Object Model
- Reusable fixtures/helpers/utilities
- Playwright assertions
- Integrated VS Code terminal for execution

If the existing workspace already uses a different supported framework/language (Cypress or Selenium in JavaScript, TypeScript, Python, Java, or C#/.NET), match the existing workspace instead of forcing a rewrite.

Your objective is NOT merely to generate code. Your objective is to:
inspect → analyze → document → derive requirements → identify validation rules → generate comprehensive test cases → implement automation → execute tests → investigate failures → fix automation issues → re-run → identify genuine application defects → document actual defects → perform final regression verification.

The final workspace must contain a complete, executable, maintainable solution satisfying every assessment requirement.

You are assisting me with the SmartUniversity UI Automation assignment. Follow the assignment exactly and work with the existing workspace. Do NOT change, delete, rename, or restructure the provided workspace. Implement on top of the existing structure only. Avoid redundant files/folders.

USER-STORY SCOPE NOTE (read carefully)
The assignment text in places says "US-01 to US-20 / all 20 user stories", but the module scope lists Add Professor as US-14 to US-21. Cover ALL of US-01 through US-21. Covering US-21 is a superset and cannot hurt your score.

1. Application Under Test

System: SmartUniversity (Rocky Global University)
URL: http://webapps.tekstac.com/SeleniumApp1/SmartUniversity
Login: Username: admin | Password: admin#123

There are 3 modules:
1. Login — login.html, US-01 to US-05
2. Add Student — add_stud.html, US-06 to US-13
3. Add Professor — add_prof.html, US-14 to US-21

Every module requires login using admin/admin#123.

2. Login/Captcha Flow

The login page contains username, password, a dynamic arithmetic captcha, a captcha result box, a captcha input, remember-me, and submit.

The captcha operation is dynamic, e.g. "98 + 65 =". The computed answer is displayed on the page in a result box. Automation must read that displayed answer and enter it into the captcha field.

For valid login, a browser alert "Login Successful" appears. Automation must accept/dismiss the alert before the home page (index.html) loads.

After login:
- Students menu → Add Students → add_stud.html
- Professor menu → Add Professor → add_prof.html

Inspect all fields, element IDs, error-message elements, captcha behaviour and validation behaviour before implementing tests.

3. Validation Rules (these are the EXPECTED behaviour — the oracle)

Login
- Username must equal admin
- Password must equal admin#123
- Captcha must equal the displayed arithmetic answer
- Empty captcha → "Captcha cannot be empty"
- Wrong captcha → "Invalid Captcha"

Add Student
- Student Name: letters and spaces only
- Father Name: letters and spaces only
- Contact Number: exactly 10 digits
- Email ID: valid email format
- Department: valid option required
- Date of Birth: Day, Month and Year required
- Gender: exactly one of Male/Female
- Country/City/State: valid option required
- SSLC Mark: numeric percentage 0–100
- HSC Mark: numeric percentage 0–100
- Photo: must be uploaded
- All fields mandatory; empty submission must produce validation errors

Add Professor
- First Name: letters and spaces only
- Last Name: letters and spaces only
- Phone Number: exactly 10 digits
- Email ID: valid email format
- Department: valid option required
- Date of Birth: valid date required
- Gender: exactly one of Male/Female
- Qualification: at least one of M.E / Ph.D mandatory
- Country/City/State: valid option required
- Photo: must be dragged into the drop box
- All fields mandatory; empty submission must produce validation errors

Treat these rules as the expected result for assertions. The goal is to find where the live application FAILS to enforce them — compare expected (these rules) against actual (observed behaviour) and log every gap.

4. Required Approach

Step 1 — Login and Inspect All Three Modules
Open the application, log in with admin/admin#123, read the dynamic captcha answer from the page, enter it, handle the success alert, and inspect:
- Login fields, error messages and element IDs
- Add Student fields and element IDs
- Add Professor fields and element IDs
- Empty-form behaviour
- Invalid-value behaviour
- Captcha behaviour and success alert
Actually submit empty forms and invalid values. Do not assume behaviour.

Step 2 — Acceptance Criteria
Create acceptance criteria for US-01 through US-21 based on the actual application, fields, validation rules and expected behaviour. Save findings in: Application Analysis & Requirements.txt

Step 3 — Validation Rules
Document all field-level validation rules for all three modules in: Application Analysis & Requirements.txt

Step 4 — Test Case Generation
Generate AT LEAST 60 detailed test cases covering all 21 user stories across all 3 modules.
CSV required columns:
TC_ID, Module, User_Story_ID, Test_Scenario, Test_Type, Priority, Precondition, Test_Steps, Test_Data, Expected_Result, Actual_Result, Status
Priority values: Critical, High, Medium, Low
Required distribution:
- 60–70% must be negative/validation tests
- Negative/validation tests: Critical or High
- Positive/success tests: Low
- Boundary tests: Medium
Use the actual validation rules, element IDs and observed behaviour. Do not invent behaviour. Update Actual_Result and Status after execution where applicable.

Step 5 — Automation Code
Create a complete automation solution for all 3 modules.
Requirements:
- At least 20 automated test methods
- Every automated test must contain meaningful assertions
- Prioritize critical/high-risk validation scenarios
- Page Object Model for Login, Add Student and Add Professor
- A reusable login helper that handles dynamic captcha reading and success-alert handling
- Every module must perform login before navigation
- Reference the actual inspected element IDs and validation rules
- Use the existing workspace structure; do not change it

Step 6 — Execute and Refine
Execute the automation using the integrated VS Code terminal. Do not assume tests pass. Do not stop at the first execution. For every failure:
1. Inspect the actual error.
2. Determine whether it is a locator, navigation, timing, test-data, assertion, application-behaviour, or genuine application-defect issue.
3. Fix automation issues.
4. Correct assertions ONLY when actual application behaviour proves the original assertion was wrong.
5. Re-run failed tests, then re-run the full suite where practical.
6. Improve selectors, stability and coverage.
Do NOT remove assertions or weaken valid validations just to make tests pass. Save files frequently (Ctrl+S). Provide execution evidence where supported.

Step 7 — Defect Detection
Defects must come from actual test execution, not predictions. Document AT LEAST 8 validation defects across the modules in one file: defect_report.json
Each defect must include at minimum: Defect ID, Module, User Story ID, Test Case ID, Test Data / what was entered, Steps/scenario, Expected Result, Actual Result, Severity, Priority, Status.
Execute enough negative/boundary/validation tests to obtain genuine observations. Do not fabricate defects merely to reach eight.

5. Required Deliverables

1. Application Analysis & Requirements.txt — application analysis, login/captcha flow, element IDs, acceptance criteria for US-01 through US-21, validation rules for all 3 modules.
2. Test case CSV — at least 60 cases, all 21 user stories, all 3 modules, required columns, 60–70% negative/validation coverage.
3. Automation codebase — page objects for Login, Add Student and Add Professor; a login helper with captcha + alert handling; at least 20 automated test methods; meaningful assertions.
4. defect_report.json — at least 8 defects across all modules, based only on actual execution, each with expected result, actual result, severity and priority.

Important Constraints
- Do NOT change the provided workspace structure.
- Do not create unnecessary/redundant files or folders.
- Use the VS Code integrated terminal for execution.
- Do not fabricate element IDs, validation behaviour, test results or defects.
- Inspect the application first and base implementation on actual behaviour.
- Use AI assistance, but ensure the analysis, requirements and prompts demonstrate that I am directing the process.
- The final solution must satisfy the assignment's evaluation criteria, not merely produce sample code.
- If the environment genuinely cannot launch a browser, read the captcha, or reach the application, STOP and report this clearly in Application Analysis & Requirements.txt and to me — do NOT invent element IDs, results, or defects to fill the gap.

EXECUTION MODE
Start by inspecting the existing workspace and the live application. Do not immediately generate code: first determine the existing framework, files, structure, selectors/element IDs and actual application behaviour. Then work autonomously through all seven steps in order, without pausing for confirmation between steps, and leave the completed, executed solution in the workspace.


---
---


# 3. College Social Network — Corrected Prompt

*(Already well-structured; only two small safeguards added — a stop-and-report rule and an explicit autonomous-run line.)*

ROLE

Act as a senior QA automation engineer, SDET, test architect, requirements analyst, and debugging specialist working directly inside the current VS Code workspace.

You are completing an AI-assisted UI automation assessment for the College Social Network web application.

Your implementation technology is strictly:
- Playwright
- TypeScript
- Page Object Model
- Reusable fixtures/helpers/utilities
- Playwright assertions
- Integrated VS Code terminal for execution

Your objective is NOT merely to generate code. Your objective is to:
inspect → analyze → document → derive requirements → identify validation rules → generate comprehensive test cases → implement automation → execute tests → investigate failures → fix automation issues → re-run → identify genuine application defects → document actual defects → perform final regression verification.

The final workspace must contain a complete, executable, maintainable solution satisfying every assessment requirement.

You are assisting me with the College Social Network assignment. Follow the assignment exactly and work with the existing workspace.

Analyse the existing project workspace first. Do NOT change, delete, rename, or restructure the provided workspace. Implement the solution on top of the existing structure only. Avoid redundant files/folders.

Use Playwright with TypeScript and Page Object Model. Use the integrated VS Code terminal for installation, execution, debugging, and validation.

The assessment evaluates both:
1. What was analysed/discovered and shared with AI: application analysis, login flow, element IDs, field types, acceptance criteria, and validation rules.
2. How AI prompts were structured: research, generation, code-generation, and refinement/follow-up prompts.

Perform the following steps in order.

APPLICATION UNDER TEST

System: College Social Network
URL: http://webapps.tekstac.com:2121/
Username: admin
Password: admin123

Navigation after login:
- Student menu → Register
- Student menu → Update Backlog
- Faculty menu → Add Faculty

SCOPE
- Module 1 — Student Registration: US-01 to US-08
- Module 2 — Update Backlog: US-09 to US-11
- Module 3 — Add Faculty: US-12 to US-16

Read the user stories/use cases already available in the workspace and treat them as the source of truth.

STEP 1 — LOGIN AND INSPECT ALL THREE MODULES

Login with admin/admin123 and inspect every form.

Identify and document:
- HTML element IDs
- Labels and field types
- Buttons/navigation
- Default values
- Success/error messages
- Form behaviour
- Mandatory fields
- Client/application validation
- Empty-form behaviour
- Invalid-value behaviour

Actually submit empty and invalid data to observe the application. Do not assume behaviour.

STEP 2 — DEFINE ACCEPTANCE CRITERIA

Create acceptance criteria for every user story US-01 through US-16 using the existing user stories plus actual application behaviour.

Create: Application Analysis & Requirements.txt

It MUST contain:
- Application overview
- Login credentials/flow
- Navigation flow
- Module analysis
- US-01 through US-16
- Acceptance criteria
- Element IDs
- Field types
- Actual observed behaviour
- Validation rules
- Success/error messages
- Any discrepancies discovered

Use this file as context for subsequent AI test-case and code generation.

STEP 3 — RESEARCH VALIDATION RULES

Document all field-level validation rules for all three modules in Application Analysis & Requirements.txt.

Required rules:

Student Registration:
- Student Name: letters and spaces only
- Mobile Number: exactly 10 digits
- Email ID: valid email format
- CGPA: 0.0 to 10.0
- Department Name: non-empty text
- Backlog Count: non-negative integer
- All 6 fields mandatory

Update Backlog:
- Roll Number: valid roll number from a completed student registration
- Backlog Count: non-negative integer
- Both fields mandatory

Add Faculty:
- Faculty Name: letters and spaces only
- Email ID: valid email format
- Designation: non-empty text
- All 3 fields mandatory

Also document any additional rules discovered during actual inspection.

STEP 4 — GENERATE COMPREHENSIVE TEST CASES

Generate AT LEAST 60 detailed test cases covering ALL 16 user stories across all 3 modules.

Create a CSV containing exactly:
TC_ID, Module, User_Story_ID, Test_Scenario, Test_Type, Priority, Precondition, Test_Steps, Test_Data, Expected_Result, Actual_Result, Status

Requirements:
- Every US-01 through US-16 must be covered.
- Include positive, negative, validation, boundary, mandatory-field, invalid-format and functional scenarios.
- 60–70% MUST be negative/validation tests.
- Negative/validation tests should be Critical or High priority.
- Positive/success tests should be Low priority.
- Boundary tests should be Medium priority.
- Allowed priorities: Critical, High, Medium, Low.
- Use actual validation rules, element IDs and observed behaviour.
- Do not invent behaviour.
- Update Actual_Result and Status after execution where applicable.

STEP 5 — AUTOMATION IMPLEMENTATION

Create a complete Playwright TypeScript automation solution using Page Object Model.

Required:
- Browser setup/configuration
- Reusable login helper/page
- Student Registration page object
- Update Backlog page object
- Add Faculty page object
- Test methods
- Reusable utilities where needed
- Meaningful assertions
- Reliable Playwright locators
- Proper waits/synchronization

Implement AT LEAST 20 automated test methods covering all 3 modules.

Prioritize:
1. Critical validations
2. High-risk negative tests
3. Boundary tests
4. Positive scenarios

Every module must login using: admin / admin123

Use the actual element IDs and validation rules discovered during inspection. Do not guess selectors.

STEP 6 — EXECUTE AND REFINE

Run the Playwright tests using the integrated VS Code terminal. Do not stop at the first execution.

For every failure:
1. Inspect the actual error.
2. Determine whether it is a locator, navigation, timing, test-data, assertion, application-behaviour, or genuine application-defect issue.
3. Fix automation issues.
4. Correct assertions only when actual application behaviour proves the original assertion was wrong.
5. Re-run failed tests.
6. Re-run the complete suite where practical.
7. Improve stability and coverage.

Do NOT remove assertions or weaken valid validations just to make tests pass.

Provide execution evidence through Playwright reports/artifacts where supported.

STEP 7 — DOCUMENT DEFECTS

Execute the validation tests across ALL THREE modules and document defects based ONLY on actual observed execution.

Create exactly ONE consolidated file: defect_report.json

It must contain AT LEAST 8 documented validation defects.

Each defect must include at minimum:
- Defect ID
- Module
- User Story ID
- Test Case ID
- Test Data / What was entered
- Steps/scenario
- Expected Result
- Actual Result
- Severity
- Priority
- Status

Defects MUST come from actual test execution, not AI prediction or assumptions.

Attempt to identify defects across all three modules. Execute enough negative/boundary/validation tests to obtain genuine observations. Do not fabricate defects merely to reach eight.

FINAL DELIVERABLES

The workspace must contain:

1. Application Analysis & Requirements.txt
   - Analysis of all 3 modules
   - Login flow
   - Element IDs
   - Field types
   - US-01 to US-16
   - Acceptance criteria
   - Validation rules
   - Actual observed behaviour
2. Test case CSV
   - 60+ detailed cases
   - All 16 user stories
   - Required columns
   - 60–70% negative/validation coverage
   - Correct priorities
3. Playwright TypeScript automation codebase
   - POM
   - Login helper
   - Student Registration page
   - Update Backlog page
   - Add Faculty page
   - At least 20 automated tests
   - Meaningful assertions
4. defect_report.json
   - One consolidated JSON file
   - At least 8 actual validation defects
   - Expected result
   - Actual result
   - Severity
   - Priority
   - Execution details

MINIMUM PASSING EXPECTATIONS

Before finishing, verify every item:
- Application Analysis & Requirements.txt exists with documented findings.
- 60+ test cases exist covering all 16 user stories.
- Test coverage is risk-based with 60–70% negative/validation scenarios.
- At least 20 automated tests exist with meaningful assertions.
- Login helper exists.
- Page objects exist for Student Registration, Update Backlog, and Add Faculty.
- All three modules are automated.
- Tests use admin/admin123 login.
- Tests have actually been executed.
- Failed tests have been investigated and refined.
- defect_report.json exists.
- defect_report.json contains at least 8 documented defects.
- Defects are based on actual test execution.
- Every defect includes expected result, actual result, severity, and priority.
- Existing workspace structure remains unchanged.
- No unnecessary/redundant files or folders are created.
- All files are saved and ready for evaluation.

IMPORTANT
- Do not fabricate element IDs, validation behaviour, test results, or defects. If the environment genuinely cannot launch a browser or reach the application, STOP and report this clearly in Application Analysis & Requirements.txt and to me rather than inventing anything.
- Do not merely explain what should be done. Perform the analysis, inspect the application, create the required files, generate the test cases, implement the Playwright TypeScript framework, execute the tests, debug/refine failures, document actual defects, and leave the completed solution in the workspace.
- Work autonomously through all seven steps in order, without pausing for confirmation between steps. Start by inspecting the existing workspace and the live application; do not immediately generate code.
