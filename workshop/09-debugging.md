![Kiro CLI header](/images/kiro-workshop-header.png)

# Debugging and Troubleshooting with Kiro CLI

Did you know that according to [getdx.com](https://getdx.com/guide/ai-assisted-engineering/)(signup required), debugging issues with code is the top use case of AI coding assistants. In this lab we will look at how to use Kiro CLI to debug and troubleshoot issues.

## Overview

This lab is theory heavy, but work through it - this will give you a good foundation to help you with your debugging efforts. The labs will help you implement some of these in the customer survey application that this workshop has been working with (which you can build from the [Writing Code](/workshop/07-writing-code.md) part of this workshop. If you have not done that, you can also check out a version of this in GitHub [here](https://github.com/094459/kiro-customer-survey)).

Using tools like Kiro CLI to debug and troubleshoot comes down to:

* Describing the problem with the right level of precision and detail
* Providing supporting information (context)

In the following sections we will explore both. 

**Fixings vs Helping**

Before you proceed, there is one final thing to consider. When using tools like Kiro CLI to debug and troubleshoot, think about whether you want it to fix the problem, or whether what you really need is help you understand what the issue is. AI coding assistants can do both, but in my experience I have seen better results pivoting from asking "Fix it!", to asking it, "What is causing this issue?" and then getting additional information that can help me understand how plausible a solution might be (and then going ahead and fixing it). 

## 1. Define the problem

In the same way that we have learned how to structure context, break down problems into smaller tasks, and use AI coding assistants to write code, we need to learn how to take a structured approach when Kiro CLI to debug and troubleshoot. The first step is to understand the actual issue or problem. This will direct you so that you can provide the right context and information, which is essential to make sure that Kiro has the right information to work with.

When you are debugging issues with code (either new code or changes to existing), you need to provide the right framing. For example: 

* Describe what happens to generate the error
* When did the issue start?
* What changed right before things broke?
* What was the expected vs. actual behavior - “Expected JSON, got HTML.”
* Is it affecting one user or many?
* Is the issue in the code or something external that it depends on?
* Ask for reasoning - “What could cause this?” instead of “Fix it.”

This will help you to craft a detailed prompt that will be more effective in troubleshooting.

---

We are going to create a custom agent that will help us be excellent code debuggers. This custom agent will help guide us through the troubleshooting workflow.

We will stick with the customer survey application that you created in the [Writing Code](/workshop/07-writing-code.md) workshop. If you have skipped that they don't worry as you can use a pre-prepared version.

**Task-01**

If you have been working with the customer survey application and have it open within a terminal or Kiro CLI session then fine. If not, then from a new terminal window, create a project directory and check out the following repo:

```
mkdir kiro-cli-debugger
cd kiro-cli-debugger
git clone https://github.com/094459/kiro-customer-survey.git
cd customer-survey
```

We are going to create a new shared prompt which our custom agent will use ([debugger.md](/resources/debugger.md)) so we need to copy this to the ".kiro/shared" directory. Sticking with the same terminal that we just cloned the previous repo, lets copy the prompt using the following command:

```
cd .kiro/shared
wget https://raw.githubusercontent.com/094459/aqd-cli-workshop/refs/heads/main/resources/debugger.md
cd ../..
```

Review the file. This is going to be our starting system prompt for our debugger, so make sure you look at the structure and how this is out together so that when we use this later you will be able to understand how you can adjust this for your own development workflow.

From this terminal session start Kiro using

```
kiro-cli
```

From the ">" prompt that appears, we will generate a new custom agent using the wizard

```
/agent generate
```

Use the following responses:

- Name = code-debugger
- Description = You are an expert code debugger
- Agent Scope = Local

```
> /agent generate

✔ Enter agent name:  · code-debugger
✔ Enter agent description:  · You are an expert code debugger
✔ Agent scope · Local (current workspace)
```

After the editor comes up, edit the file so that it looks like the following:

```
{
  "name": "code-debugger",
  "description": "You are an expert code debugger",
  "prompt": "file://../shared/debugger.md",
  "mcpServers": {},
  "tools": [
    "*"
  ],
  "toolAliases": {},
  "allowedTools": [],
  "resources": [
  ],
  "hooks": {},
  "toolsSettings": {}
}
```

After saving, we need to exit Kiro. After exiting to the terminal, restart Kiro using:

```
kiro-cli --agent code-debugger
```

Once you have the "[code-debugger]>" prompt you are ready. We will come back to this later.

---

### Understanding your codebase

Your effectiveness at debugging code within a project is related to your understanding of that code. In an earlier lab, we looked at how we could use Kiro CLI to make sense of the code within a project workspace ([Reviewing Code](/workshop/12-reviewing-code.md) and [Documenting Codebases](/workshop/06-understanding.md)).

Before you proceed with debugging, you should:

* make sure that you understand the code within your project, using the techniques outlined in the previous workshops
* review the documentation within the project workspace and make sure it is accurate and up to date - use Kiro CLI to help you update/check
* if no documentation exists, use the techniques within the previous workshops to generate these and then use the review process to help you gain an understanding of the project

I cannot tell you how important having a solid understanding of your project and code base is when debugging and troubleshooting.

Whilst it is important that you know about your code and project, we can also provide Kiro with some help. The [Code Intelligence](https://kiro.dev/docs/cli/code-intelligence/?trk=71546b8e-c969-4ead-aa9f-9cd06f6d8610&sc_channel=el) feature integrates Language Server Protocol (LSP) into Kiro CLI to enable semantic understanding of your codebase for the Kiro agent. This can be a useful tool that will help Kiro as it works in troubleshooting.

---

**Task-02**

We are going to configure the Code Intelligence feature. If you have done previous labs, including [Writing Code](/workshop/07-writing-code.md) then you will have already done this and so can skip Task-02. This task is only required if you cloned the project fresh from source.


Exit your current Kiro CLI session, and from the terminal run the following command:

```
pip install pyright
```

With that installed, from the terminal start Kiro CLI session with:

```
kiro-cli --agent code-debugger
```

and at the "[code-debugger]>" prompt enter:

```
/code --help
```

This will show you the list of available options with this feature. We need to initialize the current project workspace in order for Kiro to be able to make use of it, so we do that using:

```
/code init
```

which should generate the following output:

```
✓ Workspace initialized

Workspace: /{project workspace}/kiro-cli/kiro-customer-survey
Detected Languages: ["python"]
Project Markers: ["pyproject.toml"]

Available LSPs:
○ clangd (cpp) - available
○ gopls (go) - not installed
○ jdtls (java) - not installed
✓ pyright (python) - initialized (1.3s)
○ rust-analyzer (rust) - available
○ solargraph (ruby) - not installed
○ typescript-language-server (typescript) - not installed

Configuration can be updated at .kiro/settings/lsp.json
Learn more at https://kiro.dev/docs/cli/code-intelligence/


[code-debugger] λ >
```

You will also notice that the prompt has changed and now has a "λ" appear - this is a visual cue that your current project space has an LSP configured.

You can check the status of the LSP by running the following command:

```
[code-debugger] λ > /code status

✓ Workspace initialized

Workspace: /{project workspace}/kiro-cli/kiro-customer-survey
Detected Languages: ["python"]
Project Markers: ["pyproject.toml"]

Available LSPs:
○ clangd (cpp) - available
○ gopls (go) - not installed
○ jdtls (java) - not installed
✓ pyright (python) - initialized (1.5s)
○ rust-analyzer (rust) - available
○ solargraph (ruby) - not installed
○ typescript-language-server (typescript) - not installed

Configuration can be updated at .kiro/settings/lsp.json
Learn more at https://kiro.dev/docs/cli/code-intelligence/
```

Initializing code within our project has made a new tool is made available called "code", which you can see when using "/tools"

```
[code-debugger] λ > /tools


Tool              Permission
▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔
Built-in
- code            trusted
- shell           not trusted
- read            trust working directory
- write           not trusted
- glob            trust working directory
- grep            trust working directory
- introspect      trusted
- report          not trusted
- aws             trust read-only commands
- subagent        not trusted
- web_fetch       not trusted
- web_search      not trusted
```

Your Kiro CLI session is now ready for any debugging and troubleshooting it needs to do. Well ok, there are still a few things we need to do to so lets look at those now.

---

## 2. Prompt design

Prompt design matters. When trying to fix errors that come up, it is critical to structure the prompt so that it provides the right direction for Kiro CLI. I have put together a short primer on [good prompt practices](/workshop/14-good-prompts.md) which you can read and get a refresher.

### Iterative refinement

I have found that I re-use certain prompts/phrases that seem to work well. If the first response isn’t great, refine your prompt and try again. It is important that you spend time finding out and understanding what prompts and phrases work for you - something that you will gain an intuition for over time. 

### Be specific

When crafting your prompt, it is important you provide the right level of detail, clarity, and specificity. Some examples of being specific include:

- What is happening when the error occurs - the biggest failure from using Kiro is to provide only part of the information needed to help provide a good solution. For example, just providing a stack trace and asking for Kiro to fix the issue. You should always provide situational awareness when asking for a fix. For example, "when I do x in the web browser, I expect y, but get z and this is an error/stack trace that is produced". The same applies for non user issues, for example "when the system x is running, y event happens, and this stack error is produced".
- Provide details about changes you made - If you made changes, then add the change you made as context. Be specific: what files did you change and why. Add the previous state if you can (this is where providing git diff as context can be helpful).
- What was the expected behavior - Make sure you add what the expected behavior is supposed to be as context. I often see this as a miss.

### Leverage examples

Examples are a one of the best ways to guide AI coding assistants when we want them to create things, but it is also a great way to help them tune into specifics. For example, consider these examples:

- Bad Prompt: "Fix this broken function."
- Good Prompt: "Here is a broken function generateReport(). It's not working correctly, and generates a Type Error when a user exports a report from the application (Homepage > Company > Filter > Generate Report). For reference, generateExport(), which is on the same page works and provides similar functionality. Please compare the two and explain the error in generateReport()."

As a developer you are using your knowledge and critical thinking to help guide and narrow down where the LLM needs to look.

### Guiding thoughts

Chaining Thoughts: For complex analyses, ask the AI to “Think step-by-step before answering.” (e.g., “First, identify the error type. Second, pinpoint the line. Third, suggest a fix.”)

### Supporting context

Finally prompts need to be supported by additional information about your code base (context). As you craft your prompt, be thinking about what supporting context you will need to provide - do you have it already or do you need think about how to generate it? The following sections will provide ideas on how you might start collecting that information which you can than package up as context.

---

## 3. Debug tasks one at a time

When you have several bugs that might come at you at once, the temptation is to just ask Kiro to "fix the bugs". Whilst this might be effective in small, less complex issues, your results will vary as size and complexity increase. The trick is to use your skill as a developer and identify discrete issues and then debug those one at a time.

A good way to manage this is by creating an "issue-log.md" file and then using this to drive your debugging activities. I have provided an example [issue-log.md](/resources/issue-log.md) that I use. I create the initial file in a suitable location, and add this to the context of the debugging session I have started. During testing, I add issues to this doc. I then use this to drive the troubleshooting and debugging efforts (one at a time).

It is important to time box debugging activity. If you are spending more than 30 mins on an issue, you should pause and take a step back. If you have already tried the other techniques in this doc, then you might want to use checkpoints or git to revert code changes (if debugging is down to changes you have mode).

---

We are going to build upon the custom agent we created, and amend it so that we add the issue-log.md as context.

**Task-03**

Exit Kiro CLI and from the terminal, run the following commands

```
wget https://raw.githubusercontent.com/094459/aqd-cli-workshop/refs/heads/main/resources/issue-log.md
```

This should copy the [issue-log.md](/resources/issue-log.md) into the root directory of the customer survey application. Next we are going to edit our custom agent to look for this file. In previous labs we have used the "/agent edit --name {custom-agent-name}" to do this. This time we will edit the custom agent configuration file directly, which lives in the ".kiro/agents" directory.

Feel free to use a different editor if you want, other wise stick with the king of editors, vim.

```
vi .kiro/agents/code-debugger.json
```

And update the configuration so that it looks like the following:

```
{
  "name": "code-debugger",
  "description": "You are an expert code debugger",
  "prompt": "file://../shared/debugger.md",
  "mcpServers": {},
  "tools": [
    "*"
  ],
  "toolAliases": {},
  "allowedTools": [],
  "resources": ["file://issue-log.md"],
  "hooks": {},
  "toolsSettings": {}
}
```

Save and exit this file, and then restart Kiro CLI using:

```
kiro-cli --agent code-debugger
```

You can use "/context show" to see that your issue-log.md file is now in the context.

---

## 4. Adding instrumentation

How many times have you used 'print to console.log' to help debug and issue? How many times has a try/catch block helped you narrow down where to look for a problem? Making sure that your code has a basic level of instrumentation, and follows good practices around exception management with in the code is going to help you when issues arise. This additional level of information is great context for Kiro CLI and will help get you to a faster resolution.

### Logging

In the past when we used the most verbose levels of logging, it would be a struggle to find the relevant information to help us debug issues. AI coding assistants though love all that data, and are great pattern matchers and detectives. As your application grows or becomes part of a broader system, you can integrate OpenTelemetry and then access the data to help with troubleshooting and debugging.

When you are debugging issues, you should think about how you can use Kiro CLI to add additional logging around code blocks to help you gather more information. 

### Exception handling

Review your code for how you current manage exceptions. You can codify this using your project steering documents so that when your AI coding assistant is generating code these are automatically added. Alternatively, you can use Kiro to add these after the fact.

### Building tools

Another technique that can work well is get Kiro CLI to build tools to help it troubleshoot and debug. You can add details such as:

```
During debugging, try MAX three times. If the problems still persist, BUILD TOOLS that will help you gather more information to help you identify the problem and suggest a fix.
```

You might actually see this in action when Kiro is writing code, where it writes a temporary test function or class to validate some code and then once working, deletes that test function. This is an extension of that approach, and can work really well for some problems. 

---

In this lab we are going to add a logging framework to the code base, and create it so that it will offer different levels of instrumentation which we can configure.  

**Task-04**

We are going to switch to the original customer-survey custom agent. From your Kiro CLI session, use the following command:

```
/agent switch customer-survey
```

From the "[customer-survey]>" prompt, using /editor to enter the following prompt:

```
Add a logging framework to this application. It should:

- instrument every class and function
- allow TRACE, DEBUG, ERROR, WARN, and INFO levels of detail for logging
- allow logging configuration to be done via a configuration property
- log to a separate file (app.log)
```

Follow Kiro-CLI and as it asks for permission, review and work through the implementation of the new logging. From a different terminal:

* review updates to the documentation to see how the logging framework has been implemented
* configure DEBUG logging and start the application
* from a browser login, create a survey and then submit some results
* review the log

One of the things you will need to bear in mind when you are trying to debug issues is that you can use Kiro to expand and drill down into specific sections of your code and add more logging. This is how you can generate more output that could be useful to Kiro in helping it understand where a problem might lie.

Lets now add proper exception handling to the code. Switching back to the Kiro session, from the "[customer-survey]>" prompt, enter the following.

```
Review the code in the project workspace. Refactor code so that:

- try/catch blocks are added where it can help with debugging potential issues
- incorporates logging frameworks in use
- uses specific exceptions as well as a catch all one
```
Follow Kiro-CLI and as it asks for permission, review and work through the adding these new try/catch blocks. You should also look at:

* how this has changed the logging code?
* look at the try/catch blocks and how these have been split out
* review how many more lines of code your project now has

> **To steer or not to steer, that is the question!**
> 
> If you are wondering could I have just asked for both of these things to be added when we initially created the code, you are right. In the initial steering documents that were created in [Writing Code](/workshop/07-writing-code.md) we used [this steering document](/resources/steering-eg-pythong-dev.md). If we had added the above, the code generated would already be good to go.
> 

---

## 5. Context Management

The most effective way to get better debugging results from using Kiro is to make sure you are providing the right context. Here are some pointers that will help you ace this every time.

### Technical documentation

One of the most important documents you can provide as context to Kiro when debugging is technical documentation that describes the technical setup. This document (or documents) should cover:

* setup - precisely outline how the application is setup and include details such as versions, frameworks used, libraries, etc
* structure - the layout and structure of your project, outlining key directories and what they are for
* architecture - outline the application architecture, including components, modules, interfaces
* data model - provide details of the application's data model
* global references - provide detail of key references that are used throughout the application (key variables, classes etc)

In the [Documenting labs](/workshop/06-understanding.md) we covered how you can generate these for your project if you do not currently have this information available.

You can keep this documentation as one file or smaller files. What approach you take will depend on the complexity of the projects you are working on, and you should experiment with what works best for your projects, but keep in mind that you do not want these documents to be too big.

You will need to determine how you want Kiro CLI to pick up these documents as context. The default Kiro CLI uses just picks up the following: README.md, AGENTS.md, AmazonQ.md and then anything in the .kiro/steering directories. You can adjust this by making changes to the custom agent "Resources" configuration option.

For debugging use cases though, I find it better to manually add these document on an as needed basis using the "/context add" command. If we have docs such as "ARCH.md" and "DATA-MODEL.md" in the docs directory, then I can use:

```
> /context add docs/ARCH.md

Added 1 path(s) to context.
Note: Context modifications via slash command is temporary.
```

And when I review my context using "/context show" I can see that these docs are session scoped context, only valid for my current working session:

```
Session (temporary)
  /{project directory}/kiro-cli/kiro-customer-survey/docs/ARCH.md
  /{project directory}/kiro-cli/kiro-customer-survey/docs/DATA-MODEL.md
```

### Be specific in what you add

Whilst you can add as much context as you want, I have found that you will get better results if you are intentional in what context you provide.

How do you know what to include? This is something that you will learn over time as you start using these tools. It is a combination of understanding the code base, thinking about the problem, and then using your intuition as to what would be useful context. For smaller projects this tends to be less of an issue, but as your code bases grow larger and the problems get more specific, this technique will help you get more useful answers from Kiro.

### Use screenshot images as context

You can use more than logs and stack traces to help you debug issues. They say a picture is worth a thousand words, and this is very true when it comes to debugging issues (especially issues that involve something visual). You can  attach screenshots (and annotate them), providing text to explain what the issue is. This can be useful when other ways of trying to explain an issue are imprecise.

### Browser logs

Use browser debugging tools and then use the output of these (Inspector/Console) to provide context and errors that might be missing from the server side.

### Integrate with a Language Server Protocol (LSP)

In the previous labs we introduced Kiro's Code Intelligence, an LSP that provides Kiro with a deep understanding of your code base using external programs. This is an excellent way to provide Kiro with rich context and capabilities that will help it understand the problem better.

### Using Model Context Protocol (MCP) Servers

MCP Servers can be useful in providing context. There are many MCP Servers that provide tools to help you debug and troubleshoot issues, sometimes by directly accessing systems (for example, PostgreSQL MCP servers that connect to a database with a set of limited access credentials and then help you resolve issues you have). 

If you take this approach, updating your custom agents to include these make sense. MCP Servers love to eat through your context window, so keep this in mind, and use techniques outlined below to help you manage this.

A quick example of how you might use this. I have a number of workloads on AWS, and these are integrated into Amazon CloudWatch logs. There is an excellent MCP Server that integrates directly with CloudWatch, providing me tools to do things like search through log groups. I can use a prompt like this for example, 

```
Review the CloudWatch log group "kiro-clo-debugging-demo" for any anomalies in eu-west-1"
```

Kiro CLi will then prompt me for permission to use the MCP Server tool, connect directly to CloudWatch, and then review the logs for me and find the information I need.

As you look to debug your issue, if you are using external systems then you might want to explore what MCP Servers are available that might be useful.

---

## 6. Avoid polluting your context window when debugging

Context windows are a limited resource. In addition, we know from research done that despite context windows getting bigger, this does not lead to a linear improvement in the output. There is a danger when debugging issues that you can fill your context window, or trigger one of the context failure modes that lead to poor outputs.

One technique that is useful when debugging is to start a new session. Alternatively, you can use a cool (currently experimental) feature in Kiro CLI called "/tangent". This feature can help you reduce overloading your context window by spawning a new conversation with a new context window. You can do whatever work you want, and then "return" back to your session - context window preserved.

You can check out how to use tangent mode in the [Advanced Topics](/workshop/04-advanced-topics.md) section of this workshop, under the Experiment section.

---

## 7. Using follow up questions to get more details

You do not have to limit yourself to the knowledge you currently have on debugging and troubleshooting. Kiro CLI is a very competent companion and coach, so you can try asking Kiro for "What information do you need to debug this issue", or "What are likely reasons why I would get XXXXX"?

You can also use the conversational nature of Kiro CLI to dig deeper and surface up information which you can then use during your debugging activities.

> Initial Prompt: "Here is my {code snippet} and the {error}. What are some potential causes?"
> Follow-up Prompt: "You suggested it might be a type mismatch. Can you explain where in the code that mismatch could be happening?"
> Final Prompt: "Okay, I see the issue. Can you suggest a more efficient way to refactor this function to prevent this error in the future?"

You will find that your intuition on how to craft good follow up questions will improve over time. One technique I have used is to mimic the 5 Whys, so using the same example above.

> Initial Prompt: "Here is my {code snippet} and the {error}. Use the 5 Whys to help me understand the potential and get to the root cause of the problem

---

## 8. How to undo code changes successfully

During debugging you may end up in a situation where you have spent more time than you have to resolve the issue. At this point you will need to think about how to go back to your working state and start again.

* Use source control - one way you can revert changes is to use git (or your preferred tool) to revert back to the previous working code. When using AI coding assistants, this is why you should work on discrete tasks and small blocks of code. This will limit the amount of change and allow you to revert back and then try again.

* Use Checkpoints and Restore - Kiro provides a feature (currently experimental) called checkpoints that allows you to manage project workspace checkpoints, essentially allow you create checkpoints and then to quickly restore back to a checkpoint 

**Do not rely on your AI coding assistant to undo changes via the prompt - this is too unreliable.**

---

We are going to put together some of the things we have learned so far and show how we can go through and debug an issue. First we are going to need to generate a problem, so we will do that and then take it from there.

**Task-05**

Exit out of your Kiro session to get back to the prompt. From the terminal we are going to need to "modify" our application so that it breaks.

Edit the app/src/models/database.py file, and change line 14 from

```
class User(db.Model):
```

to

```
class Users(db.Model):
```

Save the file, and then confirm that the application no longer starts when running "uv run python run.py" (you should see something like this)

```
ImportError: cannot import name 'User' from 'app.src.models.database' (/{project directory}/kiro-cli/kcs/app/src/models/database.py)
```

Staying in the terminal, rename the README.md file to ARCH.md

```
mv README.md ARCH.md
```

Start Kiro CLI using our code debugging custom agent, using the following:

```
kiro-cli --agent code-debugger
```

We enabled the Code Interpreter in an earlier lab, which you can see by the "λ" character. This will provide Kiro information about the code base as and when it is needed.

From the "[code-debugger]>" prompt, check what context you have by using "/context show" - you should have nothing (no matches) in your context.

```
[code-debugger] λ > /context show

Agent (code-debugger)
  - AGENTS.md (no matches)
  - README.md (no matches)

Session (temporary)
  <none>

No files in the current directory matched the rules above.
```

We are now going to add the ARCH.md doc to our context. To do this we use the "/context add" command, so run this from your Kiro session:

```
/context add ARCH.md
```

which should generate the following:

```
Added 1 path(s) to context.
Note: Context modifications via slash command is temporary.
```

Now lets start debugging this issue. We are going to use some of the good practices we have learned from this section. From your Kiro session, use "/editor" and enter the following prompt:

```
Please help me debug an issue. WHEN I START the application from the command line (using uv run python run.py), I get an ERROR message:

\```
ERROR - app.src.app - Error importing routes: cannot import name 'User' from 'app.src.models.database' (/{project directory}/kiro-cli/kcs/app/src/models/database.py)
CRITICAL - app.src.app - Critical error during application creation: cannot import name 'User' from 'app.src.models.database' (/{project directory}/kiro-cli/kcs/app/src/models/database.py)
Traceback (most recent call last):
  File "/{project directory}/kiro-cli/kcs/run.py", line 9, in <module>
    app = create_app()
          ^^^^^^^^^^^^
  File "/{project directory}/kiro-cli/kcs/app/src/app.py", line 47, in create_app
    from .routes import auth, dashboard, surveys
  File "/{project directory}/kiro-cli/kcs/app/src/routes/auth.py", line 10, in <module>
    from ..models.database import User
ImportError: cannot import name 'User' from 'app.src.models.database' (/{project directory}/kiro-cli/kcs/app/src/models/database.py)
\```

Running this command SHOULD start the application and make it available on port http:127.0.0.1:5001.

Other things you should know:
- The application has an error log called "app.log".
- The application design and technical details are in ARCH.log
- The issue STARTED after I made some changes to the app/src/models/database.py

When considering a FIX, make sure you select a STRONG PREFERENCE to align with the existing architecture and technology defined.

```

Follow the output. Remember how we created a detailed system prompt for our custom agent? You should now see that this is echoed back. This is the output I get, and it should be broadly similar to yours.

```
> I can see you're dealing with an import error where the User class cannot be imported from app.src.models.database. Let me
gather some more information to help debug this effectively.

To better understand the root cause, I need to ask a few questions:

1. What specific changes did you make to app/src/models/database.py?
- Did you rename, delete, or move the User class?
- Did you modify the class definition or its imports?

2. When exactly did this issue start?
- Was it immediately after your changes, or after some other action?

3. What was the expected behavior vs. what's happening now?
- Expected: Application starts successfully on port 5001
- Actual: Import error preventing startup

Let me first examine the current state of your database.py file and the importing file to see what's missing:
Reading file: /{project directory}/kiro-cli/kcs/app/src/models/database.py, all lines (using tool: read)
 ✓ Successfully read 3841 bytes from /{project directory}/kiro-cli/kcs/app/src/models/database.py
 - Completed in 0.1s

> Now let me check the auth.py file to see exactly what it's trying to import:
Reading file: /{project directory}/kiro-cli/kcs/app/src/routes/auth.py, from line 8 to 15 (using tool: read)
 ✓ Successfully read 299 bytes from /{project directory}/kiro-cli/kcs/app/src/routes/auth.py
 - Completed in 0.1s

> Root Cause Identified:

The issue is clear now. In your database.py file, you have a class named Users (plural), but your auth.py file is trying to
import User (singular). This mismatch happened when you made changes to the database.py file.

Expected vs. Actual:
- **Expected:** Import User class from database module
- **Actual:** The class is named Users, so the import fails

The Fix:

You have two options:

1. Option 1: Rename the class from Users to User in database.py (recommended for consistency)
2. Option 2: Update all import statements to use Users instead of User

Given the Flask/SQLAlchemy convention typically uses singular model names, I recommend Option 1.

Would you like me to apply the fix by renaming Users to User in your database.py file? This will maintain consistency with your
existing import statements and follow standard naming conventions.
 ▸ Credits: 0.15 • Time: 13s
```

As you can see, it has not fixed the issue but is asking you for next steps. We are not going to fix this at the moment, as we are going to see what happens if we don't provide the detailed prompt.

From your Kiro session, at the "[code-debugger]>" prompt, use the "/clear" to clear the context window.

```
/clear
```

which will prompt you to confirm you want to clear

```
Are you sure? This will erase the conversation history and context from hooks for the current session. [y/n]:
```

Answer "Y", and you should get a "Conversation history cleared." message. From the prompt now type:

```
Help me debug an issue with the app
```

A much shorter prompt, and one that perhaps you might have used in the past. What you should notice is that you will now be prompted to supply the information that we had originally provided in the prompt. This is the output that it generated for me.

```
help me debug an issue with the app

> I'm Orac, and I'll help you debug this issue. To get to the root cause, I need some key information:

Tell me about the error:
- What exactly happens when you encounter this issue? What were you doing when it occurred?
- What specific error messages or symptoms do you see?

Context questions:
- When did this issue start happening?
- Did anything change recently? (code updates, dependencies, environment changes, etc.)
- What's the expected vs. actual behavior? Be specific - for example, "I expected JSON response, but got HTML" or "I expected the
function to return X, but it returns Y"

Once I understand these details, I can help identify the root cause and suggest a fix.
 ▸ Credits: 0.04 • Time: 5s
```

Using the information from the original prompt, provide answers and follow the debugging conversation. It should come to the same conclusion, through a different route. This is the output I got.

```

[code-debugger] λ > The problem starts when starting the app. It should start and then start listening on port 5001 so I can access it via a web browser. It started happening after I made a change to the database.py file. The app should just start when i run "uv run python run.py" but its generated this error - ImportError: cannot import name 'User' from 'app.src.models.database' (/{project folder}/kiro-cli/kcs/app/src/models/database.py)

> Perfect! This is a clear import error. The application is trying to import the User class from app.src.models.database but can'
t find it.

Since you mentioned you made changes to the database.py file, let me examine the current state of that file to see what's missing
or incorrect.
Reading file: /{project folder}/kiro-cli/kcs/app/src/models/database.py, all lines (using tool: read)
 ✓ Successfully read 3841 bytes from /{project folder}/kiro-cli/kcs/app/src/models/database.py
 - Completed in 0.1s

> Found the issue! The class is named Users (plural) in your database.py file, but the import is trying to import User (singular)
.

Let me check where this import is happening to confirm:
Searching for: from.*User.*import|import.*User in /{project folder}/kiro-cli/kcs (*.py) (using tool: grep) - Completed in 0.40s

> The problem is in /{project folder}/kiro-cli/kcs/app/src/routes/auth.py - it's trying to import User but your model class is named
Users.

Root cause: Class name mismatch between the model definition (Users) and the import statement (User).

Fix options:
1. Change the class name from Users to User in database.py
2. Change the import statement to import Users instead of User

Option 1 is typically preferred since model names are usually singular in most conventions.

Would you like me to go ahead and apply the fix by changing Users to User in the database.py file?
```

Complete the fix by asking Kiro to fix using Option 1. Follow it through and once complete, open up a terminal session to make sure that you can now start the application.

---

## 9. Other ways Kiro can help you

So far we have looked specifically at how you can use Kiro for helping debug issues, but you can also provide help with other sort of issues you have. Here are just a few ideas of how I have used Kiro, but this only scratches the surface of what is possible.

### Deciphering error messages

How many times have you seen errors in your logs or compilation and needed help to understand it. As it turns out, Kiro CLI is great at turning a cryptic error code into a plain English explanation.

### Spotting typos

Probably the most common way I currently get help from Kiro CLI is to help me spot subtle typos in the code - whether its a variable name that is slightly wrong, or messed up parenthesis, Kiro is really effective at helping to highlight potential issues like this.

### Walkthrough your code

Not sure how some function or part of your codebase works? No problem, you can use Kiro as a step debugger and ask it to give you a step by step commentary on how your code works. What is great about this is that you can stop at any point and introspect and dive deeper. This can be very helpful when you are debugging issues and you have identified where the issue is, and you now need to walk through a function's execution path to find where the reasoning went wrong.

### Rubber Ducking

According to Wikipedia [Rubber Ducking](https://en.wikipedia.org/wiki/Rubber_duck_debugging) is a classic approach to debugging and troubleshooting that involves explaining your code, line-by-line, to an inanimate object. The act of verbalizing your logic often reveals the flaw. You can use Kiro as the ultimate rubber duck because it can talk back.

Prompt: "I am trying to debug this complex function. I'm going to explain my thought process for how it's supposed to work. Please listen to my explanation and point out any logical fallacies, incorrect assumptions, or potential edge cases I might be missing."

---

## 10. When nothing works

So you have gone through this lab and when you apply it to try and debug issues you are finding you are struggling. What else can you do in this situation. Here are a couple of suggestions that you might want to think about and try:

* Try a more sophisticated model - Kiro offer different models. If you are struggling to get to a resolution, try different models. Starting with Auto, work to increasingly more sophisticated models as these have different reasoning capabilities as well as context windows.

* Clearing context and starting again - Sometimes context can get confused, so open a new chat window and try again. There can be lots of reasons for this, so using /clear and re-starting is something to try if everything else has failed. This technique tends to work better when combined with other tips in this doc (for example, maintaining an issue log)

* Negative Space programming techniques - negative space programming is focused on reducing the scope where problems can arise by removing and simplifying your code. It works on the basis that it produces less code, forces constraints which leads to better design and is simpler, therefore easier to read. You can use tools like Kiro CLI to refactor code using negative space techniques.

---

## Completed!

Head back to the [homepage](/README.md) and see what other labs you might want to try out.

