#ROLE
You are a senior QA automation engineer / SDET working inside the current VS Code workspace. Complete this UI automation assessment end to end: inspect → analyze → document → write test cases → build automation → run → fix automation issues → record real defects. Do not just generate code.

TECH
Playwright + TypeScript + Page Object Model (or match the stack already in the workspace). Run from the VS Code integrated terminal.

APPLICATION — Train Reservation System (static HTML + vanilla JS, no backend, no persistence)
- Login:          https://webapps.tekstac.com/SeleniumApp1/TrainReservation/login.html
- Ticket Booking: https://webapps.tekstac.com/SeleniumApp1/TrainReservation/index.html
- Enquiry:        https://webapps.tekstac.com/SeleniumApp1/TrainReservation/contactus.html
- Credentials: admin / admin. Login is NOT enforced — index.html and contactus.html open directly; still test login.html on its own.
- Native dialogs (handle as browser dialogs, not DOM): captcha Validate → alert "Please Enter The code"/"Valid input"/"invalid input"; login success → alert "Login Successful" then redirect to index.html; Remember me → confirm() (OK = "Username and Password Saved Successfully!", Cancel = "Changes not saved!", nothing stored).

SCOPE
Login US-01–09 | Ticket Booking US-10–21 | Enquiry US-22–26.

VALIDATION RULES (expected behaviour = assertion oracle)
Login: blank username → "Username cannot be empty", wrong → "Username is wrong"; blank password → "Password cannot be empty", wrong → "Password is wrong"; blank captcha → "Captcha code cannot be empty"; must click Validate and match the 7-char code in #code EXACTLY (case-sensitive) to set secureCheck=true; success needs admin + admin + secureCheck.
Ticket Booking (all 8 fields mandatory; failures concatenated with <br> into div#errfn): blank Travel From/Travel To/Departure/Passenger Name/Email/Phone → "<Field> can't be blank"; Class = default → "DropDown can't be blank"; Passengers = "0" → "Number of Passengers can't be Zero". NOT enforced (tests must expect these ACCEPTED): email format, digit/length phone, date format, passenger bounds, letters-only names. Fare: subtotal = passengers × price (ACSleeper 2500 / Sleeper 1250 / Seating 750); VAT = round(subtotal×0.02); total = subtotal+VAT; recalculated ONLY by the +/- buttons.
Enquiry (sequential, early return — only the first failing field shows a message): name < 3 → "Your name should be at least 3 characters long."; email must contain "." and "@" → "Please enter a valid email address."; message < 15 → "Please write a valid message in few words."; success "Thank you! We will get back to you as soon as possible." auto-clears after ~3s.

STEPS
1. Inspect all 3 live pages AND their JS (login.js / script.js / contact.js) to get the REAL element IDs, field types and error containers (inline divs vs alerts). Submit empty/invalid forms to observe behaviour. Do not guess selectors.
2. Write acceptance criteria for US-01–26, grouped by module.
3. Document all field rules, including the ones NOT enforced. Save Steps 1–3 in "Application Analysis & Requirements.txt".
4. Generate 40–45 test cases in testcases.csv covering all 26 stories. Columns exactly: TC_ID, Module, User_Story_ID, Test_Scenario, Test_Type, Priority, Precondition, Test_Steps, Test_Data, Expected_Result, Actual_Result, Status. ~60–70% negative (Critical/High), boundary Medium, positive Low. Suggested split: Login 13, Booking 19, Enquiry 10.
5. Build the framework: one page object per module (Login, Ticket Booking, Enquiry); a login helper (read #code live, type it, click Validate, accept the alert, click Login); an alert/confirm handler; a "fill all valid except X" helper. ≥25 test methods with assertions (≥7 per module); prioritise negative/validation.
6. Run from the terminal and refine from real results (expect first-run issues around the captcha alert and fare timing). Fix automation issues; don't weaken valid assertions just to pass.
7. Record EXACTLY 8 defects (≥2 per module) in defects/defect_report.json, each with what was entered, what happened, expected result, severity, priority. Defects must come from real runs.

DELIVERABLES
- Application Analysis & Requirements.txt
- testcases.csv (40–45 cases, all 26 stories, exact columns)
- Automation codebase (3 page objects + login helper + alert/confirm handler)
- defects/defect_report.json (exactly 8 defects, ≥2 per module)

RULES
- Don't change the workspace structure; no redundant files.
- Don't fabricate IDs, selectors, behaviour, results or defects — inspect the live app first. If the browser can't reach the app, STOP and say so.
- Only 3 Evaluate attempts: use them at ~50%, ~90%, and final.
- Work autonomously through all 7 steps without pausing for confirmation. Start by inspecting the workspace and the live pages; don't immediately generate code.
