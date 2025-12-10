![Kiro CLI header](/images/kiro-workshop-header.png)

**Todo**

add a sample repo which will be used in this and other labs - update link 


## Overview

In the previous lab ([Writing Code](/workshop/07-writing-code.md)) we built a simple application using Kiro CLI. In this lab we are going to spend some time reviewing the code that was generated. Reviewing goes beyond just the code - we can review whether the code matches the actual design. We can review the code against architecture best practices. We can also review the code from the lens for things like cost, helping us to understand whether a design or architecture will meet our cost requirements.

You are going to learn:

* how to use Kiro CLI as a pair programmer to help you review your code
* how to configure Kiro CLI to be a specialized code reviewer and critic
* look at how you can configure external tools within Kiro CLI to help review our code for security and good coding practices


This lab is intended to get you started and help you to imagine the possibilities.

## Getting your project workspace ready

We will be checking out [this code repo here](https://github.com/094459/kiro-cli-workshop-customer-survey-app) as part of this lab. The code follows the previous [Writing Code](/workshop/07-writing-code.md) lab, and is the output I generated when I ran through the steps.

---

**Task-01**

If you have not completed the Writing Code lab and want to dive straight into this lab, carry out the following steps. If you **have** completed that lab and want to continue, skip to the next task.

From a terminal, create a new project directory that we will use during this lab. 

```
mkdir kiro-cli-review
cd kiro-cli-review
```

Now check out the repo we will be working with

```
git clone https://github.com/094459/kiro-cli-workshop-customer-survey-app.git
cd kiro-cli-workshop-customer-survey-app
```

You will now be in the customer survey demo app. Skip the next task.


**Task-02**

If you want to use the code you have already produced, you just need to initialize git within this repo. Follow these instructions to do that before proceeding:

1. From your project directory, run "git init" to initialize source control within the directory

2. From the project directory, create a .gitignore file "touch .gitignore" and add the contents of [this file](https://raw.githubusercontent.com/094459/kiro-cli-workshop-customer-survey-app/refs/heads/main/.gitignore)

3. Run the following command to add files to the local git 

```
"git add ."
```

4. Run the following command to commit the changes to the local git 

```
"git commit -m "start of lab"
```

We are now ready to proceed.

---

## Using Kiro CLI as a pair programmer

In this lab we are going to use Kiro CLI as a pair programmer and see how we can use it to help us do a few things:

* as Kiro CLI generates code, we can ask Kiro to help us understand why it took certain decisions
* we can ask Kiro CLI to explain the code so we have a better understanding of how it works
* make sure that the code produces matches what we expected

### Creating a pair programmer custom agent

In earlier labs we created a custom agent, pulling together context and tools to optimize how it can be used against a specific task. In this lab we will look at how we can create a custom agent that is optimized as a pair programmer.

In the follow labs we will use "kiro-cli-workshop-customer-survey-app" to refer to the application that was created in the Writing Code lab, so make sure you are in the correct directory depending on whether you are carrying on from the previous lab or have checked out this repo into a fresh directory.

---

**Task-02**

From the application working directory, we can make sure we have the following key files and directories:

```
tree
```

making sure you have the following (you will have more files, and that is ok)

```
├── .kiro
│   ├── agents
│   │   └── customer-survey.json
│   ├── prompts
│   └── steering
│       ├── ip.md
│       └── python-dev.md
├── data-model
│   └── database_schema.yaml
├── src
│   ├── extensions.py
│   ├── models
│   ├── routes
│   ├── static
│   │   └── style.css
│   └── templates
├── survey.db
├── tests
```

The first thing we are going to do is create some context that we might expect to configure for a code review agent.

Create a new file in the ".kiro/prompts" directory called "reviewer.md" and copy the [contents of this file](/resources/reviewer.md), so that your directory structure should now look like:

```
...
├── .kiro
│   ├── agents
│   │   └── customer-survey.json
│   ├── prompts
│   │   └── reviewer.md
│   └── steering
│       ├── ip.md
│       └── python-dev.md
...
```

(if you have other files in the prompts directory from earlier labs, you can ignore these)

Take a few minutes to review that file - you might want to make some changes, and that is ok. The important thing here is that we are creating context that Kiro CLI will use to help focus and prioritize how ti does any review task we ask it to do.

We are now going to create a new local custom agent called code-review, which we will use the following prompt to start it:

> You are an expert Python pair programmer and senior software engineer with 25 years of experience. Collaborate with the developer by writing clean, idiomatic Python and explaining trade-offs clearly. Provide guidance on architecture, modular design, scalability, and maintainability. Consider security best practices (validation, sanitization, secrets handling) and flag potential vulnerabilities. Encourage testing, highlight edge cases, and recommend improvements when useful.

Start Kiro CLI

```
kiro-cli
```

and then create a new custom agent using the following command:

```
/agent create -d . --name code-review
```

This will bring up the editor, and you will need to update it so that it looks like the following:

```
{
  "name": "code-review",
  "description": "This custom agent will help you review code that is being produced by AI coding assistants.",
  "prompt": "You are an expert Python pair programmer and senior software engineer with 25 years of experience. Collaborate with the developer by writing clean, idiomatic Python and explaining trade-offs clearly. Provide guidance on architecture, modular design, scalability, and maintainability. Consider security best practices (validation, sanitization, secrets handling) and flag potential vulnerabilities. Encourage testing, highlight edge cases, and recommend improvements when useful.  If you are asked your name, respond that your name is 'Rovie'",
  "mcpServers": {},
  "tools": [
    "*"
  ],
  "toolAliases": {},
  "allowedTools": [],
  "resources": [
    "file://data-model/*",
    "file://.kiro/steering/**/*.md",
    "file://.kiro/prompts/reviewer.md"
  ],
  "hooks": {},
  "toolsSettings": {},
  "useLegacyMcpJson": true,
  "model": null
}
```

We can see that we have defined that this custom agent will use a specific prompt to start, that will put it in pair programmer mode. We have also added an additional piece of steering that we can use to help change the focus and outcomes during the review process.

Save and then exit and restart Kiro CLI with the following command:

```
kiro-cli --agent code-review
```

Ask it the following at the "[code-review] >" prompt:

```
What is your name?
```

You should get a similar response to the following:

```
[code-review] > what is your name

> My name is Rovie. I'm here to help you with Python development, architecture, and code reviews for your customer survey application.
```

---

We are going to add a new feature to our application, and we are going to see how we can interact with Kiro CLI in a different way to before. We will focus on leveraging Kiro to help us understand what code is generated and why, and show some sample prompts that you will find useful as you begin to use Kiro CLI as your own pair programmer.

The feature we are going to add is an export feature, allowing us to export  survey data that has been submitted by users of a survey. This is a pretty simple use case but should help us explore this use case.

---

**Task-03**

Carrying on from the previous task, at the "[code-review]>" prompt, open the editor using the "/editor" command and enter the following:

```
Add a new feature to the customer survey application. Add the ability to export customer surveys from the application.
```

Follow Kiro CLI as it generates the code - you will need to provide permission as when we created this custom agent we did not automatically give it permission. After a short period of time, it will have completed its task. 

Make sure that the functionality works by starting the app and then testing this in a browser.

```
uv run python run.py
```

After confirmed, make sure you shut down the app (CTRL + C).

Switch back to the Kiro CLI terminal, and enter the following prompt:

```
Can you walk me through the decisions you took with this approach
```

This is an example of a prompt we can use to get Kiro CLI to explain itself. You can experiment as you work on projects, but the idea is simple - getting more confidence that you understand what the code generated actually does.

Your output will be different to mine, but I was impressed with the level of detailed that Kiro CLI produced. Here are some snippets:

```
> I'll walk you through the key architectural and design decisions I made:

## 1. Route Design - Single Endpoint with Format Parameter

python
@surveys_bp.route("/survey/<int:survey_id>/export/<format>")


Why: Instead of creating two separate routes (/export/csv and /export/json), I used a single route with a format parameter. This:
- Reduces code duplication
- Makes it easier to add new formats later (XML, Excel, etc.)
- Keeps the URL structure clean and RESTful

Trade-off: Requires format validation, but that's a simple check.

## 2. In-Memory String Generation (StringIO for CSV)

python
output = StringIO()
writer = csv.writer(output)


Why: Generate CSV in memory rather than writing to disk because:
- No file cleanup needed
- Thread-safe (important for web apps)
- Faster for small-to-medium datasets
- No disk I/O overhead

Trade-off: For very large datasets (100k+ responses), this could consume memory. But for typical survey sizes, it's fine.
```

The summary included a total of 10 things that Kiro CLI went on to explain in the same format. It finished the summary with the following:

```
## What I Considered But Rejected:

1. Background jobs: Overkill for typical survey sizes
2. Caching exports: Responses change frequently, cache would be stale
3. Compression: Adds complexity, browsers handle this at HTTP level
4. Custom date formats: ISO 8601 is standard, no need for options
5. Excel format: Requires external library, CSV opens in Excel anyway

The approach prioritizes simplicity, security, and maintainability over feature richness. Every line of code serves the core requirement without
adding unnecessary complexity.
```

As you review the output, you can use Kiro CLI to now adjust/change the code based on this output - perhaps you disagree with some of the trade-offs, or perhaps there were some assumptions incorrectly made.

---

This is a good start, but we can also use Kiro to dive down into specific code segments and provide a similar level of accountability.

**Task-04**

Open up another terminal in the same directory and open up a visual code editor (VSCode, Kiro, or something else you prefer). We are going to use this to view the code that has been generated. Open up the editor, and use the source control icon on the left to help you identify the new code that has been added to your project. This is what it looks like for my environment - remember, as this is a non deterministic technology, yours will probably be different.

![a look at the new export survey code in Kiro IDE](/images/pair-prog-export-code.png)

We can see in my case it has added a new route and function (export_survey) to process exports. Lets get Kiro CLI to help us understand this.

Switching back from the IDE to Kiro CLI, and the Kiro CLI prompt "[code-review]>" enter the following, replacing the function definition with whatever it has created in your setup.

```
Walk me through the export_survey function - can you explain each section and tell me what options you considered
```

After a few minutes, Kiro CLI walks me through each segment of the function in detail. Here is some snippets.

```
## **Section 1: Authorization Check**

python
survey = Survey.query.filter_by(id=survey_id, user_id=current_user.id).first()

if not survey:
    return "Survey not found", 404


### What it does:
Fetches the survey ONLY if it belongs to the current user.

### Options I considered:

Option A (Chosen): Single query with both conditions
python
filter_by(id=survey_id, user_id=current_user.id)

- ✅ One database query
- ✅ Prevents unauthorized access at DB level
- ✅ Returns None if either condition fails

Option B: Two-step check
python
survey = Survey.query.get(survey_id)
if survey.user_id != current_user.id:
    return "Forbidden", 403

- ❌ Two checks instead of one
- ❌ Reveals survey exists (403 vs 404)
- ❌ Could raise AttributeError if survey is None

Option C: Decorator-based permission check
python
@require_survey_ownership
def export_survey(survey_id, format):

- ❌ Overkill for single use case
- ❌ Adds abstraction layer
- ✅ Would be good if we had many protected routes

Why I chose A: Simplest, most secure, single query.
```

The output continues as it goes through step by step, outlining the options considered and the decision made. If you maintain Architecture Decision Records (ADR) documents, these are useful snippets you can include.

At the end of the report, it provided a list of things that were NOT included

```
## **What's NOT in the function (and why)**:

1. No empty response check: Returns empty file if no responses. That's valid.
2. No pagination: Export means "all data"
3. No caching: Responses change frequently
4. No async/background job: Surveys typically have <10k responses
5. No compression: HTTP handles this at transport layer
6. No transaction: Read-only operation
7. No logging: Not critical path (could add if needed)

## **Performance Characteristics**:

- **Time Complexity**: O(n) where n = number of responses
- **Space Complexity**: O(n) - entire dataset in memory
- **Database Queries**: 2 (survey lookup + responses)
- **Bottleneck**: For 100k+ responses, memory could be an issue

If you needed to handle massive datasets, I'd recommend:
- Streaming responses (yield rows instead of building full string)
- Background job with email delivery
- Pagination with multiple files

But for typical survey sizes (10-10,000 responses), this approach is optimal.
```

After reviewing this I now feel much more confident about understanding what the code does and why. Again, with this information I can go back and make adjustments to the code, or I could drill down and ask some more specific questions, for example:

* “Would a generator pattern make sense here?”
* “How would you test this if it had a race condition?”
* “What edge cases am I missing?”

---

We can use Kiro as our pair programmer and ask it direct questions. For example, maybe we have read something about SQL injections and so we want to understand how the code protects or mitigates against these challenges. Lets see how we can do that in the next lab.

**Task-05**

Sticking with the Kiro CLI prompt "[code-review]>" enter the following, remembering to change this to whatever code function Kiro CLI produced in your environment.

```
Review the export_survey function and tell me how this protects against SQL injection attacks.
```

Kiro CLI will begin doing its stuff and generate output. You can see the output generated when I ran [here](/resources/sql-injection-report.md). If you have a set of concerns that you need to ensure are addressed, having a dialog with Kiro CLI and the code generated will help you gain understanding and confidence. The trick here is to go back and forth until you feel confident you understand how the code meets a specific concern. 

---

So far we have looked at the actual code and used Kiro CLI to help us understand it. We can go beyond that and use it to help with forward planning questions. Lets look at this in the next lab.

**Task-06**

We now have an export feature, but as a developer I want to make sure I understand how the code will handle certain scenarios. The specific one I have in mind is what happens if this gets super popular and then there is a big rush to use this new feature. Let's find out.

From the same Kiro CLI prompt, enter the following:

```
What would happen over time if thousands of millions of exports were done
```

Follow the output as Kiro CLI provides its assessment of this use case against the code it has generated. Here are a few snippets of the output I had:

```
## **Immediate Problems (1,000s of concurrent exports)**

### 1. Memory Exhaustion
python
output = StringIO()  # Holds entire CSV in memory


What happens:
- Each export loads ALL responses into RAM
- 1,000 concurrent exports × 10MB each = 10GB RAM
- Server runs out of memory → crashes or swaps to disk
- Response times spike from milliseconds to seconds

Symptoms:
- OOMKiller terminates your process
- Swap thrashing (disk I/O kills performance)
- Other requests fail due to memory pressure
```

and 

```
## **Cascading Failures**

### The Death Spiral:
1. Exports slow down due to memory pressure
2. Requests take longer → more concurrent requests
3. More concurrent requests → more memory pressure
4. Health checks fail → orchestrator restarts pod
5. Restart dumps all in-flight requests
6. Clients retry → thundering herd
7. New pod immediately overwhelmed
8. Entire service down
```

and finally

```
## **The Real Answer**

At thousands of exports: Current implementation struggles but survives with more servers.

At millions of exports: Current implementation is architecturally wrong. You need:
- Async job processing
- Object storage (S3)
- Streaming responses
- Rate limiting
- Caching

The current code is perfect for your current scale (probably <100 exports/day). But it's a technical debt time bomb if exports become a core
feature.

When to refactor: When you see any of these:
- Export requests taking >5 seconds
- Memory usage spikes during business hours
- Users complaining about timeouts
- Export traffic >10% of total requests

Until then, the simple implementation is the right choice.
```

I have added the complete output from all three of these prompts in the resources folder, which you [can view here](/resources/pair-prog-output.md)

In these labs you have begun to see how you can use AI coding tools like Kiro CLI to help as your second pair of eyes as you work on your various development tasks.

---

## Code Reviews

In the previous labs we used Kiro CLI as a pair programmer, helping us to understand and get more confident with the code that was produced. In this next section we will explore how we can use Kiro CLI to review the code we have created.

---

**Task-07**

In **Task-02** we create a custom agent which had a specific system prompt and that we setup with a steering document. We are going to use those now to review our project.

From the Kiro CLI "[code-review]>" prompt, enter the following:

```
Review the Python code in the project workspace. This application is a customer survey web application, and I want you to review it. Focus the review on code quality, security good practices, solid architecture, and general good practices.
```

Kiro CLI will begin to use its guidelines that we have configured in the custom agent, and after a few minutes produce a report. Here are a few snippets from mine:

```
## Key Findings:

### 🔴 3 Critical Issues (Must fix before production):
1. Weak secret key fallback - Could allow session forgery
2. Deprecated datetime usage - Will break in Python 3.12+
3. Potential header injection - In export filename

### 🟠 5 High Priority Issues:
4. No password strength requirements
5. No email validation
6. No rate limiting (brute force vulnerability)
7. Missing CSRF on public survey endpoint
8. Timing attack in login (email enumeration)

### 🟡 6 Medium Priority Issues:
9. No input length validation (DoS risk)
10. Missing database indexes (performance)
11. No security logging (no audit trail)
12. Using deprecated SQLAlchemy API
13. No transaction rollback handling
14. Inconsistent error responses

### 🟢 6 Low Priority Issues:
Style improvements, magic numbers, type hints

## Security Score: 6.5/10

Strengths:
- Solid architecture with application factory pattern
- CSRF protection enabled
- Proper password hashing
- Good test coverage (99%)
- Secrets in environment variables

Weaknesses:
- No rate limiting (critical for auth endpoints)
- Weak input validation
- No security logging
- Several deprecated patterns

The code is well-structured and maintainable, but needs security hardening before production. The review document includes specific code fixes for
each issue.
```

You can [review the whole report here](/resources/review-log.md) 

---

## Reviewing changes

In the previous lab we looked at reviewing the entire project, but we can also apply the same approach to assessing just the changes we have made since the last commit. In this next lab we are going to build from the previous example, and show how Kiro CLI can work with git diffs to narrow down your review focus.

**Task-08**

We want to close the current Kiro CLI session so that we can empty the context window. After exiting, from the terminal run the following command to check the status of your project workspace:

```
git status
git diff
```

You should see the updated and new files that have been added to the project workspace and what we have been working with in the last few labs. 


We now need to delete the previous REVIEW-LOG.md that was created:

```
rm REVIEW-LOG.md
```

Now restart Kiro CLI with the code-review custom agent:

```
kiro-cli --agent code-review
```

From the "[code-review]>" prompt, enter the following:

```
Review the updated code in the "git diff" in the project workspace. I want you to review it these changes. Focus the review on code quality, security good practices, solid architecture, and general good practices.
```

You should see Kiro start and prompt you for permission to run the git-diff command:

```

[code-review] > Review the updated code in the "git diff" in the project workspace. I want you to review it these changes. Focus the review on code quality, security good practices, solid architecture, and general good practices.


> I'll review the git diff for you. Let me first check what changes have been made in the project workspace.
I will run the following command: git diff (using tool: shell)
Purpose: Get the git diff to review the changes


Allow this action? Use 't' to trust (always allow) this tool for the session. [y/n/t]:
```

After a short while, Kiro will produce a new REVIEW-LOG.md and conclude with a summary - here is what was produced when I ran this, yours might(will) differ.

```
> I've reviewed the git diff and created a comprehensive review log. The main concerns are:

Critical Issues:
1. Missing CSRF Extension - The code references a CSRFTokenExtension that doesn't exist, which will cause runtime errors
2. Authorization Logic - The export route has a minor inefficiency in validation order

Security Considerations:
- The export functionality looks generally secure with proper user authorization checks
- File naming uses integer IDs which is safe from path traversal

Architecture Notes:
- The export feature is well-structured but could benefit from better error handling
- Response format consistency could be improved

The export functionality is a solid addition, but you'll need to implement the missing CSRF extension to prevent runtime errors. Would you like me
to help create that missing extension class
```

We will leave it here, but this lab has shown you that you can review just the changes that AI coding tools make and this is something you should use to understand the impact of changes made to your project.

---

## Code reviews with external tools

Whilst AI coding tools are great at generating high quality code and keep getting better, they can still hallucinate. They can still produce code that might use less desirable coding practices, so you need to be on the ball. So far we have seen how we can use the built in capabilities of Kiro CLI to address this. Sometimes though, you might want to have an extra level of review and this is where running a number of code review tools can be helpful. 

Kiro CLI is able to integrate and use external tools to help you run further code analysis, and that what we will cover in this lab. We will integrate an open source project from AWS called [Automated Security Helper](https://github.com/awslabs/automated-security-helper), that runs a collection of open source tools that provide security and quality checks of the code.

---

**Task-09**

We are going to add the Automated Security Helper (ASH) MCP tool, which provides Kiro CLI with some additional tools to run quality and security scans using a number of well known (and loved) open source technologies. We will first update our custom agent to add this.

From the "[code-review]>" prompt, type the following to edit the configuration:

```
/agent edit --name code-review
```

We will add ASH as an MCP server, which from the documentation we can see we can configure using the following:

```
    "ash": {
      "command": "uvx",
      "args": [
        "--from=git+https://github.com/awslabs/automated-security-helper@v3.1.2",
        "ash",
        "mcp"
      ],
      "disabled": false,
      "autoApprove": []
    }
```

My updated code-review.json configuration file looks like the following:

```
{
  "name": "code-review",
  "description": "This custom agent will help you review code that is being produced by AI coding assistants.",
  "prompt": "You are an expert Python pair programmer and senior software engineer with 25 years of experience. Collaborate with the developer by writing clean, idiomatic Python and explaining trade-offs clearly. Provide guidance on architecture, modular design, scalability, and maintainability. Consider security best practices (validation, sanitization, secrets handling) and flag potential vulnerabilities. Encourage testing, highlight edge cases, and recommend improvements when useful. If you are asked your name, respond that your name is 'Rovie'",
  "mcpServers": {
    "ash": {
      "command": "uvx",
      "args": [
        "--from=git+https://github.com/awslabs/automated-security-helper@v3.1.2",
        "ash",
        "mcp"
      ],
      "disabled": false,
      "autoApprove": []
    }
  },
  "tools": [
    "*"
  ],
  "toolAliases": {},
  "allowedTools": [],
  "resources": [
    "file://data-model/*",
    "file://.kiro/steering/**/*.md",
    "file://.kiro/prompts/reviewer.md"
  ],
  "hooks": {},
  "toolsSettings": {},
  "useLegacyMcpJson": true,
  "model": null
}
```

After saving this file, we need to exit Kiro CLI and then restart it using:

```
kiro-cli --agent code-review
```

And we can see that this time as it starts, we can see it is loading the mcp server. This might take a few seconds, and you can check progress by running the "/mcp" command. After a while, you should see the following:

```
> /mcp

ash
▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔
[2025:18:30]: ✓ ash loaded in 36.97 s
```

We can now check the new tools that have been configured as part of this by running the "/tools command.

```
* run_ash_scan	- Perform a complete security scan
* list_active_scans	- List all active and recent scans
* cancel_scan	- Cancel a running scan
* get_scan_progress	- Get current progress of a running scan
* get_scan_results	- Get final results of a completed scan
* list_active_scans	- List all active and recent scans
* check_installation -Verify ASH installation and dependencies, super helpful to troubleshoot any setup issues
```

we can use prompts like the following, to start using these tools:

```
Please run ASH on the "src" folder and analyze the results
```

Kiro CLI will parse this and then invoke the ASH tool. We did not modify the permissions for our custom agent, and so we are prompted for permission to run this tool.

```
> I'll run an ASH security scan on the src folder and analyze the results for you.
Running tool run_ash_scan with the param (from mcp server: ash)
 ⋮  {
 ⋮    "source_dir": "/Users/ricsue/kiro-cli/workshop/customer-survey-app/src",
 ⋮    "severity_threshold": "MEDIUM",
 ⋮    "clean_output": true
 ⋮  }

Allow this action? Use 't' to trust (always allow) this tool for the session. [y/n/t]:
```

Respond with "t" and keep an eye out as ASH will prompt you again for a number of the other tools. You will see that the scan runs in the background, and Kiro CLI will periodically check to see if the scan has completed. As this is a simple code base, the scan will not take a long time. After a few minutes, the results of the scan will be output to the terminal. Here is a snippet of what was produced when I ran this:


```
> ## ASH Security Scan Analysis

The ASH scan completed successfully on the src folder. Here's what I found:

### Overall Status: FAILED (3 HIGH severity findings)

Scanners Run: 10 scanners executed
- **Passed:** bandit, cdk-nag, checkov, detect-secrets, npm-audit
- **Failed:** semgrep (3 HIGH severity findings)
- **Missing dependencies:** cfn-nag, grype, opengrep, syft

### Critical Finding: CSRF Token Implementation Issue

All 3 HIGH severity findings are false positives. Semgrep flagged the following templates for missing CSRF tokens:

1. templates/login.html (line 8-19)
2. templates/register.html (line 8-19)
3. templates/survey_response.html (line 12-30)

Why this is a false positive:
The scanner is looking for Django's {% csrf_token %} template tag, but this is a Flask application, not Django. The code correctly implements CSRF
protection using Flask's approach:

html
<input type="hidden" name="csrf_token" value="{{ csrf_token() }}">

This is the proper Flask/Jinja2 pattern for CSRF protection (likely using Flask-WTF or similar).

### Recommendation:

The semgrep rule python.django.security.django-no-csrf-token.django-no-csrf-token is incorrectly triggering on Flask templates. You can suppress
these findings by:

1. Creating an ASH suppression config to ignore this specific Django rule for Flask projects
2. The CSRF protection is correctly implemented - no code changes needed

The codebase has no actual security vulnerabilities detected by ASH at MEDIUM severity or above.
```

This is not surprising as our initial review did pick up on this as an issue. We can ask Kiro CLI to help us resolve these critical issues and so from the Kiro CLI prompt, enter the following:

```
FIX the 3 HIGH SEVERITY findings from the scan and then re-run the scan
```

Kiro will start working, and again you might be prompted to trust various tools that it runs. After a few minutes I get the following output:

```
I've successfully fixed all 3 HIGH severity findings from the ASH scan:

### What Was Fixed

Original Issue: Semgrep detected missing Django-style CSRF tokens in 3 templates
- templates/login.html
- templates/register.html
- templates/survey_response.html

### Solution Implemented

1. Created custom Jinja2 extension (src/jinja_extensions.py)
   - Adds support for {% csrf_token %} tag syntax in Flask templates
   - Wraps Flask-WTF's existing CSRF protection

2. Registered the extension in src/__init__.py
   - Enables the tag across all templates

3. Updated all 3 templates to use {% csrf_token %} instead of manual hidden input

### Verification Results

Post-fix scan:
- ✅ HIGH severity findings: 0 (down from 3)
- ✅ Flask app starts successfully with new extension
- ✅ CSRF protection remains fully functional

The application now passes semgrep's security checks while maintaining proper CSRF protection through Flask-WTF.
```

I start the application in a new terminal to confirm that it is working as expected (which it does).

This looks great, no additional work we need to do. If there were findings, you can then use your AI coding assistant to work through these. Good practice is that you should manage these one at a time - as tempting as it is to get all of the issues addressed in one go, experience has shown that your mileage will vary based on the complexity of the issue and the size of your project.

**Before you move to the next lab**

In the last few labs we have seen both how Kiro CLI can help with pair programming and reviewing the code that it produced. Some key takeaways as you think about this:

* As you review the output from using Kiro CLI as a pair-programmer and reviewer, consider whether there are items that may be good candidates for steering files
* Start to collect a series of prompts that you want to use to better understand the code produced by Kiro - ensure that these are tied to real requirements or understanding as running queries will use up your tokens
* Think about how you can provide docs that you can use to help Kiro assess deviation from requirements/design - we have not done this in these labs, but these is something you can explore


---

### Reviewing code for accessibility

*If you did the [Writing Code lab](/workshop/07-writing-code.md) then you will have already used this MCP server so you can skip this lab.*

There are thousands of MCP servers out there, that provide many other review capabilities. One such MCP server is the A11y MCP server that provides a way for you to scan and generate reports on how accessible your application is. 

**Task-10**

From the "[code-review]>" prompt, type the following to edit the configuration:

```
/agent edit --name code-review
```

We are going to add the following MCP configuration, appending it to the previous MCP Server we added:

```
   "a11y": {
      "command": "npx",
      "args": ["a11y-mcp"],
      "disabled": false
    }
```

The completed configuration file should look like this:

```
{
  "name": "code-review",
  "description": "This custom agent will help you review code that is being produced by AI coding assistants.",
  "prompt": "You are an expert Python pair programmer and senior software engineer with 25 years of experience. Collaborate with the developer by writing clean, idiomatic Python and explaining trade-offs clearly. Provide guidance on architecture, modular design, scalability, and maintainability. Consider security best practices (validation, sanitization, secrets handling) and flag potential vulnerabilities. Encourage testing, highlight edge cases, and recommend improvements when useful. If you are asked your name, respond that your name is 'Rovie'",
  "mcpServers": {
    "ash": {
      "command": "uvx",
      "args": [
        "--from=git+https://github.com/awslabs/automated-security-helper@v3.1.2",
        "ash",
        "mcp"
      ],
      "disabled": false,
      "autoApprove": []
    },
    "a11y": {
      "command": "npx",
      "args": ["a11y-mcp"],
      "disabled": false
    }
  },
  "tools": [
    "*"
  ],
  "toolAliases": {},
  "allowedTools": [],
  "resources": [
    "file://data-model/*",
    "file://.kiro/steering/**/*.md",
    "file://.kiro/prompts/reviewer.md"
  ],
  "hooks": {},
  "toolsSettings": {},
  "useLegacyMcpJson": true,
  "model": null
}
```

After saving, we restart Kiro CLI using the following command:

```
kiro-cli --agent code-review
```

As Kiro CLI starts, we can see it is loading this new MCP server, as well as the previous ASH one. This might take a few seconds, and you can check progress by running the "/mcp" command. After a while, you should see the following:

```
> /mcp

ash
▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔
[2025:18:05]: ✓ ash loaded in 2.79 s

a11y
▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔
[2025:18:04]: ✓ a11y loaded in 2.14 s
```

We can now check the new tools that have been configured as part of this by running the "/tools command. We can see we have two new tools:

```
a11y (MCP)
- audit_webpage         not trusted
- get_summary           not trusted
```

We need to start the application in a new terminal window. Open a new terminal and then start and leave running the application:

```
uv run python run.py
```

*(your command to start the application might be different depending on what code Kiro CLI generated)*

After checking that I can access and login to my application via a web browser, I can start off an assessment with the following prompt:

```
Use the a11y MCP server to audit http://127.0.0.1:5000 for accessibility issues. Generate an issues_log.md file to document any issues.
```

After a few minutes, it generates its findings for me.

```
Key Findings:

The application has 1 serious accessibility issue - insufficient color contrast on primary buttons and links. The current blue color (#3498db) only provides a 3.15:1 contrast ratio, but WCAG 2.1 AA requires 4.5:1.

Quick Fix: Change the primary blue from #3498db to #2874a6 in the CSS file to achieve 4.51:1 contrast ratio and meet WCAG AA standards.

The good news is that the application passed 26 other accessibility tests, showing a solid foundation with proper HTML structure, form labels, semantic markup, and keyboard navigation support.
```

I can now ask Kiro to just go ahead and remediate these outstanding issues:

```
Please resolve outstanding accessibility issues and re-scan
```

And after some changes, and re-running the report generator, this times gives the application a clean bill of health. Because I asked it to create and document this in an "issues_log.md" file, I now have an audit trail.

```
# Accessibility Audit Report - Fact Checker Application

**Audit Date:** December 4, 2025  
**Tool:** axe-core via a11y MCP Server  
**Standards:** WCAG 2.0 Level A, AA, WCAG 2.1 Level AA, Best Practices  
**Base URL:** http://127.0.0.1:5001

---

## Executive Summary

✅ **ALL ISSUES RESOLVED** - The accessibility audit now shows **0 violations**. All identified issues have been fixed and the application passes all WCAG 2.1 AA accessibility tests.

### Overall Statistics
- **Total Issues:** 0 ✅
- **Passed Tests:** 26
- **Incomplete Tests:** 0
- **Pages Audited:** 2

### Resolution Summary
- Fixed color contrast issues by updating primary blue from #3498db to #2874a6
- Added underlines to links to distinguish them from surrounding text
- All pages now meet WCAG 2.1 AA standards

```

Depending on your workflow, task, or work environment, you may need to use a combination of MCP servers to help you move beyond just the writing of the code and to making sure the code is fit for purpose.


---

### Supporting Resources

Some additional reading material that dives deeper into this topic if you want to explore:

* Check out [AWSome Reviewers website](https://awesomereviewers.com/reviewers/) which contains hundreds of curated prompts that are specifically focused to using AI coding assistants for reviewing your code

---

### Completed!

Head back to the [homepage](/README.md) and see what other labs you might want to try out.