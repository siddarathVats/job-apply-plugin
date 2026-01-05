# Job Application Assistant

A Claude Code skill for filling job applications on LinkedIn Easy Apply, Greenhouse, and Workday using browser automation.

---

## Initial Prompt

When this skill is invoked, first check if a profile exists at `~/.claude-job-profile.json`.

**If NO profile exists**, say:

> Welcome to the Job Application Assistant! I'll help you fill out job applications on LinkedIn, Greenhouse, and Workday.
>
> First, I need to set up your profile. This is a one-time process — your information will be saved for future applications.
>
> **Please provide the path to your resume file** (PDF, DOCX, or TXT).
>
> For example: `~/Documents/resume.pdf` or `/Users/you/Desktop/MyResume.pdf`

Then wait for the user to provide the path before proceeding with profile extraction.

**If a profile DOES exist**, say:

> Welcome back! Your profile is loaded from `~/.claude-job-profile.json`.
>
> **Provide a job URL** and I'll help you apply. For example:
> - LinkedIn: `https://www.linkedin.com/jobs/view/123456789`
> - Greenhouse: `https://boards.greenhouse.io/company/jobs/123`
> - Workday: `https://company.wd5.myworkdayjobs.com/jobs/job/123`
>
> Or say **"reset profile"** if you want to update your information from a new resume.

---

## Required Input

- **Resume file path**: Path to your resume (PDF, DOCX, or TXT format)
- **Job URL**: LinkedIn job posting or direct application link

## Profile Storage

Your extracted profile is stored at `~/.claude-job-profile.json` for reuse across sessions. Run with `--reset-profile` to re-extract from resume.

---

## Workflow

### Phase 1: Profile Setup

If no profile exists at `~/.claude-job-profile.json`, or if the user requests a reset:

1. **Read the resume file** using the Read tool
2. **Extract structured data** into these categories:
   - `firstName`, `lastName`
   - `email`, `phone`
   - `location` (city, state, country, zip)
   - `linkedInUrl`, `portfolioUrl`, `githubUrl` (if present)
   - `workHistory[]`: array of { company, title, startDate, endDate, current, description }
   - `education[]`: array of { school, degree, field, startDate, endDate, gpa }
   - `skills[]`: array of skill strings
3. **Present extracted data to user** for review and correction
4. **Save confirmed profile** to `~/.claude-job-profile.json`

### Phase 2: Application Filling

1. **Load profile** from `~/.claude-job-profile.json`
2. **Navigate to job URL** using Chrome MCP
3. **Detect platform type**:
   - LinkedIn Easy Apply: Look for "Easy Apply" button or modal
   - Greenhouse: URL contains `boards.greenhouse.io` or `jobs.lever.co`
   - Workday: URL contains `myworkdayjobs.com` or `wd5.myworkdaycdn.com`
4. **Read the form structure** using `read_page` tool
5. **Map and fill fields** (see Platform-Specific Guidance below)
6. **Handle multi-page forms**: Click "Next" or "Continue" between sections
7. **Take screenshot** of completed form
8. **Present summary** to user showing all filled values
9. **Wait for explicit confirmation** before clicking Submit/Apply

---

## Platform-Specific Guidance

### LinkedIn Easy Apply

**Characteristics:**
- Modal-based multi-step wizard
- Usually 2-5 steps: Contact Info → Resume → Additional Questions → Review
- Has progress indicator at top

**Approach:**
1. Click "Easy Apply" button to open modal
2. Use `read_page` on each step to identify fields
3. Common fields:
   - Phone number (often pre-filled from LinkedIn)
   - Resume upload (use `upload_image` tool with resume path)
   - Work authorization questions (dropdowns)
   - Custom screening questions (varies by employer)
4. Click "Next" to advance, "Review" on final step
5. Screenshot the review page before "Submit application"

**Field patterns to look for:**
- `input[name*="phone"]` - Phone number
- `input[type="file"]` - Resume upload
- `select`, `[role="listbox"]` - Dropdown questions
- `[role="radio"]`, `[role="checkbox"]` - Multiple choice

### Greenhouse

**Characteristics:**
- Single long-form page with sections
- Clear field labels
- Often has "Add another" for work history/education

**Approach:**
1. Scroll to read full form structure first
2. Fill from top to bottom
3. For work history sections:
   - Fill most recent position
   - Click "Add another" if form allows and user has more history
4. Education section similar pattern
5. Handle custom questions at bottom
6. Single "Submit Application" button at end

**Field patterns:**
- Standard `<input>` and `<select>` elements
- `#first_name`, `#last_name`, `#email`, `#phone` common IDs
- `.field-container` or `.field` wrapping each question

### Workday

**Characteristics:**
- Multi-page wizard with heavy JavaScript
- Non-standard UI components (custom dropdowns, date pickers)
- Often requires account creation (STOP and inform user - do not create accounts)

**Approach:**
1. If login/account creation required, STOP and inform user they must handle this
2. Navigate through "My Information" → "My Experience" → "Application Questions"
3. For dropdowns: Click to open, then use `find` to locate option, then click
4. For date fields: May need to click calendar icon, then select date
5. Use `computer` tool with clicks for non-standard components
6. Watch for "Save and Continue" vs "Submit" buttons

**Special handling:**
- Workday dropdowns: Click field → wait → read options with `read_page` → click option
- Date pickers: Often format-sensitive, try MM/DD/YYYY
- Required fields marked with asterisk or red border after validation

---

## Field Mapping Reference

| Profile Field | Common Form Labels |
|--------------|-------------------|
| firstName | First Name, Given Name, First |
| lastName | Last Name, Family Name, Surname, Last |
| email | Email, Email Address, E-mail |
| phone | Phone, Phone Number, Mobile, Cell |
| location.city | City |
| location.state | State, Province, State/Province |
| location.zip | Zip, Postal Code, ZIP Code |
| location.country | Country |
| linkedInUrl | LinkedIn, LinkedIn URL, LinkedIn Profile |
| workHistory[0].company | Current Company, Most Recent Employer, Company |
| workHistory[0].title | Current Title, Job Title, Position, Title |
| education[0].school | School, University, College, Institution |
| education[0].degree | Degree, Degree Type |
| education[0].field | Major, Field of Study, Concentration |

---

## Chrome MCP Tool Usage

### Reading Forms
```
Use read_page with filter: "interactive" to see all form fields
Use find with natural language: "email input field", "submit button"
```

### Filling Fields
```
Use form_input for standard inputs and selects:
  - ref: element reference from read_page
  - value: the value to set

Use computer with action: "left_click" for custom dropdowns:
  1. Click to open dropdown
  2. read_page to see options
  3. Click the correct option
```

### File Upload (Resume)
```
Use find to locate file input: "resume upload field"
Use form_input or computer to trigger file picker
Note: May need to use upload_image tool for some implementations
```

### Multi-Page Navigation
```
After filling current page:
1. find "next button" or "continue button"
2. Take screenshot for record
3. Click next
4. read_page on new step
5. Repeat until review/submit page
```

---

## Safety Rules

1. **Never enter passwords** - If login required, stop and instruct user
2. **Never create accounts** - Stop and inform user they must create accounts themselves
3. **Never click Submit without confirmation** - Always screenshot and get explicit "yes"
4. **Never enter payment information** - Some applications have optional premium features
5. **Handle sensitive questions carefully** - Salary expectations, visa status, disability disclosure should be confirmed with user before filling

---

## Example Invocation

```
User: Help me apply to this job: https://www.linkedin.com/jobs/view/123456789