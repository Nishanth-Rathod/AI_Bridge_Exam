# API Automation â€” Single Prompt (Tekstac AI-Assisted Assessment)

Paste the prompt once and let it run end-to-end. Do **not** send it step by step.

**Fill first (one line):** API name / base URL `___` Â· modules & US ranges `___` Â· auth type `___` Â· counts: cases/methods/defects `___` (defaults 60 / 20 / 8, â‰¥2 defects per module) Â· files: `testcases.csv`, `defects/defect_report.json`

**Evaluate strategy:** run Tekstac Evaluate once after step 3â€“4 (~50%), again after step 6 (~90%), once at the end. Only 3 attempts.

**If it starts guessing endpoints:** reply `re-run step 1 against the live API and show me the actual responses` before letting it continue.

---

## The Prompt

```
You are a senior SDET completing an AI-assisted API automation assessment in this VS Code workspace. Work autonomously end-to-end â€” do NOT wait for my confirmation between steps. Build on the existing workspace; never rename or restructure it; no redundant files. Use the framework already in the workspace; if it's an empty scaffold, use Playwright + TypeScript with a Service Object Model (one client class per module). Run everything from the integrated terminal. Save often (Ctrl+S). Do not fabricate endpoints, behaviour, or defects â€” discover everything.

1. DISCOVER (no guessing): Read every user-story / OpenAPI / Swagger / Postman / README file in the workspace (user stories = source of truth). Then probe the LIVE API with a throwaway script (temp folder, delete after): for each endpoint send valid, missing-field, wrong-type, no-token, and bad-token requests; record method, URL, request, response status, headers, body. Resolve auth for real: call the login/token endpoint, capture the token, confirm which header carries it and that a protected endpoint fails without it. Record per endpoint: method/path/params, required vs optional fields + types, success status + schema, error status + schema, which validations are actually ENFORCED vs NOT (email format, phone digits/length, numeric/positive, date, length, enum), auth requirement, and whether one user can access another's resource.

2. DOCUMENT â†’ "Application Analysis & Requirements.txt": per-module endpoint map with field/element IDs, the auth flow, Given/When/Then acceptance criteria for every user story, and a field-level validation + status-code table that explicitly marks the rules the API does NOT enforce.

3. TEST CASES â†’ "testcases.csv", columns EXACTLY: TC_ID, Module, User_Story_ID, Test_Scenario, Test_Type, Priority, Precondition, Test_Steps, Test_Data, Expected_Result, Actual_Result, Status. Use the required count (default 60+), cover every user story, 60â€“70% negative/validation at Critical/High, boundary at Medium, positive at Low. Include auth, authorization, status-code, schema, and "non-enforced rule accepts bad input" cases. Test_Steps = endpoint+method, Test_Data = payload/headers, Expected_Result = status + body/schema. Leave Actual_Result/Status blank. RFC 4180 clean.

4. CODE: One service/client per module using the REAL endpoints/payloads from step 1. Helpers: auth helper (login â†’ token â†’ attach header â†’ handle 401), request builder, schema validator (ajv/zod or equivalent), build-valid-payload-except-X from one valid fixture. Config with base URL + JSON & HTML reporters + request/response logging on failure. Implement the required number of test methods (default â‰¥20, sensible per-module split), each tagged with its TC_ID, each asserting status + body + schema + key headers, prioritising Critical/High and auth cases over happy paths. No hard sleeps.

5. RUN + SELF-HEAL: Run the full suite. For each failure, re-send the request to diagnose, then classify â€” (A) automation fault (wrong URL/payload/auth/assertion/schema) â†’ fix and re-run until green; (B) genuine API defect (accepts bad input, wrong/missing status or error, 500 on bad body, reachable without token, cross-user access) â†’ do NOT weaken the assertion, keep it failing, save request+response as evidence. Loop until every remaining failure is a confirmed defect and everything else passes. Give a short total/passed/failed summary.

6. DEFECTS â†’ "defects/defect_report.json" (valid JSON array), exactly the required count (default â‰¥8, â‰¥2 per module), each from a test that actually reproduced it. Fields: id, module, user_story_id, related_tc_id, endpoint, method, summary, request, actual_result (status+body), expected_result (status+body/schema), severity, priority, evidence (test name + request/response path), status. If short, run more targeted negative/auth/boundary tests â€” don't invent defects. Prime suspects: 200/201 for invalid input, email with no @ or TLD accepted, phone accepts letters/any length, negative/zero amount, mandatory field not enforced, endpoint reachable without token, one user editing another's resource, 200 instead of 404, 500 on malformed JSON.

Then stop and tell me what to review.
```
