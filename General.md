# QA Automation & Testing Prompt Sequence

## Prompt 1: [AI Research And Inspection and Applications Analysis of the modules]

You are a senior QA analyst. You need to be inspecting a web app. You will be inspecting `[ ]` number of modules. You need to inspect all the modules and understand the modules, finding the locators, analysing the web content and javascript behaviours of respective modules, and understand the web-component inputs [valid input] [example: phone is asked for input, must be numbers and 10 digits] including name, email, phone number, etc. Also, inspecting and analysing the captcha handling of the module if it's included.

## Prompt 2: [Acceptance Criteria]

Act as a BA. Write acceptance criteria for user stories US-`__` to US-`__` of the `[______]`, grouped by module:
- Module 1: US-`__`–`__`
- Module 2: US-`__`–`__`
- Module 3: US-`__`–`__`

Create acceptance criteria of all `[ ]` number of modules for US-`__` through US-`__` based on the actual application, fields, validation rules, and expected behavior. 

Save findings in: `Application Analysis & Requirements.txt` for all the modules with proper valid fields.

Explain the proper Flow of ALL the given modules, with proper input explanation end to end.

*Note: Continuously use Ctrl+S to save the files after doing any editing.*

## Prompt 3: [Validation Rules]

Using the analysis and acceptance criteria above as context, produce a single consolidated validation-rules reference table for all `__` modules.
Document all field-level validation rules for all `__` modules in: `Application Analysis & Requirements.txt`.
*[If possible, use Gherkin Language]*

## Prompt 4: [Test Case Generation]

Now act as if you are a senior test designer. Generate 45–50 test cases covering ALL given modules and respective US.
Generate at least 50 detailed test cases covering all 20 user stories across all 3 modules. 

**CSV required columns:** 
`TC_ID`, `Module`, `User_Story_ID`, `Test_Scenario`, `Test_Type`, `Priority`, `Precondition`, `Test_Steps`, `Test_Data`, `Expected_Result`, `Actual_Result`, `Status`

**Priority values:** Critical, High, Medium, Low 

**Required distribution:** 
- 60–70% must be negative/validation tests 
- Negative/validation tests should be Critical or High 
- Positive/success tests should be Low 
- Boundary tests should be Medium

Produce a CSV file with the name: `Test Design.csv`

## Prompt 5: [Building Automation Code Framework for all the given modules]

Create a complete automation solution for all `[__]` modules. Provide modules.
 
**Requirements:** 
- At least 20 automated test methods 
- Every automated test must contain meaningful assertions 
- Prioritize critical/high-risk validation scenarios 
- Use Page Object Model for all `[__]` number of modules   
- Create a reusable login helper 
- Login helper must handle dynamic captcha reading and success-alert handling 
- Every module must perform login before navigation 
- Reference the actual inspected element IDs and validation rules 
- Use the existing workspace structure; do not change it 

*Tech Stack:* You may use Playwright, TypeScript, Playwright Assertions, Allure report, and include JSON report, as appropriate for the existing workspace.

*Note: Run the scripts in HEADLESS Mode so that I can see the browser activities.*

## Prompt 6: [Execute and Refine]

Execute the automation using the integrated VS Code terminal. 
Do not simply assume tests pass. Use actual execution results to: 
- Fix failures 
- Correct assertions 
- Improve selectors 
- Improve coverage 
- Refine the automation based on observed application behavior 

*Note: Save files frequently using Ctrl+S.*

## Prompt 7: [Generation: defect report → defect_report.json]

From my ACTUAL test-run results below (not predictions), produce a single consolidated defect report at `defect_report.json`. Document exactly 8 validation defects, at least 2 per module.

**My observed results:** `[PASTE WHAT YOU ACTUALLY SAW: input used, actual behaviour, which test, which module]`

**JSON Format:** An array of defect objects, each with: `id`, `module`, `user_story_id`, `summary`, `input_entered`, `actual_result`, `expected_result`, `severity`, `priority`. 

*Likely real defects to look for:* booking accepts email with no @/domain, phone accepts letters, Departure accepts non-date text, passenger count decrements below zero, enquiry sequential early-return hides later errors, captcha case-sensitivity. 

*Output valid JSON only.*
