![Kiro CLI header](/images/kiro-workshop-header.png)

# Exploring Kiro CLI

In this lab we are going to start exploring Kiro CLI's agentic features. This lab assumes you have met all the pre-reqs and installed Kiro CLI [see here](/workshop/01-setup.md).

## Interactive (Agentic) Chat

When you start Kiro CLI, it provides an interactive chat mode that lets you have natural conversations with AI directly in your terminal. This feature brings the power of conversational AI to your commandline workflow. 

> It is worth knowing that Kiro CLI supports multiple languages, not just English.

---

**Task-01**

On your machine, create a new project directory and then start Kiro CLI. In my example, I am creating a directory in my home folder called "kiro-cli-workshop", but feel free to create this where ever you want:

```
cd
mkdir kiro-cli-workshop && cd kiro-cli-workshop
kiro-cli
```

You will see the ">" prompt which indicates it is ready for action. Pay attention to the ">" prompt, as this will provide you some additional hints when using it to do work. For example "!>" will tell you that you are running Kiro CLI with all tools trusted (don't worry about what this means for the moment, we will cover that soon enough).

We can type at the ">" and when we press enter, Kiro CLI will act upon whatever you have asked it to. Ask Kiro CLI the following prompt:

```
do you prefer tabs or spaces
```

Review the output, and take note of the ">" prompt. Now exit Kiro CLI - we do that either using the **"/q"** command, or by using CTRL + C - we will try the "/q" command:

```
/q
```

Now from the terminal, launch Kiro CLI using the following command:

```
kiro-cli chat "do you prefer tabs or spaces"
```

You should see that it does the same thing, although the output might be different (that's non deterministic systems for you!)

Leave this session open as we will be continuing to use it throughout these labs.

### Getting Help within Kiro CLI

At any time, you can use the "/help" command to get help. The help command works at the root level, as well as providing help for each command that Kiro CLI provides. If you get stuck, use it to provide you with more information.

### Available commands

Throughout these labs we will be using various Kiro CLI commands. Commands in Kiro CLI are prefixed with "/". 

You can use **"/help"** to list all available commands in Kiro CLI.

You can use CTRL + S to use a fuzzy search to find commands. When you press CTRL + S, as you type the list of available commands (with options) will filter out. To exit the fuzzy search either select the command you want or press ESC.

---

### Multi-line prompts

So far we have just tried a single line prompt in Kiro CLI chat, but you can enter multi line prompts in a number of ways. The first way is to press **CTRL and J** and you will be moved to a new line. Alternatively, you and add a backslash **"\\"** to the end of your line, and when you hit enter, you will start a new line. This is what most people who spend time at the terminal will be familiar with.

---

**Task-02**

From an Kiro CLI chat session, use the two methods to enter the following multi-line prompt:

```
When creating Python code, use the following guidance
- Use Flask as the web framework
- Follow Flask's application factory pattern
- Use environment variables for configuration
- Implement Flask-SQLAlchemy for database operations
```

Whilst using **CTRL + J** and **\\** works for adhoc multi-line prompts, you might find it easier enabling your favorite editor which we will look at next.

---

### Configuring your favorite text editor

When it comes to editors, its hard to beat vim. I know that others have a different perspective. The good news is that whatever is your favorite editor, you can configure Kiro CLI to use it in only a few steps.

By default, Kiro CLI will use your terminals default editor (vim, nano) but you can change this. If you want to specify which editor Kiro CLI will use, you will need to set an environment variable called EDITOR. If you wanted to use vim, I would enter the following in the terminal **before** starting Kiro CLI.

```
export EDITOR=vim
```

When I launch Kiro CLI, and now I can use the **"/editor"** command and it will launch vim for me, and I can edit the prompt until I am happy. When I save and exit, it returns me back to Kiro CLI, and it starts to work.

---

**Task-03**

Close your Kiro CLI session, and set your EDITOR environment variable to your preferred editor. Here are some options:

* export EDITOR=vi
* export EDITOR="code -w"
* export EDITOR="nano"

Feel free to try with your own preferred editor. 

Once you have done that, restart the Kiro CLI and run the **"/editor"** command. This should launch the editor you configured. Write a multi-line prompt, save and then exit. You should see Kiro CLI chat now processing your prompt.

---

### Using Reply

As you start entering prompts and working through tasks one handy time save is the **"/reply"** feature which you can enter at the ">" prompt, and it will launch the editor and add the previous conversation (and response) within the body of the message. 

---

**Task-04**

From your Kiro CLI session, at the ">" prompt, type the following:

```
show me some code that returns json date data
```

Now use the "/reply" command and see what happens. You should see that it launches your editor, and brings back the previous message which you can now editor and resubmit as required.

---

### Using images as part of your prompt

Kiro CLI supports images as part of your prompts. This means that you can refer to images within your project workspace as part of what you want Kiro CLI to help you with. For example, maybe you have some architecture diagrams you want to help build out some infrastructure as code, or you have a entity relationship diagram of a data model you want to build some SQL for, or maybe you have some sample design layouts you want to convert to some prototype code.

In the next lab we will take a look at how you can do this.

---

**Task-05**

In the resources directory you will find a file called [example-erd.png](/resources/example-erd.png). Copy this to your local directory into a directory called "data-model".

Start Kiro CLI and then from the **">"** enter the following prompt:

```
> Create a sample data model in python from the example-erd.png diagram in the data-model directory
```

Review the output. You should see how it is able to read the file, extract the key elements and then start generated code.

You can also use the **"/paste"** command to bring images into Kiro CLI. Lets do the same thing as before but this time, open up the image in your machines default image viewer and copy the contents of the image to the clipboard. Once you have done this, from the ">" prompt, type the following:

```
/paste
```

You should see output similar to the following:

```
Reading images: /var/folders/sn/h2yml_6n46vcq93q4c9qrzqh0000gr/T/.tmpFeCK60.png
 (using tool: read)
Allow this action? Use 't' to trust (always allow) this tool for the session. [y/n/t]:

```

You are being prompted for action. This is the first time we are seeing this, and we will explore this in more detail soon. For the time being enter "y" and review the output.

Leave your Kiro CLI session open, as we need it for the next lab.

---

### Exiting and resuming conversations

So far we have been using Kiro CLI within one session, and used some basic functionality to chat with Kiro. During your normal use you will likely exit your session but might want to resume where you left off. Kiro CLI support this with the **"--resume"** command.

---

**Task-06**

From the current Kiro CLI session, exit by typing "/q" in the command prompt. You should now be at your terminal shell. From the shell, start Kiro CLI with the "chat --resume" option as follows:

```
kiro-cli chat --resume
```

You should see the following appear:

```
Picking up where we left off..

You shared an ER diagram for a student enrollment system, and I identified the three tables (STUDENTS,
COURSES, ENROLLMENT) and asked what you'd like me to help you with.
```

This is a super handy feature that allows you to maintain your conversation context even if you have to leave your Kiro CLI session. What if you wanted to pick up from a different chat session, not just the last one you had? Don't worry, Kiro CLI has you covered. The "--list-sessions" command will list all your previous sessions.

```
kiro-cli chat --list-sessions

Chat sessions for /{program directory}/kiro-cli/:

Chat SessionId: 29b04f37-19de-42b6-b07a-bfcee3c288e6
  11 seconds ago | show, dont create, me a way in java to display the date at the cli | 1 msgs

Chat SessionId: 0389a798-b52a-45bb-9254-08cd6a8aaa16
  52 seconds ago | what is your name | 1 msgs

Chat SessionId: 41cd0688-5450-423f-9550-af9c5b710d26
  21 minutes ago | find the User class | 3 msgs

To delete a session, use: kiro-cli chat --delete-session <SESSION_ID>
```

This works across all your chat sessions within a given directory. You will see that the command says "Chat sessions for /{program directory}/kiro-cli/", so it displays all the chats that I have initiated from within this project directory. If I switch to a different directory:

```
kiro-cli chat --list-sessions
No saved chat sessions for /{project directory}/Projects
```

You can then use the "--resume-chat" command line switch to resume from a specific chat session, and then you will be able to select using the up and down arrows, which session to resume

```
kiro-cli chat --resume-picker
 Select a chat session to resume:
❯ 4 minutes ago | show, dont create, me a way in java to display the date at the cli | 1 msgs
  4 minutes ago | what is your name | 1 msgs
  25 minutes ago | find the User class | 3 msgs
```

---

### Saving you chat sessions

As you are working, there might be times when want to provide a quick summary of the current conversation you have been having within your Kiro CLI. Perhaps it has been a long session and you want to retain all the prompts and information so you can review at a later time. Maybe you want to generate some detailed context files.

You can now do this, using the **"/save {name}"** command, providing a name of the conversation you want to save. It will then generate a json document in the current directory.

```
> /save my-conversation

✔ Exported conversation state to my-conversation
```

If you want to specify a specific directory then you need to specify this, for example

```
> /save /{your home directory}/kiro-chats/my-conversation

✔ Exported conversation state to my-conversation
```

If you try and save a conversation to one tht already exists, you will see the following error:

```
File at customer-service already exists. To overwrite, use -f or --force
```

You can append the "-f" at the end of the command and this will now overwrite what you had before.

Saving your chat session is no good if you cannot use it to carry on. To do that, we use the **"/load {name}"** command, providing the name of the conversation we want to load up and resume.

```
> /load my-conversation

✔ Imported conversation state from my-conversation
```

---

**Task-07**

From a new Kiro CLI session, at the **">"** prompt lets generate some history by asking the following (feel free to use your own if you prefer)

```
I want to design a customer service application. Can you provide me with the top three things i need to think about
 What sort of data model would a customer service application need? Just show me, don't generate any code.
```

Now try saving your current conversation:

```
/save customer-service
```

After it has saved, open up a new terminal and review the output. 

Now close your Kiro CLI session and start it again. From the **">"** prompt, lets reload that conversation:

```
/load session-checkpoint
```

After it loads, ask it something about the customer service application (or if you did your own prompts, something about those)

You should notice that Kiro CLI is able to carry on where it left off.

---

### Choosing a model to use with Kiro CLI

You can select the model you want Kiro CLI to use. When you start Kiro CLI, the default model is "Auto". This configures Kiro CLI to use an intelligent model router that combines multiple frontier models with advanced optimization techniques. You can see which model you have when you start Kiro CLI, as this will be displayed below the Kiro splash logo.

```
Model: Auto 
```

You can change this to use a specific model however, using the **"/model"** command and then use the arrow keys and return to select the mode you want.

```
> /model
 Press (↑↓) to navigate · Enter(⏎) to select model
❯ Auto (current)| 1x credit | Models chosen by task for optimal usage and consistent quality
  claude-sonnet-4.5 | 1.3x credit | The latest Claude Sonnet model
  claude-sonnet-4  | 1.3x credit | Hybrid reasoning and coding for regular use
  claude-haiku-4.5 | 0.4x credit | The latest Claude Haiku model
```

You can also set this two other ways:

* when you start your Kiro CLI session, providing an argument in the format of **"kiro-cli chat --model <model-name>"**
* configure the default model in your Kiro CLI settings file

You can check out the [kiro docs](https://kiro.dev/docs/cli/chat/model-selection/?trk=fd6bb27a-13b0-4286-8269-c7b1cfaa29f0&sc_channel=el) to see the available models to you, as well as get some additional info about how to choose a model.

For many tasks, leaving the default Auto is a good choice.

---

**Task-08**

If you are not already in a chat session, start Kiro CLI, and from the **">"** prompt, type in the following:

```
/model
```

Change the Model and try some Prompts. Exit the chat session and then from the command line type the following:

```
kiro-cli chat --model claude-4-sonnet
```

You should see that you are now using Claude 4 Sonnet. 

Exit the chat session and then restart Kiro CLI. The Model should now have reverted back to the default Model.

We can configure the default Model by setting this in the Kiro CLI configuration file. Exit Kiro CLI again, and from the terminal and using your favorite editor, make the following addition.

```
vi ~/.kiro/settings/cli.json
```

The file might be empty (if this is the first time you are using Kiro CLI), but that is ok. If you already have stuff in there, add this as the last entry.

```
 "chat.defaultModel": "claude-4-sonnet"
```

Save the file and now restart Kiro CLI. You should notice that it has changed the model.

---

### Executing commands

So far we have been using Kiro CLI in chat mode to write prompts and then get responses. Sometimes you might want to run a command from within your chat session. You can use the **"!"** to preface any command you want to run, and it will execute the command and then return the results back to you in the chat session. There are two things you need to bear in mind When running commands. 

* Each invocation is stateless. If I use "! cd {directory}" and then run "! ls" I will not get the directory listing for the {directory} but for the current Kiro CLI directory where you launched it. If you need to do that you should concatenate your commands, so using this as an example "!cd {directory} && ls".

* The output of the commands you run is added to the context window. If you run command that generate a lot of output, you will potentially use up your context window more quickly. (If you are not sure what "context window" is, don't worry we are about to get to that.)

---

**Task-09**

Open up Kiro CLI and run the following commands:

```
! ls -altr
```

Review the output - you should see your current directory listed out. Experiment with a few other commands.

---

### Managing prompts

Kiro CLI allows you to save and then re-use "prompts". What is a prompt? A prompt is a markdown file that contains instructions you want to provide Kiro CLI. You might have developed over a period of time a set of prompts that you re-use, and so having a way to easily re-use this from within Kiro CLI is super handy.

You can define prompts at a global level or just at the project workspace level. If there is a clash, local scoped prompts will be used.

From Kiro CLI's ">" I can see available prompts by using the **"/prompts list"** command, which will tell me what prompts are available and whether these are at the global or project workspace level. When I run this command on my machine, I get the following:


```
> /prompts list

Usage: You can use a prompt by typing '@<prompt name> [...args]'

Prompt                         Description                              Arguments (* = required)
▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔
Global:
/{your home directory}/.kiro/prompts/Bootstrap-app-generation.md
/{your home directory}/.kiro/prompts/Generate-Entity-diagrams.md
/{your home directory}/.kiro/prompts/Add-UML-Sequence-diagram.md

Local:
/{your home directory}/{your project directory}/.kiro/prompts/proj-setup.md
```

From Kiro CLI, I can use these prompts by using the **"@"** command followed by the name of the prompt (but without the tailing.md). So for example

```
> @Add-UML-Sequence-diagram

Review the project in this workspace and then add a detailed UML sequence diagram that shows how:

- users login to the application
- user flows through the application
- user logout
...
...
```

Kiro CLI provides you with the ability to create, edit, and view prompts from within your session (at the global or project workspace level). We will take a look at those during the lab.

---

**Task-10**

From your Kiro CLI session, at the ">" prompt, type the following:

```
/prompt create --name proj-setup
```

Which will open up the editor. Add the following text:

```
You like to code in Python
You prefer Flask web application framework
Use tabs not spaces
Use PostgreSQL for all data
```

And then save the file. From the ">" type the following:

```
/prompts list
```

You should now see that you have a new saved prompt, under the local section. By default when you create a prompt, it will create it in your local project workspace. If you want to create a global prompt, then you can use the "--global" argument.

We can view what is in these prompts (especially handy over time as you begin to get a collection of these). You can try this by using the following command at your ">"

```
/prompts details proj-setup
```

which should give you output similar to the following:

```
> /prompts details proj-setup


Prompt Details
▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔

Name: proj-setup
Source: /{your home directory}/{your project directory}/.kiro/prompts/proj-setup.md

Usage: @proj-setup

Content Preview:
  You like to code in Python
  You prefer Flask web application framework
  Use tabs not spaces
  Use PostgreSQL for all data

```

Finally you can edit these prompts if you want to make some adjustments. From your ">" prompt:

```
/prompts edit proj-setup
```

Which will bring up the editor again allowing you to make any changes. Ok we are now ready to run this, which you can do from the ">" prompt:

```
@set-preference
```

Review the output before proceeding.

---

## Tools

As you begin to use Kiro CLI, you are going to want to get Kiro CLI to interact with the wider world. Kiro CLI has a concept of **Tools** that allow it to do this: reading and writing files, run commands, and more. To make sure that we have control, Kiro CLI has the concept of permissions when using **Tools**, allowing you to set the level of trust. This is important as setting levels of trust allows you to streamline automation of running those commands (so that you are not forever being asked for permission!)

Kiro CLI configures **Tools** as **"Trusted"** or **"Untrusted"**. What this means is that when Kiro CLI wants to use one of these Tools, if a Tool is "Untrusted" you will be prompted to ask for permission to proceed with running that Tool. You can either trust one time, trust it going forward, or decline.

When you start Kiro CLI, there are a number of built in tools. We can see this when we run the **"/tools"** command:

```
> /tools

Tool              Permission
▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔
Built-in
- shell           not trusted
- read            trust working directory
- write           not trusted
- glob            trust working directory
- grep            trust working directory
- introspect      trusted
- report          not trusted
- aws             trust read-only commands
- web_fetch       not trusted
- web_search      not trusted

```

You will notice that we have one tool that is automatically trusted - **introspect**. We have four tools that are trusted with caveats, with **aws**, **glob**, **grep**, and **read** constrained to the current working directory and read only commands.We can see writing and executing files (shell) are not trusted. What this means is that if we ask Kiro CLI to do something that wants to use these tools, it is going to prompt us for permission. Lets see this in action. We also have **web_fetch** and **web_search** tools that are disabled by default. 

The number of tools that appears will change based on a number of factors:

* whether you have configured any Kiro CLI experimental features [(see the advanced setup for more details about that)](/workshop/02-advanced-setup-topics.md)
* new features introduced by Kiro CLI updates (for example, v1.21.0 added web_fetch and web_search) - the best source of know what available tools exist is [the built in tools reference pages](https://kiro.dev/docs/cli/reference/built-in-tools/?trk=fd6bb27a-13b0-4286-8269-c7b1cfaa29f0&sc_channel=el)
* the number of Model Context Protocol (MCP) servers you configure - each MCP server will have it's own tools (don;t worry if you are not sure what MCP servers are, we will explore those in a bit)


By default, Kiro CLI auto trusts the built in Tools that perform "Read" operations, but does not trust any tool that needs to write to the file system or execute a command or program. Kiro CLI gives you the ability to control though. Lets take a look at this, exploring Tools permissions and then seeing how this work by extending the previous task.

When you trust a tool, that trust is valid for the duration of your Kiro CLI session. Once you exit and restart, the permissions will reset. Later on in this lab we will see how you can change that behavior.

---

**Task-11**

Open up a Kiro CLI chat session. From the **">"** prompt, type the following:

```
/tools
```

You should see something like the following:

```
> /tools

Tool              Permission
▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔
Built-in
- shell           not trusted
- read            trust working directory
- write           not trusted
- glob            trust working directory
- grep            trust working directory
- introspect      trusted
- report          not trusted
- aws             trust read-only commands
- web_fetch       not trusted
- web_search      not trusted

```

Enter the following prompt:

```
> Write a script that prompts the user for their name and then shares a Yoda wisdom with them
```

Your output will likely be different to the output below, but the important thing here is that you see that you are being asked for permission to use a specific tool. The tool is mentioned in the first line, and then at the bottom you can see your options.

```
I'll create the following file: /{project directory}/kiro-cli/workshop/yoda_wisdom.py **(using tool: write)****
Purpose: Create Python script that prompts for name and shares Yoda wisdom

+     1: import random
+     2:
+     3: name = input("Your name, what is? ")
+     4:
+     5: wisdoms = [
+     6: 	"Do or do not. There is no try.",
+     7: 	"Fear is the path to the dark side.",
+     8: 	"In a dark place we find ourselves, and a little more knowledge lights our way.",
+     9: 	"Wars not make one great.",
+    10: 	"You must unlearn what you have learned.",
+    11: 	"Size matters not. Judge me by my size, do you?"
+    12: ]
+    13:
+    14: print(f"\n{name}, {random.choice(wisdoms)}")

**Allow this action? Use 't' to trust (always allow) this tool for the session. [y/n/t]:**
```

Enter **"t"** and hit return. You should see it complete the creation of the script. Exit Kiro CLI, review the script and see if it works.

If we exit Kiro CLI and go back, and repeat the process you should be prompted again. The trust settings have reset back the the default trust settings. We will look more about Tools and Trust in a later lab.

---

## Context

Context files contain information you want Kiro CLI to consider during your conversations. These can include project requirements, coding standards, development rules, or any other information that helps Kiro provide more relevant responses. Understanding how to manage context is critical to getting good output from tools like Kiro CLI. 

Whilst it might be tempting to think that adding everything you have as context will lead to better results, research done by Stanford University tells us different. Overloading context can lead to degraded output, so it is really important you spend time and fine tune our efforts.

Context is **not** and infinite resource, and needs to be managed. Something called the **"context window"** defines the amount of context that foundation models can work with. This a finite resource measured in tokens, which are the unit of text processing that large language models use. Currently Kiro CLI has a maximum context window of 200K tokens.

---

### Managing Context

As you start your Kiro CLI session and start to work, sometimes you will want to add context to help with the task at hand. Other times you want start your session with some defined context. Kiro CLI allows you to easily configure both, as we will see in a moment.

Kiro CLI also has a concept of persistent knowledge about your project through markdown files. These are called **"steering docs"**, and are located in a specific directory (.kiro/steering/). Instead of explaining your conventions in every chat, steering files ensure Kiro consistently follows your established patterns, libraries, and standards.

> In the advanced section we will look at experimental features of Kiro CLI that provide additional mechanisms for you to manage your context.

In the next lab we will show you the different options you have for managing context during your sessions.

---

#### Showing the current Context

At any time you can see what your Kiro CLI session has configured for context by using the **"/context show"** command.

---

**Task-12**

We can see the current status of our configured context by using the **"/context show"** command. From the Kiro CLI chat session, at the ">" prompt type

```
> /context show
```

You should see output similar to the following:

```
> /context show

Agent (kiro_default)
  - AmazonQ.md (no matches)
  - AGENTS.md (no matches)
  - README.md (no matches)
  - /{your home directory}/.kiro/steering/**/*.md (no matches)

Session (temporary)
  <none>

No files in the current directory matched the rules above.
```

So what is this showing us. A few things:

* We can see our temporary context for our permanent context
* We can see that Kiro CLI automatically looks for and include in context certain files in the current project directory (AmazonQ.md, AGENTS.md, and README.md) - these are the persistent context files that are from when Kiro CLI was Amazon Q CLI
* We can see that it also looks to include any steering documents it finds (in the .kiro/steering/ directory and subdirectories)

---

#### Clearing context files

There may be times when you want to clear all the context that has been configured within your current Kiro CLI session. You can do this with the **"/context clear"** command.

Once you exit and then restart your Kiro CLI session the **persistent** context files will come back, but any temporary, session context files will not.

---

**Task-13**

From your Kiro CLI session, at the ">" prompt type the following:

```
/context clear
```

You should see the following output:

```
Cleared context
Note: Context modifications via slash command is temporary.
```

Run the **"/context show"** command. What has happened? Exit Kiro CLI and restart it, and then re-run the command. Did the context files come back?

---

#### Context usage

You can check your current **"context window"** by using the **"/context"** command. 

---

**Task-14**

From your Kiro CLI session, at the ">" prompt, type

```
> /context
```

You should see output similar to the following.

```
> /context

Current context window (1.1% used)
|███████████████████████████████████████████████████████████████████████████████ 1.1%

█ Context files 0.0%
█ Tools 1.1%
█ Kiro responses 0.0%
█ Your prompts 0.0%

```

Not surprisingly, our context window is looking healthy. As you go through this workshop, run this command from time to time and see how this changes.

Check the Advanced Topics for an experimental feature that provides a visual cue for your context window capacity that is super handy.

---

#### Clearing your context window

There maybe times when you want to empty your context window and start from fresh. Kiro CLI provides a command to help you do this. **"/clear"** will empty your current session's context window. **This should be used with care** as this will clear your current context window and cannot be undone.

You will be prompted whether you really want to do this.

```
Are you sure? This will erase the conversation history and context from hooks for the current session. [y/n]:
```

*No Lab in this section*

---

#### Compacting

If you do start to reach the limits of your context window, you can compact your context. The way this works is that Kiro CLI will generate a summary of everything in the current context window. Once this summary has been completed, it will replace your current context.

Kiro CLI checks to see what the status of your context window is when running this command, and if it feels this is not needed it will generate the following output:

```
Conversation too short to compact.
```

The default behavior is that Kiro CLI will automatically try and compact as you reach the limit of your context window. You might want to control this behavior, and can do this by setting the following in your Kiro CLI settings.json.

```
q settings chat.disableAutoCompaction true
```

After you run this command, compaction will now not automatically occur and you can control this within your Kiro CLI Session. When that event occurs, you will be notified with a message at which point you can then decide what you want to do.

You will need to determine what best works for your developer flow. For some, they prefer that Kiro CLI automatically compacts, whereas others want to be in control of that.

*No Lab in this section*

---

#### Adding and removing session context

You can temporarily add files to your current chat session using the **"/context add"** command. These additions are only available for the current session and will not persist when you start a new chat session. You can add single documents or add multiple using global file patterns. For example, both of these would add a single or multiple markdown files to the current session context.

```
/context add README.md
Added 1 path(s) to context.
Note: Context modifications via slash command is temporary.
```

and

```
> /context add ~/context-repo/2-sample-data/*.md

Added 1 path(s) to context.
Note: Context modifications via slash command is temporary.
```

When I run "/context show" we can see these are now available as context for this session

```
> /context show

Agent (kiro_default)
  <none>

Session (temporary)
  /{your home directory}/{your project directory}/README.md
  /{your home directory}/context-repo/2-sample-data/AGENTS.md
  /{your home directory}/context-repo/2-sample-data/data-bod.md

3 matched files in use
- /{your home directory}/context-repo/2-sample-data/AGENTS.md (0.2% of context window)
- /{your home directory}/context-repo/2-sample-data/data-bod.md (0.2% of context window)
- /{your home directory}/{your project directory}/README.md (0.5% of context window)

Context files total: 0.9% of context window
```

Kiro CLI will tell you when it has a "match", and what this means is that it has found the file and loaded it into its context window.

When using this command to add context files, Kiro CLI will check to see if they exist. If you are in a situation where you want to add the files later, you can force the adding of context using the "-f" argument.

```
/context add {doc} -f
```

---

**Task-15**

Exit your Kiro CLI session, and create a new markdown file with the following text:

```
You like to code in Python
You prefer Flask web application framework
Use tabs not spaces
Use PostgreSQL for all data
```

Save this file as "tmp-context.md". 

Restart your Kiro CLI session, and from the ">" use the following command to add this into your session context.

```
/context add tmp-context.md
```

Verify it works, and check your current context using the context show command.

---

## Model Context Protocol (MCP)

We have just seen some of the ways you can add context to your Kiro CLI session. Model Context Protocol (MCP) extends Kiro CLI's capabilities by connecting to specialized servers that **provide additional tools and context**.

> If you want to dive deeper into MCP, check out [this section](/workshop/08-mcp.md)

MCP Server expose different capabilities, so it is important to understand before you connect Kiro CLI to an MCP Server what capabilities it will provide. 

Many MCP Servers will expose Tools, and as you have seen in previous labs, these will be visible from within your Kiro CLI sessions when you use the **"/tools"**. 

Any tools that an MCP Server make available will untrusted by default. You will be able to change that in the same way as you saw in the previous labs.

So how do we add MCP Servers to our Kiro CLI session? Before we show you, we need to introduce Kiro CLI Custom Agents.

---

## Custom Agents

Custom agents provide a way to customize Kiro behavior by defining specific configurations for different use cases. Each custom agent is defined by a configuration file that specifies which tools the agent can access, what permissions it has, and what context it should include.

By default, Kiro CLI provides access to all available tools but requires user confirmation for most operations. As you add MCP servers that provide additional tools, the number of tools you have to manage increases significantly.  This approach prioritizes security but can interrupt your workflow with frequent permission prompts. Custom agents solve this by allowing you to:

* Pre-approve specific tools - Define which tools can run without prompting
* Limit tool access - Restrict which tools are available to reduce complexity
* Configure tool behavior - Set specific parameters for how tools should operate

We have also seen in the previous section, that we can add context at the session level but what if we wanted to have context available at the start of our session and not have to add it each time. And of course, just like tools, we will probably want different context files for different tasks we are working on. Custom agents solve this by allow us to:

* Include relevant context - Automatically load project files, documentation, or system information

### Global vs Project custom agents

Custom agents can have either a global or project scope.

* Global custom agents are available across all projects and directories on your system. JSON configuration for global agents are located at **"~/.kiro/agents/{agent-name}.json"**

* Project-level custom agents are available only within the specific project directory. JSON configuration for project level custom agents are in the current directory, in the **".kiro/agents/{agent-name}.json"**

Global custom agents are global for a specific installation of Kiro CLI (i.e. the developer machine). On that developer's machine, I will be able to use global custom agents every time I start Kiro CLI. Project custom agents reside within the project workspace, so can potentially work across different developers and different developer environments. If your project is using source control, the custom agents are version controlled like other application artifacts, meaning that any developer who checks out the code can access any custom agents defined within that checked out repo.

What if you have local, project custom agents and global custom agents? When Kiro CLI looks for an custom agent, it follows a specific precedence order: First it will look for **Local custom agents first**, checking for custom agents in the current working directory. Next it will look for **Global custom agents**, and fall back to custom agents in your home directory. If it fails to find any, it will use the **Built-in default** custom agent.

If both local and global directories contain custom agents with the same name, the local custom agent takes precedence. Kiro CLI will display a warning message when this occurs:

```
WARNING: Agent conflict for my-agent. Using workspace version.
```

#### Starting Kiro CLI with a custom agent

When we start Kiro CLI, it defaults to a custom agent configuration (**kiro_default**). We can see this by running the following command:

```
kiro-cli agent list

* kiro_default    (Built-in)
* kiro_planner    (Built-in)
```

> If you are wondering what the **kiro_planner** is, this is used by Kiro when you go into plan mode using /plan - don't worry about this yet!

A custom agent is a json configuration file which we create and then configure for what need (we will see what this looks like in a moment). Once we have created it, we can then use it when we start Kiro CLI.

You start your Kiro CLI chat with a custom agents by using the **"--agent {name-of-agent}"** argument.

For example, if you created a custom agent called "php-dev", you would start your Kiro CLI session with the command 
**"kiro-cli --agent php-dev"** . This then starts your Kiro CLI session using the configuration options defined in that custom agent. You know whether you are using a custom agent because the Kiro CLI prompt will change. 

Kiro CLI provides a visual cue as to which custom agent your current session is using. When you are using the default custom agent, the prompt is **">"**. In the previous example, if we had started Kiro CLI with the "--agent php-dev" argument, our Kiro CLI prompt would change to **"[php-dev] >"**.

Let's create our first custom agent to see this in action.

---

**Task-16**

We are going to create our first custom agent, configure some MCP Servers and add some additional files we want as context. We are going to create this with a local, project workspace scope (not global).

> We will be diving much deeper into custom agents in the advanced part of this workshop, but before we do that we need to get comfy with the basics.

From your Kiro CLI session, at the ">" prompt, type the following

```
> /agent create -d . --name python-dev
```

the "/agent create" is the Kiro CLI command to create a custom agent. You can use "/agent create --help" to see the various options. We are using the "-d" argument to specify the directory in which to create the custom agent. The "." represents the current directory, which means we are creating a local project workspace custom agent. If we had left this out, it would default to creating a global custom agent, storing this in the "~.kiro/agents" directory. Finally we provide the custom agent with a name.

When you hit enter, Kiro CLI will bring up the configured editor, and you should see something like this:

```
{
  "name": "python-dev",
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

Don't worry too much about the details in this configuration file. We will be covering these in detail later, so for the time being do not make any changes, just save this file. When you exit the editor, you should see something like the following:

```
Agent python-dev has been created successfully
Changes take effect on next launch
```

From Kiro CLI's ">" prompt type the following to see all the commands we have available for working with custom agents:

```
/agent --help
```

If we run the "/agent list" command now, we should see that we have our new custom agent listed. From your ">" prompt try the following:

```
> /agent list
```

Which should generate output like the following:

```
  kiro_default    (Built-in)
  kiro_planner    (Built-in)
  python-dev      /{your home directory}/{your project directory}/.kiro/agents
```

You can edit our custom agent at any time using the "/agent edit --name {custom agent}" command. From your ">" prompt try the following:

```
> /agent edit --name python-dev
```

Which will switch you back to the editor where you can make any changes you want. Exit your editor to return back to the Kiro CLI.

Lets now switch our Kiro CLI session to use this custom agent. To do that we use another command, "/agent swap {custom agent}. From the ">" Kiro prompt, type the following:

```
> /agent swap python-dev
```

Review the prompt, you should see that it has changed to "[python-dev] >". If you create multiple custom agents you can switch between them using this command.

Finally you can start your Kiro CLI session with a specific custom agent by using the "--agent {custom agent}" argument. Exit your current Kiro CLI session, and from the terminal type the following:

```
kiro-cli --agent python-dev
```

Which will start your Kiro CLI session with that custom agent. Now this might be a pain to do every time, so in the next section we will look at how to change the behavior of Kiro CLi when it starts.

---

#### Recovering for bad configuration changes

Whilst you are editing the custom agent configuration files, you may make an error or mistake in your edits. When Kiro CLI saves custom agent configurations, it will check and let you know.

Here is an example of one such error - when making a change, I had created a duplicate field.

```
Error: Json supplied at /{path}/.kiro/agents/python-dev.json is invalid: duplicate field `allowedTools` at line 24 column 16

Failed to execute command: Post edit validation failed for agent 'python-dev'. Malformed config detected: Json supplied at /{path}/.kiro/agents/python-dev.json is invalid: duplicate field `allowedTools` at line 24 column 16

Failed to execute command: duplicate field `allowedTools` at line 24 column 16
```

If you try and start Kiro CLI it will fail, so you will need to fix these issues by directly editing the custom agent configuration file. The path will be displayed in the error message, so use that to locate the file and then fix the issue.

---

#### Setting a default custom agent

When you start your Kiro CLI session, it will start the session using the default custom agent. If you create a number of different custom agents, you may decided that you want to make that the default one so that you don't always have to start your Kiro CLI session using the "--agent {name}" command.

You can change the default custom agent by using the **"/agent set-default"** command. Lets take a look at that next.

**Task-17**

Exit and restart your Kiro CLI session without any arguments so you are starting using the default custom agent. From the **">"** prompt, enter the following:

```
/agent set-default -n python-dev
```

You should see something like the following:

```
[python-dev] > /agent set-default -n python-dev
✓ Default agent set to 'python-dev'. This will take effect the next time q chat is launched.
```

No exit your Kiro CLI session and restart it - again without using any arguments. What happens? When you use this command, it writes into the Kiro CLI settings file. You can see this by running the following command from the terminal

```
kiro-cli settings list
```

You will see this line appear at the end of your config file:

```
chat.defaultAgent	"python-dev"
```

Edit the file and remove this line - for the next lab we do not want to start with this custom agent automatically.

---

#### Deleting custom agents

To delete a custom agent you have to delete the custom agent json configuration file. Depending on whether this is a local project workspace custom agent or global one, will determine which directory you will find your agent.

Once you delete the configuration file, Kiro CLI will not show it or be able to switch to using it.

*No labs in this section*

---

### Adding a prompt to our custom agent

So far we have not actually done anything useful with our custom agent. In this lab we are going to look at how we can create custom agents that have a "persona" which we can define via a prompt. The prompt field is intended to provide high level context to the agent, similar to a system prompt. It supports both inline text and file:// URIs to reference external files.

For the custom agent we created in previous labs (python-dev) it might be useful to start that custom agent with the following prompt (this is a deliberately simplified example for this workshop)

```
You are a Python developer. You have a preference for Flask web framework. You comply with PEP-8 when writing code
```

With Kiro CLI we can add a prompt to a custom agent that will be used when we switch to using that custom agent. We do that by making changes to the custom agent configuration file.

**Task-18**

From the Kiro CLI session, at the ">" prompt lets edit the custom agent we created

```
/agent edit --name python-dev
```

This will bring up the editor. Edit the file to add the above prompt ("ou are a Python developer. You have a preference for Flask web framework. You comply with PEP-8 when writing code") in the prompt field. It should look like the following:

```
{
  "name": "python-dev",
  "description": "",
  "prompt": "You are a Python developer. You have a preference for Flask web framework. You comply with PEP-8 when writing code.",
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

Save and close the file. 

Now close your Kiro CLI session and restart it - it should start using the default custom agent (if it starts with the [python-dev], go back to the last item in the previous lab and delete the default custom agent setting from your cli.json file).

Type the following at the ">" prompt

```
show me some code that will display hello world
```

Review the output - it should provide some generic advice, possibly in different programming languages.

Now from the ">" prompt, switch to the python-dev custom agent using the following command:

```
/agent swap --name python-dev
```

You will see that the prompt has changed to [python-dev]. Now try the same prompt at the ">"

```
show me some code that will display hello world
```

What has changed? You should see that we have changed how this custom agent behaves by adding those details in the prompt. We will come back to this in the advanced topic and see how we can use Kiro CLI to automatically generate this for us.

---

### Adding Context in your custom agent

Being able to add context to our Kiro CLI sessions is critical to get the best output. So far we have seen how we can add and remove session based context, but in this section we are going to see how we can add context when we start our Kiro CLI session. We do that by:

* adding context information in markdown files
* making changes to the custom agent configuration to point to those files

Kiro CLI's lets us add context as "resources" in the custom agent configuration file. When we make changes to add context, those resources can include: specific files (e.g. file://python-dev-standards.md) or glob patterns for multiple files (e.g.file://.kiro/steering/*.md).

Kiro CLI adds some resources (context) automatically when creating custom agents. You might have noticed this in the previous labs when we looked at the custom agent configuration.

```
{
  "name": "python-dev",
  "description": "",
  "prompt": "You are a Python developer. You have a preference for Flask web framework. You comply with PEP-8 when writing code.",
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

We can see that Kiro CLI will automatically load AGENTS.md and README.md into context. AGENTS.md is a common standard used across many AI coding assistants.

When the custom agent starts it will look for these resources, and if they are present it will load them. We can see this by using another command we used earlier **"/context show"**.

```
[python-dev] 2% > /context show

Agent (python-dev)
  - AGENTS.md (no matches)
  - README.md (no matches)

Session (temporary)
  <none>

No files in the current directory matched the rules above.
```

We can see that we have **"no matches"** which means Kiro CLI has not found these files. This is a useful command to make sure that you understand what your Kiro CLI session is using when doing a task.

---

**Task-19**

In this lab we are going to update our custom agent context, adding some additional resources that will help Kiro CLI generate code that we prefer.

Exit Kiro CLI, and in your project directory create a directory called "steering" in the ".kiro" directory. Copy the resource from this workshop ([python-dev.md](/resources/python-standards.md)) into this directory. Your project directory should look like the following

```
├── .kiro
   ├── agents
   │   └── python-dev.json
   ├── prompts
   │   └── proj-setup.md
   └── steering
       └── python-standards.md
```

Start Kiro CLI **without** using a custom agent, and from the ">" run the following command:

```
/context show
```

You should see something similar to the following:

```
Agent (kiro_default)
  - AmazonQ.md (no matches)
  - AGENTS.md (no matches)
  - README.md (no matches)
  - /{your home directory}/.kiro/steering/**/*.md (no matches)
  - /{your home directory}/{your project directory}/.kiro/steering/**/*.md /{your home directory}/{your project directory}/.kiro/steering/python-dev.md

Session (temporary)
  <none>

1 matched file in use
- /{your home directory}/{your project directory}/.kiro/steering/python-standards.md (0.3% of context window)

Context files total: 0.3% of context window
```

Kiro CLI's default agent has picked up our context resource (python-standards.md) from the steering directory. Kiro CLI lets us know there is one match, so Kiro has loaded this into its context window. This is as expected as the default behavior for Kiro CLI is to look at the following files:

- AmazonQ.md
- AGENTS.md
- README.md)
- /{your home directory}/.kiro/steering/**/*.md
- /{your home directory}/{your project directory}/.kiro/steering/*.md

As we copied our python-dev.md file to the .kiro/steering directory, Kiro has picked this up. Lets switch to our custom agent now.

```
/agent swap python-dev
```

run the same command. What do you notice that is different? This is the output you should see:

```
[python-dev] 2% > /context show

Agent (python-dev)
  - AGENTS.md (no matches)
  - README.md (no matches)

Session (temporary)
  <none>

No files in the current directory matched the rules above.
```

This time we have no matches. This is because the configuration of our custom agent only has two rules defined to look for context (AGENTS.md and README.md). Lets change this. Edit the python-dev custom-agent using the following command:

```
/agent edit --name python-dev
```

And change the resources section to include a new rule to look for all files in the steering directory. You configuration should look like this:

```
{
  "name": "python-dev",
  "description": "",
  "prompt": "You are a Python developer. You have a preference for Flask web framework. You comply with PEP-8 when writing code.",
  "mcpServers": {},
  "tools": [
    "*"
  ],
  "toolAliases": {},
  "allowedTools": [],
  "resources": [
    "file://AGENTS.md",
    "file://README.md",
    "file://.kiro/steering/*.md"
  ],
  "hooks": {},
  "toolsSettings": {},
  "useLegacyMcpJson": true,
  "model": null
}
```

After saving, you will notice the following that appears:

```
Agent python-dev has been edited successfully
Changes take effect on next launch
```

Exit Kiro CLI and then restart, this time starting the python-dev custom agent.

```
kiro-cli --agent python-dev
```

Now look at the context using the following command:

```
/context show
```

What has changed? Your output should look similar to the following:

```
[python-dev] 2% > /context show

Agent (python-dev)
  - AGENTS.md (no matches)
  - README.md (no matches)
  - .kiro/steering/*.md /{your home directory}/{your project directory}/.kiro/steering/python-dev.md

Session (temporary)
  <none>

1 matched file in use
- /{your home directory}/{your project directory}/.kiro/steering/python-dev.md (0.3% of context window)

Context files total: 0.3% of context window
```

In this lab we have shown you how you can update your custom agent to add context resources based on your needs. This is something that you will spend a lot of time doing as you starting using Kiro CLI in your day to day activities.

---

### Adding MCP Servers in your custom agent

Next we are going to see how we can add MCP Servers within Kiro CLI. We do this within the custom agent configuration file. We are going to use a couple of MCP Servers in this lab, which you can [find here at the official AWS MCP Server website](https://awslabs.github.io/mcp/).

We are going to build upon the previous custom agent we created, "python-dev" and add the [AWS Documentation MCP Server](https://awslabs.github.io/mcp/servers/aws-documentation-mcp-server). Clicking on that link will show you how to configure this MCP Server as well as other details such as what tools, resources, and prompts it provides.

> **Note!** Remember before adding any MCP server, to do a thorough validation of the code. Check out [this](/workshop/08-mcp.md) for more details about the risks associated.

When we look at the AWS Documentation MCP Server documentation page, we can see that it provides details of how to configure it for your AI coding tool.

```
"awslabs.aws-documentation-mcp-server": {
      "command": "uvx",
      "args": ["awslabs.aws-documentation-mcp-server@latest"],
      "env": {
        "FASTMCP_LOG_LEVEL": "ERROR",
        "AWS_DOCUMENTATION_PARTITION": "aws",
        "MCP_USER_AGENT": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/131.0.0.0 Safari/537.36"
      },
      "disabled": false,
      "autoApprove": []
    }
```

We will use a simplified version of this for the workshop

```
"awslabs.aws-documentation-mcp-server": {
      "command": "uvx",
      "args": ["awslabs.aws-documentation-mcp-server@latest"],
      "disabled": false
    }
```

As we will be running these MCP Servers locally, you will need to make sure that you have the necessary tools. In this instance we need to make sure that you have [uv](https://github.com/astral-sh/uv) as this is what will be used to download and run the Python code. From a terminal, confirm that you have installed and that running "uv --version" returns a version number and not an error. 

---

**Task-20**

From your Kiro CLI session, edit the python-dev custom-agent using the following command:

```
/agent edit --name python-dev
```

Add the MCP Server configuration details into the custom agent configuration file. After editing it, it should look like the following:

```
{
  "name": "python-dev",
  "description": "",
  "prompt": "You are a Python developer. You have a preference for Flask web framework. You comply with PEP-8 when writing code.",
  "mcpServers": {
        "awslabs.aws-documentation-mcp-server": {
        "command": "uvx",
        "args": ["awslabs.aws-documentation-mcp-server@latest"],
        "disabled": false
        }
  },
  "tools": [
    "*"
  ],
  "toolAliases": {},
  "allowedTools": [],
  "resources": [
    "file://AGENTS.md",
    "file://README.md",
    "file://.kiro/steering/*.md"
  ],
  "hooks": {},
  "toolsSettings": {},
  "useLegacyMcpJson": true,
  "model": null
}
```

Save and exit. We now need to restart Kiro CLI, with the python-dev custom agent to see the effects of this. Start Kiro CLI with the following command:

```
kiro-cli --agent python-dev
```

Pay attention to what happens at the top of the splash screen. You should see something like the following appear:

```
⠏ 0 of 1 mcp servers initialized. ctrl-c to start chatting now
```

Kiro CLI is loading the MCP Server we just configured. After about 10 seconds this should finish, and you will be back at the "[python-dev] >" prompt. 

We can check the status of any MCP Servers that we have configured using the **"/mcp"** command. From the ">" prompt, type the following

```
> /mcp
```

You should see something like this.

```
[python-dev] 2% > /mcp

awslabs.aws-documentation-mcp-server
▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔
[2025:15:47]: ✓ awslabs.aws-documentation-mcp-server loaded in 9.71 s

```

So we have verified it has loaded ok. Now lets explore how we can use the tools that MCP Servers provide.

---

#### Configuring MCP Server Tools

We mentioned previously that MCP Servers will document what tools they make available. Once we have configured the MCP Server, we can see the available tools using the **"/tools"** command.

Tools added when you integrate MCP Servers are treated the same as other tools in your Kiro CLI session. They can be trusted or untrusted, which defines how these will operate when they are triggered. Like the built in tools within Kiro CLI, we can configure these by changing the properties of the custom agent by making updates to the JSON configuration file.  In addition to being able to change the default trust level of a tool, we can also configure which tools are available for any given custom agent. When you add these two things together, this allow you to:

* include or exclude specific tools for this custom agent
* change the default trust level of specific tools

We will look at this in the next lab.

---

**Task-21**

Run the following command to view the available tools in your Kiro CLI session:

```
> /tools
```

You should see something similar to the following:

```
[python-dev] > /tools

Tool                      Permission
▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔
Built-in
- shell                   not trusted
- read                    trust working directory
- write                   not trusted
- glob                    trust working directory
- grep                    trust working directory
- introspect              trusted
- report                  not trusted
- aws                     trust read-only commands
- web_fetch               not trusted
- web_search              not trusted

awslabs.aws-documentation-mcp-server (MCP)
- read_documentation      not trusted
- recommend               not trusted
- search_documentation    not trusted

```

As you can see, we have three new tools available that have been provided by the AWS Documentation MCP Server.

Lets change this and make some changes to our custom agent and change what tools we want available as well as the default permissions. From your Kiro CLI session, edit the python-dev custom agent we have been working on.

```
/agent edit --name python-dev
```

We configure both which tools are available, and the default trust in this section of the configuration file:

```
  "tools": ["*"],
  "allowedTools": [],
```

The tools field lists all tools that the agent can potentially use. Tools include built-in tools and tools from MCP servers, with the following conventions:

* Built in tools are specified by their name (e.g., read, shell)
* MCP server tools are prefixed with @ followed by the server name (e.g., @awslabs.aws-documentation-mcp-server)
* To specify a specific tool from an MCP server, use @awslabs.aws-documentation-mcp-server/search_documentation
* Use * as a special wildcard to include all available tools (both built-in and from MCP servers)
* Use @builtin to include all built in tools
* Use @server_name to include all tools from a specific MCP server

The allowedTools field specifies which tools can be used without prompting the user for permission. This is a security feature that helps prevent unauthorized tool usage. When you configure tools you can allow tools using several patterns:

* Specific built in tools - for example, "read", "shell", "aws"
* Specific MCP tools - for example "@awslabs.aws-documentation-mcp-server/read_documentation"
* Wildcard patterns - using * and ? for either built in or MCP tools, for example "@awslabs*"


We are going to update our python-dev custom agent so that it only has access to the read, write, and aws (no shell) built in tools, and read and search AWS documentation tools from the MCP Server. We edit our custom agent configuration so that it looks like the following:


```
{
  "name": "python-dev",
  "description": "",
  "prompt": "You are a Python developer. You have a preference for Flask web framework. You comply with PEP-8 when writing code.",
  "mcpServers": {
        "awslabs.aws-documentation-mcp-server": {
        "command": "uvx",
        "args": ["awslabs.aws-documentation-mcp-server@latest"],
        "disabled": false
        }
  },
   "tools": [
    "read",
    "write",
    "aws",
    "@awslabs.aws-documentation-mcp-server/read_documentation",
    "@awslabs.aws-documentation-mcp-server/search_documentation"
  ],
  "toolAliases": {},
  "allowedTools": [],
  "resources": [
    "file://AGENTS.md",
    "file://README.md",
    "file://.kiro/steering/*.md"
  ],
  "hooks": {},
  "toolsSettings": {},
  "useLegacyMcpJson": true,
  "model": null
}
```

Save and exit and then exit your Kiro CLI session, and restart with the python-dev custom agent:

```
kiro-cli --agent python-dev
```

From the "[python-dev] > " prompt, check to see which tools you have available. What do you notice? This is what you should see:

```
Tool                      Permission
▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔
Built-in
- read                    trust working directory
- write                   not trusted
- aws                     trust read-only commands

awslabs.aws-documentation-mcp-server (MCP)
- read_documentation      not trusted
- search_documentation    not trusted
```

You will see we have changed the available tools that our custom agent has access to. You should also notice that the default permissions have not changed. Lets change this so that we automatically trust the read and search documentation tools.

Edit the python-dev custom agent using the following command:

```
/agent edit --name python-dev
```

And modify the "allowedTools" section so that it looks like the following:

```
"allowedTools": ["@awslabs.aws-documentation-mcp-server/read_*","@awslabs.aws-documentation-mcp-server/search_*"],
```

this is the complete custom agent configuration file:

```
{
  "name": "python-dev",
  "description": "",
  "prompt": "You are a Python developer. You have a preference for Flask web framework. You comply with PEP-8 when writing code.",
  "mcpServers": {
        "awslabs.aws-documentation-mcp-server": {
        "command": "uvx",
        "args": ["awslabs.aws-documentation-mcp-server@latest"],
        "disabled": false
        }
  },
  "tools": [
    "fs_read",
    "fs_write",
    "use_aws",
    "@awslabs.aws-documentation-mcp-server/read_documentation",
    "@awslabs.aws-documentation-mcp-server/search_documentation"
  ],
  "toolAliases": {},
  "allowedTools": [
        "@awslabs.aws-documentation-mcp-server/read_*",
        "@awslabs.aws-documentation-mcp-server/search_*"
  ],
  "resources": [
    "file://AGENTS.md",
    "file://README.md",
    "file://.kiro/steering/*.md"
  ],
  "hooks": {},
  "toolsSettings": {},
  "useLegacyMcpJson": true,
  "model": null
}
```

After saving, exit your Kiro CLI session and again restart with the python-dev custom agent.

```
kiro-cli --agent python-dev
```

From the "[python-dev] > " prompt, now see the status of your tools using

```
/tools
```

What has changed? You should see something like this, with the read and search tools from the MCP server now automatically trusted.

```
Tool                      Permission
▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔
Built-in
- read                    trust working directory
- write                   not trusted
- aws                     trust read-only commands

awslabs.aws-documentation-mcp-server (MCP)
- read_documentation      trusted
- search_documentation    trusted

```

Let's test this out. 

```
> Check the AWS documentation for MWAA worker sizes
```

Follow the output. You should not be prompted for permission, and you should see something like this in the output

```
> I'll search the AWS documentation for information about MWAA worker sizes.
Running tool search_documentation with the param (from mcp server: awslabs.aws-documentation-mcp-server)
 ⋮  {
 ⋮    "search_phrase": "MWAA worker sizes Amazon Managed Workflows Apache Airflow"
 ⋮  }
 - Completed in 2.386s
```

That shows you its using the tools from the MCP Server.
---

#### Disabling MCP Servers

MCP Servers are very useful and can help bridge the context gap between what the underlying model needs and the task you are trying to complete. That said, when you configure and use MCP tools, they can quickly consume a lot of your context window. As such, when you do not need them, you should disable them.

We are able to disable MCP Servers from custom agents by editing the custom agent JSON file and adding new configuration item called "disabled" and setting this to true (the default is false).

**Task-22**

Edit the python-dev custom agent using the following command:

```
/agent edit --name python-dev
```


Edit the custom agent configuration as follows, locating the MCP Server details we added and changing the value of disabled:

```
{
  "name": "python-dev",
  "description": "",
  "prompt": "You are a Python developer. You have a preference for Flask web framework. You comply with PEP-8 when writing code.",
  "mcpServers": {
        "awslabs.aws-documentation-mcp-server": {
        "command": "uvx",
        "args": ["awslabs.aws-documentation-mcp-server@latest"],
        "disabled": true
        }
  },
  "tools": [
    "fs_read",
    "fs_write",
    "use_aws",
    "@awslabs.aws-documentation-mcp-server/read_documentation",
    "@awslabs.aws-documentation-mcp-server/search_documentation"
  ],
  "toolAliases": {},
  "allowedTools": [
        "@awslabs.aws-documentation-mcp-server/read_*",
        "@awslabs.aws-documentation-mcp-server/search_*"
  ],
  "resources": [
    "file://AGENTS.md",
    "file://README.md",
    "file://.kiro/steering/*.md"
  ],
  "hooks": {},
  "toolsSettings": {},
  "useLegacyMcpJson": true,
  "model": null
}
```

Save the configuration file and then restart your Kiro CLI session. You should see output at the top of the screen that displays:

```
○ awslabs.aws-documentation-mcp-server is disabled
```

If you run **"/mcp"** what do you get?

---

#### Disabling MCP for your organization

MCP Servers provide powerful capabilities to AI Coding Agents like Kiro CLI, but as mentioned above, they come with some risks. For some those risks might be too high and you might want to disable the configuration of MCP Servers at an organizational level. This capability is available but only for those that login using Enterprise accounts.

When you are using Kiro CLI, if you now start an instance of Kiro CLI with a custom agent that has MCP Servers configured, and you see the following:

```
⚠️  WARNING: MCP functionality has been disabled by your administrator.
```

You will know that MCP has been disabled at the org level.

### Supporting Resources

Some additional reading material that dives deeper into this topic if you want to explore:

* [Kiro.dev](https://kiro.dev/docs/cli/?trk=fd6bb27a-13b0-4286-8269-c7b1cfaa29f0&sc_channel=el)

* [Kiro Steering documents](https://kiro.dev/docs/cli/steering/?trk=fd6bb27a-13b0-4286-8269-c7b1cfaa29f0&sc_channel=el)

* [Kiro Context Management](https://kiro.dev/docs/cli/chat/context/?trk=fd6bb27a-13b0-4286-8269-c7b1cfaa29f0&sc_channel=el)

* [Kiro custom agent examples](https://kiro.dev/docs/cli/custom-agents/examples/?trk=fd6bb27a-13b0-4286-8269-c7b1cfaa29f0&sc_channel=el)

### Completed!

Now that you have got started with Kiro CLI, you can explore [Kiro CLI Advanced Topics](/workshop/04-advanced-topics.md) to dive deeper and get the most out of this tool.