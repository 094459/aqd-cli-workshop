![Amazon Q Developer header](/images/kiro-workshop-header.png)

## Overview

We are going to put in practice some of the techniques we have learned and use them to build an application. In the first part of this lab we are going to build a simple application to show how good Kiro CLI is at generating code. We will share good practices that you should think about as you look to use Kiro CLI in your day to day workflow.

*What are we going to build?*

We are going to build a customer feedback survey application. The application will initially have some basic functionality, an MLP - Minimum Lovable Product. We will be using Kiro CLI to help us add code and improve the basic functionality. We will dive into the details as we get into the workshop, but we want our application to be able to:

* allow users to register and then login to this application to use it
* let logged in users create surveys
* make it so that anyone can submit feedback from a survey
* let only owners of a survey see the survey results

## Creating our project workspace

From a terminal, navigate to a new directory where we will start work (In my setup I have a "projects" directory  where I have all the code I write, so change this to whatever you use). Once in your chosen project directory, type in the following. 

```
mkdir kiro-customer-app
cd kiro-customer-app
```

This will be our working directory for this lab. Don't start Kiro CLI just yet, as there are some thing we want to setup before we start.

---

**Task-01**

The first thing we are going to do is create some context files which Kiro CLI will use when generating output. As we mentioned in earlier parts of this tutorial, context is critical in getting good results when using AI coding assistants. As we think about what context we might want to provide to help steer Kiro CLI, we are going to provide context around how we want the code to be structured, and which technologies that we want it to use. 

In the current (kiro-customer-app) directory:

1. Create a new directory **".kiro/steering"**
2. In the **".kiro/steering"** create one file. **"python-dev.md"**
3. In the **"python-dev.md"** file, add the contents of the [steering-eg-python-dev.md](/resources/steering-eg-pythong-dev.md) file that is in the resources directory of this project.

After saving these files, you should something that looks like the following in your project workspace.

```
└── .kiro
    └── steering
        └── python-dev.md
```

Not very exciting yet!

Whilst this file exists, we need to make sure that our Kiro CLI session uses this. We will do that in the next lab.

---

**Task-02**

We have created some context information (steering file in the previous lab) and there is likely to be more stuff we want to add. Creating a new custom agent will allow us to keep all of this together, so in this next lab we are going to create a custom agent and use this to define some additional context files we want to use just for this project. We are going to create a project scoped custom agent, and then use that to configure context initially.

Start Kiro CLI, 

```
kiro-cli
```

and from the **">"** prompt, lets create our profile

```
> /agent create -d . --name customer-survey
```

Which will bring up the default editor in Kiro CLI, and look like the following:

```
{
  "name": "customer-survey",
  "description": "",
  "prompt": null,
  "mcpServers": {},
  "tools": [
    "*"
  ],
  "toolAliases": {},
  "allowedTools": [],
  "resources": [
    "file://AGENTS.md",
    "file://README.md"
  ],
  "hooks": {},
  "toolsSettings": {},
  "useLegacyMcpJson": true,
  "model": null
}
```


This will bring up the editor to edit the JSON configuration for this custom agent. Edit the file so that it looks like the following:

```
{
  "name": "customer-survey",
  "mcpServers": {},
  "tools": ["*"],
  "toolAliases": {},
  "allowedTools": [],
  "resources": [
    "file://data-model/*",
    "file://.kiro/steering/**/*.md"
  ],
  "hooks": {}
}
```

We have created a custom agent and configured it to look for context resources in a couple of directories. We will come back to this again later on to make changes. You will notice that the first resource we have listed (file://data-model/*) does not exist in our project workspace. That's fine, we will be adding this later on.

When we create custom agents, we can check that this has been configured correctly by checking using the **"/agent list"** command:

```
> /agent list

* kiro_default       (Built-in)
  customer-survey    /{your project directory}/kiro-customer-app/.kiro/agents
```

If we look at our file system now, it should look something like:

```
└── .kiro
    ├── agents
    │   └── customer-survey.json
    └── steering
        └── python-dev.md
```

We now need to restart Kiro CLI and restart it so that we can start using our custom agent. We run the following command:

```
kiro-cli --agent customer-survey 
```

You will notice that even though this is a local custom agent, we did not need to specify this when using the **"--agent"** argument. This is because Kiro CLI will look at the local project workspace first before looking for globally scoped custom agents.

You should notice that your prompt looks like the following:

```
[customer-survey] >
```

Now run the following command to view our current context that is configured within our custom agent.

```
/context show
```

Your output should be similar to the following:

```
Agent (customer-survey)
  - data-model/* (no matches)
  - .kiro/steering/**/*.md /{your project directory}/kiro-customer-app/.kiro/steering/python-dev.md

Session (temporary)
  <none>

1 matched file in use
- /{your project directory}/kiro-customer-app/.kiro/steering/python-dev.md (0.2% of context window)

Context files total: 0.2% of context window
```

We have successfully created our custom agent, and configured some local resources as context. However, as we are going to be building an application we want to add some more context next.

---

**Task-03**

When writing code, providing a data model as context is a useful way to ensure that the code generated has a good baseline to work from.

Exit your Kiro CLI session, and create a new directory called "data model" in the current project directory, and then copy the [data model from the resources directory](/resources/database_schema.yaml) into this directory.

You should now have the following directory layout.

```
├── .kiro
│   ├── agents
│   │   └── customer-survey.json
│   └── steering
│       └── python-dev.md
└── data-model
    └── database-schema.yaml

```

Start Kiro CLI again using our local custom agent:

```
kiro-cli --agent customer-survey
```

When Kiro CLI starts, run **"/context show"** to see how your environment looks. It should look something similar to the following:

```
[customer-survey] > /context show

Agent (customer-survey)
  - data-model/* /{your project directory}/kiro-customer-app/data-model/database-schema.yaml
  - .kiro/steering/**/*.md /{your project directory}/kiro-customer-app/.kiro/steering/python-dev.md

Session (temporary)
  <none>

2 matched files in use
- /{your project directory}/kiro-customer-app/.kiro/steering/python-dev.md (0.2% of context window)
- /{your project directory}/kiro-customer-app/data-model/database-schema.yaml (0.3% of context window)

Context files total: 0.5% of context window
```

Our Kiro CLI session now shows that both a rule and data model are in our context. We want to use these to help ground decisions and outputs it generates.

---

**Task-04**

We are now ready to ask Kiro CLI to write some code for use. For that we need a prompt. Here is one I prepared earlier.

```
Build a simple Customer Survey application. 

- Generate a web application that can be used in a browser
- Users will need to register with an email address to login
- When Users login, a Dashboard will be displayed that provides a simple explanation of what the application does.
- From the home Dashboard, Users will be displays any available Customer Surveys that they have been created
- When Users are viewing their Customer Surveys, they will have the ability to view the results
- When Users are viewing the Dashboard, any available Customer Surveys will provide a link that can be shared so people can submit feedback on a specific survey
- Provide a simple web design that can be updated easily using CSS
- Ensure you review the data model when building the application
- Ensure the database is initialised properly
```

When using Kiro CLI with a complex prompt like this, you will find it easier to use the "/editor" command. Before we do that though, we need to think about how we can use Kiro CLI to improve our initial prompt. Instead of add the above as a prompt, use the following:

```
Review the following prompt and suggest ways you can improve it:

Build a simple Customer Survey application. 

- Generate a web application that can be used in a browser
- Users will need to register with an email address to login
- When Users login, a Dashboard will be displayed that provides a simple explanation of what the application does.
- From the home Dashboard, Users will be displays any available Customer Surveys that they have been created
- When Users are viewing their Customer Surveys, they will have the ability to view the results
- When Users are viewing the Dashboard, any available Customer Surveys will provide a link that can be shared so people can submit feedback on a specific survey
- Provide a simple web design that can be updated easily using CSS
- Ensure you review the data model when building the application
- Ensure the database is initialised properly
```

From your "[customer-survey] >" prompt, paste that text into the prompt and review the output. What do you notice?

We are now going to move this from the chat and into a document. This is an important technique that allows you to have an resource that can be used as context for Kiro CLI. Having this file will also make it easier for you to track progress and make changes as needed.

From your "[customer-survey] >" prompt, enter the following text:

```
write this to a doc called implementation.md that you can follow to start writing code
```

Kiro CLI should prompt you for permission to write a file.

```
Allow this action? Use 't' to trust (always allow) this tool for the session. [y/n/t]:
```

Answer "y" so that it can create the file. Once it has created, your project workspace should look like this:

```
├── .kiro
│   ├── agents
│   │   └── customer-survey.json
│   └── steering
│       └── python-dev.md
├── data-model
│   └── database-schema.yaml
└── implementation.md
```

Review the file. Does it look as you expect? What about the technology choices, do they match the steering document and the data-model? Typically you would spend time here reviewing, editing and refining this document before proceeding. For this workshop we will do a light review, adjust anything we see as obviously wrong, and work with that to start building our application.

> You can see the output of when I ran this by looking at the [resources directory](/resources/implementation-plan-eg.md).

---

**Task-05**

In the previous lab we saw that when we asked Kiro CLI to create the implementation document, we were prompted for permission. Kiro uses "tools" to be able to take various actions (read a file, write a file, run a command, etc) and by default these permissions have sensible defaults. You can view these again by running the following command from your Kiro CLI prompt:

```
/tools
```

You should see that your tools (and their permissions) look similar to the following:

```
Tool              Permission
▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔
Built-in
- shell           not trusted
- read            not trusted
- write           not trusted
- introspect      trusted
- report          not trusted
- aws             trust read-only commands
- web_fetch       not trusted
- web_search      not trusted
```

When we use Kiro CLI to write some code, maybe run it to make sure its working, we will be prompted every time Kiro CLI wants to do something that exceeds the permission of a given tool. In agentic workflows this is suboptimal as you ant to leave the tool running.

We can configure tool permissions within custom agents. We looked at this during the [advanced topics](/workshop/04-advanced-topics.md). In this next lab we are going to make some changes to that Kiro CLI can get on with writing its code.

From your "[customer-survey] >" prompt, enter the following

```
/agent edit --name customer-survey
```

And amend your custom agent configuration so that it looks like the following:

```
{
  "name": "customer-survey",
  "mcpServers": {},
  "tools": ["*"],
  "toolAliases": {},
  "allowedTools": ["read","write"],
  "resources": [
    "file://data-model/*",
    "file://.kiro/steering/**/*.md"
  ],
  "hooks": {}
}
```

Save and exit. You will then need to restart Kiro CLI, so exit and then start with

```
kiro-cli --agent customer-survey
```

Now try running the "/tools" command again. What has changed? You should see that read and write are now trusted. We will see if this works when we run the next lab.

---

**Task-06**

Now we have an implementation plan, we can get Kiro CLI to start writing some code for us. From the From your "[customer-survey] >" prompt, enter the following text:

```
start building the app by reading the implementation.md and following the plan
```

You should now see Kiro CLI burst into activity. Look at the activity, and you will see that read and write operations are now running without you being prompted:

```
> I'll read the implementation.md file to understand the plan and start building the app.
Reading file: /{your project directory}/kiro-customer-app/implementation.md, all lines (using tool: read)
 ✓ Successfully read 2847 bytes from /{your project directory}/kiro-customer-app/implementation.md
 - Completed in 0.1s
```

and

```
> Now I'll start implementing the app following the plan. Let me begin by setting up the Flask
application structure:
I'll create the following file: /{your project directory}/kiro-customer-app/app/config.py (using tool: write)
Purpose: Create Flask configuration file
```

After writing some code, Kiro CLI will need to start running some commands. This is what popped up after about 3-4 minutes in my session, and you will see something similar.

```
> Now I'll install the required dependencies using uv:
I will run the following command: cd /{your project directory}/kiro-customer-app/app && uv init --no-readme --no-workspace (using tool: shell)
```

It needs to run a command so is requesting the use of the shell tool. In this instance I am going to respond "t" to trust this. When you trust a tool, that trust is only valid for the duration of that session. Once I exit Kiro CLI, trust will revert back to "not trusted".

It will take 4-5 minutes to run some commands (to install libraries and dependencies, and start the server), so a great time to stretch your legs and rehydrate.

---

**Task-07**

Now Kiro CLI has created the application code, we need to start it so we can test to make sure it is working. During the previous lab, a README might have been created that explains how to start this application. We will ignore that for the moment, and get Kiro CLI to help us get the app up and running

From the From your "[customer-survey] >" prompt, enter the following 

```
Show me how to start this application
```

Review the output - it should provide you with detailed instructions on how to start the application. The steering document defined uv as the tool for library and dependency management, and so your instructions should highlight how to use uv to start the application.

We can also get Kiro CLI to start it for us. Follow the previous prompt with this:

```
Can you start the app for me so I can test it
```

After a few seconds, Kiro CLI should start the application. Open the link using a web browser and test it to make sure its working. Congratulations, you have built your first application.

Stop the application (CTRL + C) before proceeding with the next lab.

---

**Task-08**

Kiro CLI makes it easy to add tests to our project. In this next lab we are going to generate tests for the code we have just created. 

From the **"[customer survey]>"** enter the following prompt:

```
> Add unit tests to this project for the routes and models. Aim for 90% test coverage
```

Try and follow the output. You will see that it again references what we defined in our steering document and starts to install and write tests using pytest and fixtures.

It is going to vary in how much time it will take, but expect anything from around 5-15 minutes. When it does finish, it should generate a status update confirming the tests have been generated, and the coverage. This is the output from my machine:

```
================================ tests coverage ================================
______________ coverage: platform darwin, python 3.11.11-final-0 _______________

Name                      Stmts   Miss  Cover   Missing
-------------------------------------------------------
src/__init__.py               0      0   100%
src/extensions.py             2      0   100%
src/models/__init__.py        3      0   100%
src/models/response.py        9      0   100%
src/models/survey.py         21      0   100%
src/models/user.py           15      0   100%
src/routes/__init__.py        0      0   100%
src/routes/auth.py           42      0   100%
src/routes/dashboard.py      24      0   100%
src/routes/public.py         31      0   100%
src/routes/survey.py         61      0   100%
-------------------------------------------------------
TOTAL                       208      0   100%
Coverage HTML written to dir htmlcov
Required test coverage of 90% reached. Total coverage: 100.00%
============================== 41 passed in 5.00s ==============================
🧪 Running tests with coverage...

✅ All tests passed with 90%+ coverage!
📊 Coverage report generated in htmlcov/index.html
```

If your tests generate warnings, use the chat interface to work through these. For example, you can enter:

```
Resolve any warnings that come up during the tests
```

And see how Kiro CLI will quickly review and update the tests accordingly.

You can run the tests directly from Kiro CLI, by using the "!" to run a command. From the "[customer-survey] >" prompt, enter the following:

```
!pytest
```

After hitting return it should run the tests, and you should see something like:

```
====================================== test session starts ======================================
platform darwin -- Python 3.10.13, pytest-8.3.2, pluggy-1.5.0
rootdir: /{project directory}/kiro-customer-app
plugins: cov-5.0.0, flask-1.3.0, anyio-4.10.0, typeguard-2.13.3, mock-3.14.0
collected 42 items

app/test_app.py .                                                                         [  2%]
app/tests/test_auth_routes.py ..........                                                  [ 26%]
app/tests/test_dashboard_routes.py ....                                                   [ 35%]
app/tests/test_models.py ........                                                         [ 54%]
app/tests/test_public_routes.py ........                                                  [ 73%]
app/tests/test_survey_routes.py ...........                                               [100%]

====================================== 42 passed in 8.54s =======================================

 ▸ Credits: 5.00 • Time: 2m 52s
```

Exit Kiro CLI and start up a code editor (have you tried Kiro IDE?) and use that to review the tests.

---

**Task-09**

We have our application up and running, but I want to do one final thing before we are done. I want to check the generated web application for accessibility. There are various open source tools that help you perform audits and help you take remedial action. Some of these are now available as MCP Servers, so we are going to configure one and then use it against the code we have created.

> **Note!** This MCP Server requires NodeJS to be available on your machine. From a new terminal window, try running:
>
> ```
> npx a11y-mcp
> ```
> If you get an error, then you will need to check the [documentation](https://github.com/priyankark/a11y-mcp?tab=readme-ov-file#installation) and install any dependencies needed.


We are going to update the custom agent, so from your Kiro CLI session, use the following command:

```
/agent edit --name customer-survey
```

And edit the file so that the JSON configuration so that it looks like the following:

```
{
  "name": "customer-survey",
  "mcpServers": {
    "a11y": {
      "command": "npx",
      "args": ["a11y-mcp"],
      "disabled": false
    }
  },
  "tools": ["*"],
  "toolAliases": {},
  "allowedTools": ["read","write"],
  "resources": [
    "file://data-model/*",
    "file://.kiro/steering/**/*.md"
  ],
  "hooks": {}
}
```

After saving, restart your Kiro CLI session with:

```
q chat --agent customer-survey-project
```

You should notice during the start that Kiro CLI attempts to start the MCP server. It will continue to load this in the background whilst it presents you with the "[customer-survey] >" prompt.

We can see the status of any MCP servers we have added using the "/mcp" command. From the Kiro CLI prompt enter:

```
/mcp
```

You should get output similar to the following (with a different time/date stamp)

```
> /mcp
a11y
▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔
[2025:21:47]: ✓ a11y loaded in 2.40 s
```

and we can see what additional tools have been made available with **"/tools"**:

```
[customer-survey-project] > /tools


Tool               Permission
▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔
Built-in:
- shell            not trusted
- read             trusted
- write            trusted
- introspect       trusted
- report           not trusted
- aws              trust read-only commands
- web_fetch        not trusted
- web_search       not trusted

a11y (MCP):
- audit_webpage    * not trusted
- get_summary      * not trusted
```

Lets use this tool to run an audit on our code. From the [documentation](https://github.com/priyankark/a11y-mcp?tab=readme-ov-file#available-tools) we can see that we need to provide a url for this to work.

Open up a new terminal session, and then start the customer survey application. Confirm its working by opening up a browser and logging in.

Switch back to your Kiro CLI session. From the prompt, type the following

```
> Use the a11y MCP server to audit http://127.0.0.1:5001 for accessibility issues
```

If your application is listening on a different port, update that prompt accordingly.

After a moment, Kiro CLI will ask you to confirm that you are happy to run this tool. If you look up, by default new tools are not trusted.

```
[customer-survey] > Use the a11y MCP server to audit http://127.0.0.1:5001 for accessibility issues

> I'll audit the local Flask application for accessibility issues using the a11y MCP server.
Running tool audit_webpage with the param (from mcp server: a11y)
 ⋮  {
 ⋮    "url": "http://127.0.0.1:5001"
 ⋮  }

Allow this action? Use 't' to trust (always allow) this tool for the session. [y/n/t]:
```

Confirm by responding "t" and then leave the tool to run and generate its findings. You should see traffic appear in the other terminal where you have the application running. Make sure you keep an eye on Kiro CLI as you will be asked to trust the new mcp tool again.

After a short while, Kiro CLI will generate a summary for you. This is the output from running it on my machine - yours might vary.

```
 - Completed in 2.179s

> Summary: Your application has 1 serious accessibility issue out of 24 total tests, with a 96%
pass rate. The single color contrast issue is easily fixable by adjusting the button styling.
 ▸ Credits: 0.13 • Time: 2m 38s
```

I can now ask Kiro CLI to remediate this one serious accessibility issue. From the Kiro CLI prompt I type:

```
address the serious accessibility issue and then re-run the analysis
```

After a few minutes, the code has been updated and it has re-run the accessibility audit. This time it looks much better.

```
> Fixed! I've changed all button backgrounds from #3498db (light blue) to #1f5582 (dark blue)
which provides a 7.2:1 contrast ratio with white text, exceeding the WCAG 2 AA requirement of 4.5
:1.

The changes were made to:
- Main button styling (line 100)
- .btn-primary class (line 115)
- Progress bar fill (line 258) for consistency

Your Flask application should now pass the accessibility audit. Refresh the page and the buttons
will have proper contrast for users with visual impairments.

```

Just to make sure, I access the application from a browser to make sure that this looks ok. When I check it looks better, so that is a big win.

---

## Summary

In this lab we have seen how we can use Kiro CLI capabilities to help us build an application within minutes, leveraging MCP Servers to make sure we build inclusive and accessible applications.

You can [proceed back to the README](/README.md).