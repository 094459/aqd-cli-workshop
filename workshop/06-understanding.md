![Kiro CLI header](/images/kiro-workshop-header.png)

# Documenting Codebases

In this lab we are going to build upon the previous labs but this time with a focus on how we can use Kiro CLI to generate next level documentation. We will also take a look at how you can use some of the same techniques to build up documentation of existing projects you have, where perhaps you have little or no documentation or understanding.

Creating good documentation has always been important, but in the era of AI coding assistants, it has become critical. The better the docs within a project, the better the context you can provide. In my experience, this has a significant impact on improving the quality of the output they generate.

## Overview

In the following labs we are going to carry on working on the project that you build in the [Writing Code](/workshop/07-writing-code.md) labs. If you have not completed that and just want to start from here, don't worry, there is a version of the application ready to go which you can use.

### Setting up your workspace

If you want to carry on working in the project you built as part of earlier labs then you can skip this section. If you are starting from new, then you will need to check out the sample customer survey application that was built as part of the [Writing Code](/workshop/07-writing-code.md) lab.

---

**Task-01**

From a terminal, create a new project directory that we will use during this lab. 

```
mkdir kiro-cli-documentation
cd kiro-cli-documentation
```

Now check out the repo we will be working with

```
git clone https://github.com/094459/kiro-customer-survey.git
cd kiro-customer-survey
```

We are now ready to get started.

---

### Creating a custom agent

As we have seen in previous labs, we can use Kiro CLI's custom agent feature to bundle up various resources that are optimized for a specific task. We are going to create two custom agents that will help us with documentation tasks that we do as part of this lab.

**Task-02**

From the terminal, make sure you are in your project directory. You should see something similar when you run the following command:

```
tree .kiro

.kiro
├── agents
│   ├── customer-survey.json
│   └── python-dev.json
├── prompts
├── shared
└── steering

```

Don't worry if it is not identical - as long as you have the agents and steering directory there then we should be good to go. Before we start, we are going to remove the README.md file that is in the project. From the terminal, delete the README.md file in the project using the following command.

```
rm README.md
```

The first thing we are going to do is to copy some prompts that will help define our custom agents. We will store these in the ".kiro/shared" directory.

```
cd .kiro/shared
wget https://raw.githubusercontent.com/094459/aqd-cli-workshop/refs/heads/main/resources/code-documenter.md
wget https://raw.githubusercontent.com/094459/aqd-cli-workshop/refs/heads/main/resources/documentation-guru.md
cd ../..
```

Which should add two new files into your project workspace. So what are these?

* code-documenter.md - this provides a sample prompt that we are going to use that will ensure that we apply good practices to documenting the code within a project.
* documentation-guru.md - this sample prompt provides some general instructions on how you want to generate documentation for a project

Both of these are just initial starting prompts, and these can easily be built upon and improved based on your own needs and requirements. You should also think about how you might incorporate these into other custom agent prompts you might create - for example, adding these to code generation prompts to ensure that documentation is consistently created during the generation.

We are going to use these as prompts for the custom agents we create in the next step.

Start Kiro CLI from the terminal

```
kiro-cli
```

And from the ">" prompt, we are going to use the custom agent generation wizard so type the following.

```
/agent generate
```
You will then be asked three questions:

* agent name - enter "doc-code"
* agent description - enter "Custom agent for documenting code"
* agent scope - select LOCAL

It should look like this.

```
> /agent generate

✔ Enter agent name:  · doc-code
✔ Enter agent description:  · Custom agent for documenting code
✔ Agent scope · Local (current workspace)
```
When you hit enter after selecting Local, it will bring up the custom agent configuration file in the editor. Modify it so that it looks like the following:

```
{
  "name": "doc-code",
  "description": "Custom agent for documenting code",
  "prompt": "file://../shared/code-documenter.md",
  "tools": [
    "*"
  ],
  "toolAliases": {},
  "allowedTools": [],
  "resources": [
    "file://AGENTS.md"
  ],
  "hooks": {},
  "toolsSettings": {}
}
```

Save and you should be greeted with the following message.

```
✓ Agent 'doc-code' has been created and saved successfully!
```

Right we have created our first custom agent (doc-code) and we are going to repeat this process, creating another custom agent called "doc-guru". From the ">" prompt, 

```
/agent generate
```

enter the following:

```
✔ Enter agent name:  · doc-guru
✔ Enter agent description:  · Custom agent for documenting projects
✔ Agent scope · Local (current workspace)
```

And when the editor comes up, change the configuration so that it looks like the following:

```
{
  "name": "doc-guru",
  "description": "Custom agent for documenting projects",
  "prompt": "file://../shared/documentation-guru.md",
  "tools": [
    "*"
  ],
  "toolAliases": {},
  "allowedTools": [],
  "resources": [
    "file://AGENTS.md"
  ],
  "hooks": {},
  "toolsSettings": {}
}
```

After saving and exiting, we are done. We now have two new custom agents called "doc-code" and "doc-guru", which we can now text. Exit your Kiro CLI session, and then starting from the same terminal window.

```
kiro-cli --agent doc-code
```

When it starts, you should note that the prompt changes to "[doc-code]>" - if that works ok, exit and try the next custom agent.

```
kiro-cli --agent doc-guru
```

Congrats, we are now ready to put these to the test.

---

### Documenting code

Our project currently has some basic documentation within the code, however, the docstrings are one liners and do not really provide much information that is useful. Lets change that with the help of our custom agent that will take our documentation up a level. 

**Task-03**

From the terminal, lets start Kiro using the "doc-code" custom agent, so start Kiro using

```
kiro-cli --agent doc-code
```

From the "[doc-code]>" prompt, now enter the following:

```
Review the project and update the existing docstrings within the code.
```

After hitting enter, you should see Kiro CLI kick into action. As initially it needs just READ access, you will not be prompted for tool permissions. Once it has read the code, and then wants to make some updates it will prompt you with something like:

```
> Now I'll update the docstrings in all the Python files to follow PEP 257 standards. Let me start with the models:
I'll modify the following file: src/models/user.py (using tool: write)
```

This might take 3-4 minutes to complete, but you can watch as Kiro scrutinizes the code and then makes what it thinks are suitable updates. When it finishes,

* review the application source files - use "git diff" to see the differences, or if you prefer view this in your favorite IDE
* see if it produced a markdown file - sometimes Kiro will generate a report of what it done

You should notice that the docstrings are now different and they following the guidelines we defined in our custom agent's system prompt.

---

### Generating documentation

We now have updated the documentation in our code, but there is no documentation about our project. In this lab we are going to use a combination of custom agent and prompt to create documentation to be proud of.

Before we dive into the lab, one thing you should be thinking of as you start to use tools like Kiro CLI to help you with documentation is how you are going to approach this task. There are different strategies you might want to employ. For example:

* creating single vs multiple docs to help with context management
* build up your documentation iteratively as your project evolves
* leverage Kiro's custom hooks to automatically update the documentation after files have changed

Ok, keep that in mind as you progress through this next lab.

**Task-04**

Exit the current Kiro CLI session you are in, and re-start Kiro using the "doc-guru" custom agent using

```
kiro-cli --agent doc-guru
```

From the "[doc-guru]>" prompt, enter the following:

```
Review the code in the project workspace and generate comprehensive project documentation. Create documentation in a file called README.md.
```

Kiro CLI will start doing its thing - you will see that it starts off by reading the project files and trying to make some sense of it. Once it's ready, you will be prompted for permission to write some documentation files. This can take between 2-5 minutes to complete, but bear in mind for bigger, more complex projects this can take longer.

Review the output of the generated documentation and look for the following:

- What is the general quality (accuracy, completeness) of the README file created
- Did it miss anything out?
- Did it create any additional documentation?
- Did it create any diagrams?

In the next lab, we will explore more the last review question, and how Kiro CLI can not only generate text documentation, but also diagrams too.

#### Going deeper

One of the nice things about using Kiro CLI to help you with documentation is that if you need to, you can dive really deep into specific details of how the code in your project works. We explored this in earlier labs, when we looked at how you can use Kiro CLI as a pair programmer.

**Task-05**

For example type this in:

```
> can you provide in-depth and deep explanation of how the authorization works in this codebase. Go level 500
```

Review the output. It should walk you through in great detail how authorization works for this codebase. Imagine you were new to this codebase, how useful do you think this would be?

---

### Generating diagrams

Building off the previous lab, lets enhance our documentation to add some diagrams. Specifically lets look at how we can use Kiro to generate some nice sequence diagrams of the auth and user registration flows.

**Task-06**

We are going to stay in the current custom agent from the previous lab, so from the "[doc-guru]>" prompt, type the following:

```
Generate sequence diagrams for the Authentication flows, showing user registration, login, and then application access.
```

after hitting enter, you should see Kiro start to think

```
> I'll generate sequence diagrams for the authentication flows. Let me first
examine the authentication code to understand the exact flow, then create the
diagrams.
Reading file: /{project workspace}/kiro-cli/kiro-customer-survey/src/routes/auth.py, all lines (using tool: read)
 ✓ Successfully read 2469 bytes from /{project workspace}/kiro-cli/kiro-customer-survey/src/routes/auth.py
 - Completed in 0.143s
```

It should not take very long, and soon you will have an update to your README file. Here I use the enhanced markdown preview tool to display the updated README. Yours might look a little different but take a few moments to review your output.

![Generated sequence diagrams](/images/sequence-diagram.png)

Back at the "[doc-guru]>" prompt, lets now generate an entity relationship diagram (ERD) of the data model. Type in the following:

```
Generate an entity relationship diagram (ERD) of the data model in this application. Create a new doc called "DATA-MODEL.md"
```

Watch the output, watching out for any permissions you might be asked to approve. After a few minutes, you should have a new markdown document called "DATA-MODEL.md". Open this up on your machine and review the output.

These are just two examples of how Kiro CLI can generate more than text documentation. You can extend this to creating architecture diagrams, UML class diagrams, and much more.

---

### Generating API documentation

In this lab we are going to see how we can approach documenting APIs within your application. We are going to create a new custom agent with a specific prompt, and then use that to generate our API documentation.

**Task-07**

The first thing we are going to do is to copy a prompt that will help define the API custom agents. We will store this with the other prompts in the ".kiro/shared" directory.

```
cd .kiro/shared
wget https://raw.githubusercontent.com/094459/aqd-cli-workshop/refs/heads/main/resources/api-docs.md
cd ../..
```
Feel free to review this prompt. This is a simple starting prompt that helps shape the output we expect it to generate for us. You can see an example of a more detailed on [here](https://raw.githubusercontent.com/094459/aqd-cli-workshop/refs/heads/main/resources/api-docs-gen.md)

If you are not already in Kiro, start it using

```
kiro-cli
```

And from the ">" prompt, we are going to use the custom agent generation wizard so type the following.

```
/agent generate
```
You will then be asked three questions:

* agent name - enter "doc-api"
* agent description - enter "Custom agent for documenting APIs"
* agent scope - select LOCAL

It should look like this.

```
> /agent generate

✔ Enter agent name:  · doc-api
✔ Enter agent description:  · Custom agent for documenting APIs
✔ Agent scope · Local (current workspace)
```
When you hit enter after selecting Local, it will bring up the custom agent configuration file in the editor. Modify it so that it looks like the following:

```
{
  "name": "doc-api",
  "description": "Custom agent for documenting APIs",
  "prompt": "file://../shared/api-docs.md",
  "tools": [
    "*"
  ],
  "toolAliases": {},
  "allowedTools": [],
  "resources": [
    "file://AGENTS.md"
  ],
  "hooks": {},
  "toolsSettings": {}
}
```

Save and you should be greeted with the following message.

```
✓ Agent 'doc-api' has been created and saved successfully!
```

We are now ready to use this, so exit your Kiro CLI session and from the terminal enter:

```
kiro-cli --agent api-doc
```

From the "[doc-api]>" prompt, enter the following:

```
Document the APIs in this project.
```

After hitting return, follow the output from Kiro and provide input as it asks you for permissions. You should see things in the output that reflect how we defined our custom agent behind.

After about 5 minutes, Kiro CLI finishes. When I review what it has done, I can see that it has:

* generated an OpenAPI 3.0 spec for the API
* updated the code in the project to add API Key functionality
* implemented the Swagger UI to provide a nice web interface for the API
* created documentation in a file called API-DOCUMENTATION.md

When finished it should provide you with a URL to access the API documentation. In a new terminal, start the application using the following command:

```
uv run python run.py
```

After it starts, open up a browser and access http://127.0.0.0:5000/api/docs - you should see something like this appear.

![swagger UI for customer survey](/images/swagger-ui.png)

During testing I noticed that they had included an API which was not really an API, but was a route within Flask to provide the Dashboard. 

![Not a real API](/images/remove-api-before.png)

Once of the nice things is that it is easy to update your APIs. From the same Kiro CLI session I enter.

```
Remove the /dashboard from the Surveys APIs in the Swagger doc
```

And when I refresh the API docs, I now see its been updated.

![post removal by Kiro](/images/remove-api-after.png)

---

### Generating docs for brownfield projects

So far in these labs we have been working on the customer survey project, a greenfield application. You can use the same approach to document existing projects you have.

There are no labs in this section.

---

### Automating documentation tasks

Before we finish, I want to cover how you can use Kiro CLI to automate some of your documentation tasks.

#### Automating adding copyright headers

In the [Writing Code](/workshop/07-writing-code.md) (Task 6) you saw how you can [add a new steering file](/resources/ip.md) within your Kiro CLI session which would then automatically add copyright headers automatically to files created in your project.


#### Updating documentation automatically

You can also use agent hooks within your custom agents to potentially automate your documentation. In this next lab we will look at how you can set this up, adding a new hook that will automatically trigger a review and update of documentation as Kiro CLI works.

**Task-08**

We are going to create a new agent that we will use to make the updates, and configure this so that it has locked down permissions. We will then update one of the current custom agents we have been using in previous labs and configure a hook that will trigger at the end of a prompt cycle.

From your current Kiro CLI session, run this at the ">" prompt.

```
/agent generate
```

You will then be asked three questions:

* agent name - enter "doc-updater"
* agent description - enter "Updates README.md based on src/ changes"
* agent scope - select LOCAL

It should look like this.

```
> /agent generate

✔ Enter agent name:  · doc-updater
✔ Enter agent description:  · Updates README.md based on src/ changes
✔ Agent scope · Local (current workspace)
```
When you hit enter after selecting Local, it will bring up the custom agent configuration file in the editor. Review and update it so it looks like the following:

```
{
  "name": "doc-updater",
  "description": "Updates README.md based on src/ changes",
  "prompt": null,
  "tools": ["read", "write", "shell"],
  "allowedTools": ["read", "write", "shell"],
  "toolsSettings": {
    "write": {
      "allowedPaths": ["README.md"]
    },
    "shell": {
      "autoAllowReadonly": true,
      "allowedCommands": [
        "git status [^;&|]*",
        "git diff [^;&|]*"
      ]
    }
  }
}
```

Exit and save, you should be greeted with

```
✓ Agent 'doc-updater' has been created and saved successfully!
```

Lets explore what we just configured.

* we defined which tools this custom agent (doc-updated) will be able to use
* configured the tool permissions so that the custom agent can read changes about the project repo, but will also trust a few git commands
* configured tool permission so that it can write only to the README.md file

The next step is to update our existing custom agent, so again from the Kiro CLI prompt, we will edit the custom agent.

```
/agent edit --name doc-guru
```

Using the editor, update the custom agent configuration so that it looks like the following:

```
{
  "name": "doc-guru",
  "description": "Custom agent for documenting projects",
  "prompt": "file://../shared/documentation-guru.md",
  "tools": ["read","write","shell"],
  "allowedTools": ["read"],
  "resources": [],
  "hooks": {
      "stop": [
      {
        "command": "kiro-cli chat --non-interactive --agent doc-updater 'Review git diff src/ and update README.md if needed.'",
        "timeout_ms":120000
      }]
  },
  "toolsSettings": {}
}
```

So what have we changed? We have add a hook, that is configured for the "stop" event (this runs at the end of every cycle after entering a prompt). When this event happens, we run a command. In this instance we are actually invoking another instance of Kiro CLI, using the custom agent we created previous which has the permissions configured so it will not have issues with waiting for user confirmation. We are using the "--non-interactive" feature of Kiro CLI to run this command and then return control back to the shell. Finally we define the prompt we want to run - in this instance review changes and update the README - this is deliberately specific as we need to make sure that this will work within the constraints we have defined in the doc-updater custom agent.

We have also defined a timeout value, as the default of 30 secs typically expires for this task (in my experiments this typically takes 40-50 seconds)


Lets test this out now. Exit your current Kiro CLI session, and we will restart using the doc-guru custom agent.

```
kiro-cli --agent doc-guru
```

From the "[doc-guru]>" prompt, enter the following:

```
add a new feature in the About page that displays a MotD message
```

Kiro CLI will run through its normal process, so follow the output. You will be prompted as always for permission
After creating the initial updates (for which Kiro will still prompt you)

```
⢃⠀ 0 of 1 hooks finished
```

This will take approx 35-50 seconds to run, but once you have returned back to the prompt

```
✓ 1 of 1 hooks finished in 34.71 s
```
From a different terminal session, check out the README file. Review for "MotD" references, and you should find that this is now in the README - your docs have been updated without you explicitly asking it to.

There is one drawback to using hooks to update your documentation, and that it is a blocking task meaning that you have to wait until it finishes before you can use Kiro. If you are using a tool to auto-generate your documentation (sphinx, mkdocs, hugo) then these would be better options, especially as you can 


---

## Completed!

Head back to the [homepage](/README.md) and see what other labs you might want to try out.

