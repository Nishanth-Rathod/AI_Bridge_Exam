# Tekstac API Automation â€” Fixed Prompt Sequence (Playwright + TypeScript + Supertest + Jest + Axios)

A 7-prompt sequence for the Tekstac AI-Augmented QE **API-automation** assessments, built on the same scored skeleton as the UI assessments (Train Reservation, ParaBank, SmartUniversity, College Social Network). It is adaptive about the API itself: Prompt 1 makes the agent discover the endpoints, base URL, auth flow, user-story ranges, and the real validation behaviour from the workspace spec and the live API, so you do not need the exact question in advance.

**The stack is fixed** â€” it does not adapt to the workspace. Build this stack on top of the existing workspace structure; never rename or restructure it.

## Stack (fixed) and tool roles

- **TypeScript** â€” the language (transpiled by ts-jest).
- **Jest** â€” the single test runner and assertion library (`describe` / `it` / `expect`). Do not add a second runner.
- **Axios** â€” the HTTP client. One configured instance (baseURL, timeout) with interceptors that attach the auth token and log every request/response to an evidence folder; the service/client classes are built on it.
- **Supertest** â€” fluent endpoint assertions, e.g. `request(baseURL).post(path).set(headers).send(body).expect(status)`; it targets the live base URL as a string.
- **Playwright** â€” used **only** via `APIRequestContext` (`request.newContext`) to capture HAR/trace evidence and as a backup client. **Not** a test runner â€” do not introduce `@playwright/test`'s `test()`.
- **Reports** â€” Jest's built-in results plus `--json` output for a machine-readable summary, `jest-html-reporter` for HTML, and optionally `jest-junit` for JUnit XML. Install the extra reporters only if the network allows; do not block on them.

## How to use

Fill STEP 0 (FACTS) from the question on your screen â€” just the numbers and the auth type that carry weightage. Leave anything blank and the agent will read it from the workspace in Prompt 1.

Send Prompt 1 through Prompt 7 in order, one at a time, in your AI chat. Let each finish before sending the next. Copy the plain text under each Prompt heading.

Run Tekstac Evaluate at about 50 percent (after Prompt 4 to 5), about 90 percent (after Prompt 6), and once at the end (after Prompt 7). You get only 3 attempts.

The assessment grades two things: what you analysed and shared with AI, and how you directed your prompts. Each step below is a distinct research, generation, code, or refinement prompt so this is visible.

## Notes

There is no DOM and no browser here. "Inspection" means hitting endpoints and reading the actual status codes, response bodies, headers, and schemas. Evidence is the saved request/response pairs (written by the Axios interceptor) plus the HAR captured via Playwright, the HTML report, and the terminal logs.

Deliverable names must match the assessment exactly. The UI exams use `testcases.csv` and `defects/defect_report.json` â€” assume the same unless your exam states otherwise, in which case put the real names in STEP 0.

If the service is **SOAP/XML** rather than REST/JSON, keep the same seven steps and the same stack but swap JSON-schema (ajv) validation for XML/XSD validation, send XML bodies through Axios/Supertest, and assert on SOAP `<Fault>` elements instead of HTTP status codes. Everything below assumes REST/JSON.

---

## STEP 0 â€” FACTS (type these from the on-screen question; blanks are fine)

API / system name: ______

Modules (resource groups, count and names): ______

Base URL (and environment, if given): ______

User-story range(s): US-__ to US-__ (per-module split if shown)

Required test-case count: ______ (UI exams use 40 to 45, or 60+)

Required automated test methods: ______ (UI exams use at least 20, or at least 25)

Required defect count: ______ (UI exams use exactly 8 with at least 2 per module, or 5)

Auth type: ______ (for example Bearer / JWT, API-key header, Basic, OAuth2 client-credentials, session cookie, none) and login/token endpoint if known: ______

Spec / contract present: ______ (OpenAPI or Swagger URL or file, Postman collection, WSDL, README â€” or none)

Test-case file name: testcases.csv

Defect file path: defects/defect_report.json

Stack: FIXED â€” Playwright + TypeScript + Supertest + Jest + Axios (do not change)

The agent will confirm and complete all of the facts above from the workspace and the live API in Prompt 1.

---

## Prompt 1 â€” Research, Inspect and Analyze (no code yet)

You are a senior QA automation engineer and SDET working inside this VS Code workspace. Do not write automation or test code yet.

First, read every requirement, user-story, and contract file already in this workspace â€” for example User Stories.txt, any OpenAPI or Swagger file, any Postman collection, any README or assessment file. From them, list the API name, each module or resource group, every endpoint it exposes with its HTTP method and path, and its user-story range. Treat the workspace user stories as the source of truth. The implementation stack is fixed â€” TypeScript with Jest as the runner, Axios as the HTTP client, Supertest for endpoint assertions, and Playwright's APIRequestContext for evidence â€” so inspect the existing workspace structure (package.json, tsconfig.json, existing folders) and plan to build this stack on top of it exactly, without restructuring; note what is already installed versus what you will need to add.

Then inspect the live API yourself. Write one throwaway script, using plain Axios or node and saved under a temp or scratch folder and deleted when done, that for every endpoint sends a representative set of requests â€” a valid request, an invalid one with a missing required field, one with a wrong data type, one with no authentication, and one with a bad or expired token â€” and prints for each: the method, the full URL, the request headers and body, the response status code, the response headers (especially Content-Type), and the full response body. Resolve the authentication flow end to end in the same script: call the login or token endpoint, capture the token, show exactly which header carries it and in what format, then prove a protected endpoint succeeds with that token and fails without it. Run the script from the integrated terminal and read its output.

From the actual responses, not from guesses, record for each module and endpoint: the method, path, and any path or query parameters; the required versus optional request-body fields and their data types; the exact success status code and the success response schema; the exact error status code and the error response schema or message shape; which validations the server actually enforces versus those documented but not enforced, for example email format, digit or length phone, numeric or positive number, date format, string length, and enum membership; the auth requirement per endpoint, including whether a protected endpoint wrongly returns 200 without a token; the authorization behaviour, for example whether one user can read or modify another id's resource; any pagination, filtering, or sorting parameters; and any rate-limit or throttling signals.

Give me a concise findings summary grouped by module and endpoint. Do not fabricate anything. If any endpoint or spec will not load, say exactly what failed and stop.

---

## Prompt 2 â€” Acceptance Criteria

Act as a Business Analyst. Using your Prompt 1 findings and the workspace user stories as the source of truth, write Given/When/Then acceptance criteria for every user story across all modules, grouped by module and traceable to the validation rules and the expected status codes.

For each module also write a short end-to-end flow â€” what a valid journey looks like, request by request, through to the success state. For example: authenticate and receive a token; POST the resource and receive 201 with an id; GET that id and receive 200 with the expected schema; PUT an update and receive 200; DELETE and receive 204; GET the id again and receive 404.

Save all of this to a file named exactly `Application Analysis & Requirements.txt`, creating it if missing and appending if it exists, with clear per-module sections. Press Ctrl+S.

---

## Prompt 3 â€” Validation Rules

Using the analysis and acceptance criteria above as context, produce one consolidated validation-rules table covering every field in every request payload across every module, plus the response-level expectations. For each field give the rule, the expected error status code and error message, and whether the API actually enforces it â€” explicitly mark the rules it does not enforce, for example email format, digit or length phone, numeric or positive values, and date format. Add a status-code matrix: for each endpoint, which status code it should return for a valid request, a validation failure, a missing or bad token, an unauthorized resource, a not-found id, and a malformed body. Express the key rules in Gherkin where it helps.

Append this as a Validation Rules section to `Application Analysis & Requirements.txt`. Press Ctrl+S.

---

## Prompt 4 â€” Test Case Design

Act as a senior test designer. Generate the number of test cases the assessment requires (use the stated count, for example 40 to 45, or 60+) covering all user stories across all modules. Save them to a CSV named exactly as the assessment requires (default `testcases.csv`).

Columns, in this exact order: TC_ID, Module, User_Story_ID, Test_Scenario, Test_Type, Priority, Precondition, Test_Steps, Test_Data, Expected_Result, Actual_Result, Status.

For API cases, Test_Steps should name the endpoint and method, Test_Data should hold the request payload and any relevant headers, and Expected_Result should state the expected status code plus the body or schema expectation.

Rules: every user story must appear at least once; 60 to 70 percent must be negative or validation cases at Critical or High priority; boundary and format-gap cases at Medium, such as a value exactly at a minimum length, a count decremented below zero, an enum or dependent value changed after a related field is set, an alphabetic phone, an email with no at-sign or no TLD, a missing-token attempt, and an expired-token attempt; positive and success cases at Low; include cases that assert the non-enforced rules accept bad input; include explicit authentication, authorization, status-code, and schema-validation cases; leave Actual_Result and Status blank until execution; make the CSV RFC 4180 clean by wrapping any field that contains a comma or line break in double quotes and doubling any internal quotes. Press Ctrl+S.

---

## Prompt 5 â€” Automation Framework

Build a complete API-automation solution using the fixed stack â€” TypeScript, Jest (runner and assertions), Axios (HTTP client), Supertest (endpoint assertions), and Playwright's APIRequestContext (evidence) â€” on top of the existing workspace structure. Do not rename, move, or restructure anything already there, and do not create redundant files, and do not add a second test runner.

Install only what is missing: typescript, ts-jest, @types/jest, jest, supertest, @types/supertest, axios, ajv (or zod) for schema validation, @playwright/test, and optionally jest-html-reporter and jest-junit for reports â€” skip the extra reporters if they will not install without network.

Config: a `jest.config.ts` using the ts-jest preset, a sensible testTimeout, and the reporters (default plus jest-html-reporter for an HTML report, plus jest-junit for a JUnit XML) when they install. Put the base URL from STEP 0 or the spec in one config or env module.

HTTP client and Service Object Model: create one configured Axios instance (baseURL, timeout) with a request interceptor that attaches the auth header and a response-plus-error interceptor that writes every request/response pair â€” method, URL, headers, body, status â€” to an evidence folder, so each result carries proof. Build one service or client class per module or resource on top of this Axios instance, using the real endpoints, paths, and payloads you found in Prompt 1, not guesses.

Helpers: an auth helper that calls the login or token endpoint via Axios, caches the token, feeds it to the interceptor, and handles a 401 by refreshing or failing clearly; a request builder; a schema validator built on ajv (or zod) whose result you assert with Jest `expect`; and a build-valid-payload-except-X data helper driven by one valid-data fixture.

Assertion styles: use Supertest for the fluent endpoint tests â€” `request(baseURL).<method>(path).set(headers).send(body).expect(status)` then assert the body and schema â€” and use the Axios service classes for flows that chain calls (create, read, update, delete) with Jest `expect` on the status, body, schema, and key headers. Capture a representative HAR via Playwright's `APIRequestContext` (`request.newContext({ baseURL })`) for evidence; do not use the Playwright test runner.

Auth logic: if the endpoints require a token, authenticate first via the helper; if an endpoint is public, call it directly, but still cover the auth or login module's own user stories.

Implement the exact number of automated test methods the assessment requires (each a Jest `test` or `it`), with a sensible per-module split, each tagged with its TC_ID in the test title, each with meaningful assertions on the status code, the response body, the schema, and the key headers â€” prioritising Critical and High validations and the authentication and authorization cases over happy paths. Use no hard sleeps â€” poll or retry for any asynchronous endpoint, and assert any transient response immediately after the call. Press Ctrl+S.

---

## Prompt 6 â€” Execute, Self-Heal and Classify Failures

Run the full suite from the integrated terminal with `npx jest`. Do not assume anything passes â€” read the actual results. Work autonomously: do not ask me what to do, keep going until the suite is stable.

For every failing test, first re-inspect the relevant live endpoint by re-sending the request (via Axios or your throwaway probe) and dumping the status, headers, and body, diagnose the exact root cause, then classify the failure into one of two buckets.

Bucket A, the failure is caused by the automation: a wrong URL, path, or parameter, a wrong payload or headers, a missing or wrong auth step, a schema that does not match, a wrong assertion that does not match the API's correct behaviour, or an environment issue. Fix the script and re-run that test. Repeat the inspect, fix, re-run loop until every Bucket A test passes. Every automation-caused failure must end up passing.

Bucket B, the test is correct but the API genuinely misbehaves: it accepts input it should reject, returns the wrong status code or the wrong error or no error, miscalculates, leaks data, is reachable without a token, lets one user touch another's resource, returns 500 on a malformed body, and so on. This is a real defect and a success for your testing, not something to hide. Do not weaken, delete, or flip the assertion to make it green. Confirm the misbehaviour by re-running and by direct inspection, keep the failing test and its evidence (the saved request and response, and the HAR), and note it for the defect report.

Keep looping until there are no unexplained failures left: every remaining failure is a confirmed Bucket B API defect, and everything else passes. Then give me a short run summary â€” total, passed, failed â€” and list which failures are confirmed real API defects, with a one-line reason for each. Press Ctrl+S.

---

## Prompt 7 â€” Defect Report

From the actual results of the run you just executed, not from predictions, produce one consolidated defect report at exactly `defects/defect_report.json`. Document exactly the number of defects the assessment requires, for example 8 with at least 2 per module, or 5, each taken from a test that actually reproduced it.

Each defect object has these fields: id, module, user_story_id, related_tc_id, endpoint, method, summary, request (payload plus relevant headers), actual_result (status code plus body), expected_result (status code plus body or schema), severity, priority, evidence (test name plus the saved request/response or HAR path), and status. Output valid JSON only, as an array of these objects.

If you do not yet have enough confirmed real failures to reach the required count, design and run more targeted negative, boundary, and auth tests first to surface additional genuine defects â€” do not invent defects to reach the number. Likely real API defects to look for: a 200 or 201 returned for invalid input instead of 400 or 422; an email accepted with no at-sign or no TLD; a phone that accepts letters or any length; a numeric or date field that accepts non-numeric or non-date text; a negative or zero amount accepted; a count decremented below zero; a mandatory field not actually enforced; an endpoint reachable without a token (missing authentication); one user able to read or modify another user's resource (broken authorization); the wrong status on a missing id, such as 200 instead of 404; duplicate resource creation allowed with no 409; a 500 on a malformed JSON body; an inconsistent or absent error schema; and a sensitive field leaked in the response, such as a password hash.

Press Ctrl+S, then run Tekstac Evaluate.

---

## Final checklist (have the agent confirm before you submit)

`Application Analysis & Requirements.txt` exists with per-module endpoint analysis (method, path, parameters), the full auth flow, the request and response schemas, acceptance criteria, and the validation plus status-code table.

The test CSV has the exact name, the exact columns, the required count, every user story, 60 to 70 percent negative at the right priorities, and explicit authentication, authorization, status-code, and schema cases.

The stack is exactly Jest (runner and assertions) plus Axios (client and evidence logging) plus Supertest (endpoint assertions) plus Playwright `APIRequestContext` (HAR evidence); a `jest.config.ts` runs the suite; one service or client class per module plus the auth, schema, and payload helpers; the required number of Jest test methods; meaningful assertions on status, body, schema, and headers; the last full run green except for the confirmed real-defect tests.

`defects/defect_report.json` is valid JSON with the exact required defect count and per-module minimum, every item traceable to an executed request, with the saved request/response or HAR as evidence.

Workspace structure unchanged; no redundant files; everything saved with Ctrl+S.
