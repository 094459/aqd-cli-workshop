![Amazon Q Developer header](/images/kiro-workshop-header.png)

# Diving deeper into Kiro CLI

In this lab we are going to build on what you have already learned about Kiro CLI's features, looking at some of the more advanced features and capabilities.

## Custom Agents configuration file

In the previous labs we looked at what custom agents were, why they were useful, and used Kiro CLI to create some. We took a look at some of the basic configuration options around configuring MCP Servers and adding context. There is a lot more we can do, and so in this section we are going to dive deep into the configuration file and show you how you can create detailed and powerful custom agents that will help you get the best output form Kiro CLI.

### Configuration reference

In previous labs we have created custom agents and looked at some of the configuration options available. We have so far covered the following 

* name — The name of the agent (in previous labs we have created "python-dev")
* description — A description of the agent (optional)
* prompt — High-level context for the agent
* mcpServers — The MCP servers the agent has access to
* tools — The tools available to the agent
* allowedTools — Tools that can be used without prompting
* resources — Resources available to the agent

In the labs in the part of the workshop, we will be exploring the remaining configuration options.

* toolAliases — Tool name remapping for handling naming collisions
* toolsSettings — Configuration for specific tools
* hooks — Commands run at specific trigger points
* model — The model ID to use for this agent

----

### Associating a specific model with your custom agent

We saw that you can choose the model you want to use when working in the Kiro CLI chat interface using the "/model" command. We also saw how you can set a specific model as the default. You might want to use a specific model within your custom agent, and the good news is that you can define a specific model by setting the **"model"** configuration setting within your custom agent configuration. 

Using the previous custom agent configuration from previous labs, we could set our custom agent to use claude-4.5-sonnet by editing and saving the configuration as follows.

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
  "model": "claude-sonnet-4.5"
}
```

When we save and exit, you will need to restart Kiro CLI. The next time you switch to this custom agent it will use this specific model. If you have no preference, or just want to use the default Kiro CLI model (Auto), then you can leave this configuration setting set to null.

---

## Using the Generate Wizard

In the previous labs we showed how to create custom agents via the "/agent create" command which allowed you to use the editor to then add your configuration details. There is also a wizard that will guide you through the creation process. The "/agent generate" will walk you through the steps for creating a custom agent, and this can be a super handy way to get started.

**Task-01**

We have already created a Python custom agent, now lets create a Java one using the generate option. From your Kiro CLI ">" prompt, run the following command:

```
/agent generate
```

You will now be prompted via a text UI to enter:

* agent name - enter "java-dev"
* agent description - enter "A custom agent that provides Java Spring Boot developer guidance for Spring Boot v3.x"
* agent scope - enter "Local (current workspace)"

It should respond with output similar to the following:

```
⠴ Generating agent config for 'java-dev'...

✓ Agent 'java-dev' has been created and saved successfully!
```

Review the custom agent configuration by using the following command:

```
/agent edit --name java-dev
```

Review the output. Check the value of the prompt configuration. One of the advantages of using the "/agent generate" when initially creating custom agents is that it will try and craft an initial prompt based on the description you provide. You can of course edit this as needed.

You can use and configure this custom agent in exactly the same way that we have seen in previous labs.

---

### Creating prompts for your custom agent

In the previous lab we looked at how you could generate a custom agent, which would also provide a starting prompt for that custom agent. You can also link to existing markdown documents that you might have or prefer to use. This might be handy if you want to create standardized custom agent configurations that might share common prompts.

**Task-02**

Exit Kiro CLI, and create a file a new directory in your ".kiro" directory called "shared", and add [this file from the resources](/resources/springboot.md) into this directory. This should be what your directory will look like

```
├── .kiro
   ├── agents
   │    ├── java-dev.json
   │    └── python-dev.json
   ├── prompts
   │   └── proj-setup.md
   ├── shared
   │   └── springboot.md
   └── steering
       └── python-standards.md
```

Start Kiro CLI with no custom agent, and from the ">" prompt, lets edit the newly created "java-dev" custom agent to grab the prompt from this resource file.

```
/agent edit --name java-dev
```

and we can replace the "prompt" with a link to the resource file.

```
{
  "name": "java-dev",
  "description": "A custom agent that provides Java Spring Boot developer guidance for using Spring Boot v3.x",
  "prompt": "file://../shared/springboot.md",
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
  "useLegacyMcpJson": false,
  "model": null
}
```

After saving and exiting, you can now try and switch to this agent.

```
/agent swap java-dev
```

Lets try and see whether this new prompt will give us some Java Spring Boot goodness. From the "[java-dev] > " prompt,

```
show me some code that will display hello world
```

Review the output - it should be beautiful Spring Boot loveliness.

---

## Context Hooks

Context hooks are a feature in Kiro CLI that you can use to automatically inject context into your conversations at certain events. Context hooks run commands and include the output of those commands as context. We configure context hooks within custom agents.

Kiro CLI supports four types of context hooks:

* **Conversation start hooks ("AgentSpawn")** - Run once at the beginning of a conversation. Their output is added to the conversation context and persists throughout the session.
* **Per-prompt hooks ("UserPromptSubmit")** - Run with each user message. Their output is added only to the current prompt.
* **Hooks that run before tools execution ("PreToolUse")** - This hook will runs before tool execution, which might be useful to validate and block tool usage.
* **Hooks that run post tool execution ("PostToolUse")** - Runs after tool use, and will be provided the output of the tool.

You can view any available context hooks by using the command **"/hooks"** within your Kiro CLI session. 

```
[java-dev] > /hooks

No hooks are configured.
```

As you can see, we do not currently have any setup so lets change that.

---

**Task-03**

Lets add a new context hook that is invoked every time we prompt (Per-prompt hook "UserPromptSubmit"). We are going to have some fun and create a per prompt hook that will make Kiro CLI talk like a pirate.

Exit Kiro CLI and create a new file called **"pirate.md"** in the .kiro/shared directory, and add this to the file:

```
Talk like a pirate and make bad jokes. Always ask for grog as you are thirsty.
```

Edit the custom agent JSON configuration file we worked on as part of the previous task. Edit the file so that it looks like the following:

```
{
  "name": "java-dev",
  "description": "A custom agent that provides Java Spring Boot developer guidance for using Spring Boot v3.x",
  "prompt": "file://../shared/springboot.md",
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
  "hooks": {"userPromptSubmit": [
      {
        "command": "cat .kiro/shared/pirate.md"
      }
    ]
  },
  "toolsSettings": {},
  "useLegacyMcpJson": false,
  "model": null
}
```

After saving the file, restart your Kiro CLI session with the java-dev custom agent

```
kiro-cli --agent java-dev
```

and then run the following command:

```
> /hooks
```

You should get output like the following:

```
[java-dev] > /hooks

userPromptSubmit:
  - cat .kiro/shared/pirate.md

```

Lets test this out now. Enter the following prompt:

```
> show me some code that will display hello world
```

Review the output. What happens? Try running the same prompt again.

You can edit and remove the context hook after this task if you want, or enjoy that Kiro CLI is talking to you in pirate.

---

### Running hooks when you start Kiro CLI session

The previous example showed you how to configure hooks that operated when you a submit prompts. You can also configure hooks that operate when you first start your Kiro CLI session (again when starting with a custom agent). This is done using the **"agentSpawn"** configuration argument, and can be very useful if you want to prepare your session before you start - for example you might want to install some binaries, or perhaps copy down some context files from a central repository.

This is how I would configure this in the custom agent configuration file, in this example I am running a command to copy some context from a central GitHub repo, which I might want to use as context. Every time the custom agent starts, it will download a fresh copy of this file, ensuring I always have the latest version.

```
{
  "name": "java-dev",
  "description": "A custom agent that provides Java Spring Boot developer guidance for using Spring Boot v3.x",
  "prompt": "file://../shared/springboot.md",
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
  "hooks": {"agentSpawn": [
      {
        "command": "cd .kiro/shared && wget -N https://raw.githubusercontent.com/094459/aqd-cli-workshop/refs/heads/main/resources/python-dev.md"
      }
    ]
  },
  "toolsSettings": {},
  "useLegacyMcpJson": false,
  "model": null
}
```
---

### Tool-related hooks

Whilst the previous two agent hooks have focused on starting Kiro CLI or entering prompts, you can also configure hooks to work with tools. When you want to use the tool related hooks, there are additional configuration parameters you need to define.

**"PreToolUse"** and "PostToolUse"** are used to configure hooks that run either before or after tools are run. You define these slightly different to the previous hooks. You will need to use **"matcher"** to tag the tool you want to assign the custom hook to. 

In the following examples you can see that we are using this to tag on when the **execute_bash** and **fs_Write** tools are used. One audits what is being run, and the other runs the rustftm to ensure all files are formatted consistently.

```
{
  "$schema": "https://raw.githubusercontent.com/aws/amazon-q-developer-cli/refs/heads/main/schemas/agent-v1.json",
  "name": "python-developer",
  "description": "",
  "prompt": null,
  "mcpServers": {},
  "tools": ["*"],
  "toolAliases": {},
  "allowedTools": ["fs_read","fs_write","use_aws"],
  "resources": [
    "file://.amazonq/rules/**/*.md"
  ],
  "hooks": {
    "preToolUse": [
      {
        "matcher": "execute_bash",
        "command": "{ echo \"$(date) - Bash command:\"; cat; echo; } >> /tmp/bash_audit_log"
      }
    ],
    "postToolUse": [
      {
        "matcher": "fs_write",
        "command": "cargo fmt --all"
      }
    ] 
  },
  "toolsSettings": {}
}
```

---


## Configuring fine grained permissions for your Tools

We saw earlier in this lab how we can configure tools within custom agents, controlling which tools a specific custom agent had access to as well as the permissions granted. This is great, but what if you wanted to have more fine grain control over access. For example, we saw that we have a tool called **"execute_bash"** that allows you to run bash commands. What if you wanted to only allow certain commands to run?

This is possible using the **"toolsSettings"** configuration within custom agents. It allows you to configure specific controls that a given tool provides. For example, from the [Github pages](https://github.com/aws/amazon-q-developer-cli/blob/main/docs/built-in-tools.md) we can see that:

* **fs_write** and **fs_read** provide **"allowedPaths"** and **"deniedPaths"** configuration options
* **use_aws** provide **"allowedServices"** and **"deniedServices"** configuration options
* **execute_bash** provide **"allowedCommands"**, **"deniedCommands""**, and **"allowReadOnly"** options

You can provide explicity configuration references or use regex. Here is an example of how we would provide more fine grain control of what the **"execute_bash"** tool could do.

```
"toolsSettings": {
    "execute_bash": {
      "allowedCommands": ["git status", "git fetch"],
      "deniedCommands": ["git commit .*", "git push .*"],
      "allowReadOnly": true
    }
  }

```


If you are using tools that have been surfaced up via an MCP Server, you will need to review the documentation for that MCP Server and then use this configuration option in the same was as the built in ones. For example:

```
  "toolsSettings": {
    "@git/git_status": {
      "git_user": "$GIT_USER"
    }
  }
```

Not every tool has configuration options, so review documentation to find out if they do.

**Task-10**

Lets configure our custom agent so that we limit what files Kiro CLI has access to. We want it to be able to read files in the current project directory, but exclude more sensitive files.

Following on from previous examples, we will modify the custom agent as follows:

```
{
  "$schema": "https://raw.githubusercontent.com/aws/amazon-q-developer-cli/refs/heads/main/schemas/agent-v1.json",
  "name": "python-developer",
  "description": "",
  "prompt": null,
  "mcpServers": {},
  "tools": [
    "fs_read",
    "fs_write",
    "use_aws"
  ],
  "toolAliases": {},
  "allowedTools": ["fs_read", "fs_write"],
  "resources": [
    "file://.amazonq/rules/**/*.md"
  ],
  "hooks": {},
  "toolsSettings": {
    "fs_read": {
      "allowedPaths": ["~/projects", "./src/**"],
      "deniedPaths": ["/tmp/*"]
    }
  }
}
```

As you can see we have added the following:

```
"toolsSettings": {
    "fs_read": {
      "allowedPaths": ["./**"],
      "deniedPaths": ["/tmp/*"]
    }
  }
```

Which will block Kiro CLI's ability to read files from a specific directory (in this case "/tmp",) but allow it to read files from the current directory and all subdirectories.

Lets create some files to test this out.

```
echo "import os" > /tmp/q-cli-test.py
echo "import os" > q-cli-test.py
echo "import os" > ~/q-cli-test.py
```

Restart your Kiro CLI session, and try the following prompts:

```
> Can you review the "q-cli-test.py" file and tell me what programming language its wriiten in
```

Does it run ok?

Now try the following:

```
> Can you review the "/tmp/q-cli-test.py" file and tell me what programming language its wriiten in
> Can you review the "~/q-cli-test.py" file and tell me what programming language its wriiten in
```

Now what happens? You should see something like the following:

```
> I'll read the "/tmp/q-cli-test.py" file to review it and identify the
programming language.

> I understand that the file read was rejected due to forbidden arguments. This is
likely because the path "/tmp/q-cli-test.py" is outside the allowed directory
scope or contains restricted content.

```

The ability to control at a granular level what your tools have access to is a very powerful capability that you can use to ensure that Kiro CLI is only operating on files that you want it to, or run commands that you want it to execute.

---

---

## Clashing tool names - using toolAliases

One thing to bear in mind as you add MCP Servers to your Kiro CLI is the potential for tools to have a name clash. For example, imagine if you have a tool called "get_issues" and had integrated the GitHub and GitLab MCP Servers. These both expose tools called "get_issues", so how do you manage this?

Within the custom agent configuration, you can create alias for tools and map these to the explicit tool you want to map it to. In the above example, we would add the following to the custom agent JSON configuration file:

```
  "toolAliases": {
    "@github-mcp/get_issues": "github_issues",
    "@gitlab-mcp/get_issues": "gitlab_issues"
  }
```

This is something to bear in mind as you start adding MCP Server tools and want to manage any conflicts that might arise.

---

### Changing how you refer to tools using toolsAliases

In the previous section we shared how you can use **"toolsAliases"** to address namespace clashes.

You can also use aliases to create shorter or more intuitive names for frequently used tools. For example, if you have some long tools you can shorten them with something like:

```
{
  "toolAliases": {
    "@aws-cloud-formation/deploy_stack_with_parameters": "deploy_cf",
    "@kubernetes-tools/get_pod_logs_with_namespace": "pod_logs"
  }
}
```

This is a nice developer experience feature that I am using with some of my custom agent tools.













***











---

## MCP Prompts

We have already looked at how Kiro CLI supports MCP Tools, but it also supports MCP Prompts. In earlier labs we looked at how you can use the **"/prompts"** command to list and use prompts that are stored on your machine, but you can also extend this to use prompts that are made available via MCP Servers.

To demonstrate this we are going to implement a simple MCP Server that provides Prompt resources, and then configure Kiro CLI to use this and show you how it works.

**Task-08**

Open a **new terminal window** and create a new directory (for example, "mcp-prompts").

From this window, create a file called **"mcp-server.py"** and add the following code. This will generate a simple MCP Server that provides a Prompt:

```
import asyncio
import sys

from mcp.server.fastmcp import Context, FastMCP

# Create an MCP server
mcp = FastMCP("QCLIPromptDemo")

@mcp.prompt()
def create_new_project() -> str:
    """A prompt that bootstraps a new project repository as per our requirements"""
    return f'Create a new project layout. Create a top level src directory, and in the src directory create subdirectories for templates, models, routes, and static. Add a README.md to the src directory.'

if __name__ == "__main__":
    mcp.run()
```

So what have we created here?

We have created an MCP Server which provides a sample prompt to bootstrap a project using a specific layout (this is a very simplified example, you could build something much more detailed and complex).

After saving this file, create a virtual python environment and install a couple of dependencies:

```
cd mcp-prompts
uv init
uv add mcp "mcp[cli]"
```

That's it we now have a simple custom MCP Server that provides Prompts. The configuration to use this is as follows, replacing {path to directory} with the directory you created in the previous steps, in our example this was 'mcp-server' :

```
{
    "mcpServers": {
        "QCLIPromptDemo": {
            "command": "uv",
            "args": ["--directory", "{path to directory}", "run", "--with", "mcp", "mcp", "run", "mcp-server.py"]
        }
    }
}
```

We now need to add this MCP Server to our custom agent JSON configuration. We will use the one we have been using throughout this lab (but feel to create a new one if you want). When I do a "pwd" I get the following working directory:

```
/Users/ricsue/amazon-q-developer-cli/mcp-server
```

Which gives me the path I need for the MCP Server. I then edit this JSON configuration file so it looks like this:

```
{
  "$schema": "https://raw.githubusercontent.com/aws/amazon-q-developer-cli/refs/heads/main/schemas/agent-v1.json",
  "name": "python-developer",
  "description": "",
  "prompt": null,
  "mcpServers": {
	"awslabs.aws-documentation-mcp-server": {
        	"command": "uvx",
        	"args": ["awslabs.aws-documentation-mcp-server@latest"],
        	"env": {
          		"FASTMCP_LOG_LEVEL": "ERROR",
          		"AWS_DOCUMENTATION_PARTITION": "aws"
        	  },
		      "disabled": false
		    },
	"QCLIPromptDemo": {
          "command": "uv",
          "args": ["--directory", "/Users/ricsue/amazon-q-developer-cli/mcp-server", "run", "--with", "mcp", "mcp", "run", "mcp-server.py"]
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
  "allowedTools": ["@awslabs.aws-documentation-mcp-server/read_documentation"],
  "resources": [
    "file://.amazonq/rules/**/*.md"
  ],
  "hooks": {},
  "toolsSettings": {}
}
```



You can see we have added the new MCP Server we created by adding the following to the previous custom agent configuration file. Again, yours will be slightly different as the directory you have created for mcp-server will be different.:

```
	"QCLIPromptDemo": {
          "command": "uv",
          "args": ["--directory", "/Users/ricsue/amazon-q-developer-cli/mcp-server", "run", "--with", "mcp", "mcp", "run", "mcp-server.py"]
        }
```

After saving this file, in the same directory, start an Kiro CLI session and confirm that the MCP Server has started ok:

```
[python-developer] > /mcp

QCLIPromptDemo
▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔
✓ QCLIPromptDemo loaded in 9.79 s

awslabs.aws-documentation-mcp-server
▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔
✓ awslabs.aws-documentation-mcp-server loaded in 7.91 s
```

If you run **"/tools"**, what do you see? It should look exactly as previous. We have not added any new tools with this MCP Server, we have added prompts. So how do we see these? Simple, we use the **"/prompts"** command within our Kiro CLI Session.

From the **">"** prompt, type **"/prompts"** and hit return. You should see something like the following:

```
> /prompts list

Prompt                    Arguments (* = required)
▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔
QCLIPromptDemo (MCP):
- create_new_project
```

We can try this out now, from a new **">"** enter the following

```
@create_new_project
```

Which should generate output like the following:

```
> @create_new_project

I'll help you create this project layout with the requested directory structure and README.md file. Let
me do that for you.

```

Follow the output - you may need to provide permission as it will write files and we have not given this custom agent fs_write permission. Once it has finished, what did you get? Compare it to the prompt defined in the function - does it look like its created what was asked?





---

## Experimental Commands

During the advanced setup section we looked at how you can enable "Experimental" features within Kiro CLI. This section explores some of these experimental features so that you can understand them. The caveat here is that you should treat these tools accordingly - do not use them for production or real activities, and they may disappear in future updates.


### Managing Tasks with "todos"

As you start to use Kiro CLI on a daily basis, you will start to pick up habits and learn new ways of working. One of the emerging patterns that we have seen provide developers with a lot of success in using AI coding assistants, is to use them to create task lists and the iterate on those tasks sequentially. This helps both the developer know where they are, but also provides the AI coding assistant a useful checkpoint too.

A new experimental feature simplifies how you can do this in Kiro CLI. There are two parts to this: the new **"/todos"** command available from within your Kiro CLI session, and the **"todo_list"** tool that is made available for the underlying model that Kiro CLI is using to interact with those lists. The workflow is as follows:

1. Use Kiro CLI to generate a Todo list
2. Use the new **"/todos"** command to view, list, and select a Todo list you want to work on
3. Use the new **"/todos resume"** to start working on the next task in that list

Todo's are files that get created in your local filesystem, within your current project workspace. There is a new directory called **".amazonq/cli-todo-lists"** that is created, and when you define your lists, they will appear here as json files.

Lets explore this new feature in the next lab.

**Task-11**

The first thing we need to do is enable this feature by using the /experiment mode ( [see the advanced section for more details if you missed that section out](/workshop/01b-advanced-setup-topics.md) ) - move down to "Todo Lists" and then press space

```
? Select an experiment to toggle ›
  Knowledge          [ON]  - Enables persistent context storage and retrieval across chat sessions (/knowledge)
  Thinking           [OFF] - Enables complex reasoning with step-by-step thought processes
❯  Tangent Mode      [OFF]  - Enables entering into a temporary mode for sending isolated conversations (/tangent)
  Todo Lists         [OFF] - Enables Q to create todo lists that can be viewed and managed using /todos
```

After pressing space, you should see the followiing.

```
 Todo Lists experiment enabled
```

We should now have access to a new command **"/todos"** which we can see by running the following command:

```
> /todos --help

View, manage, and resume to-do lists

Usage: /todos <COMMAND>

Commands:
  clear-finished  Delete all completed to-do lists
  resume          Resume a selected to-do list
  view            View a to-do list
  delete          Delete a to-do list
  help            Print this message or the help of the given subcommand(s)
```

If we run the **"/tools"** command, we can see we now have a new tool available - todo_list, and that it is automatically trusted.

```
> /tools

Tool              Permission
▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔
Built-in:
- execute_bash    * trust read-only commands
- fs_read         * trusted
- fs_write        * not trusted
- introspect      * trusted
- report_issue    * trusted
- todo_list       * trusted
- use_aws         * trust read-only commands

```

Now that we have this enabled, lets try it out by creating our first Todo list.

From your Kiro CLI session, at the **">"** prompt, write the following:

```
Create a todo list to generate a simple python flask app that returns a Yoda quote of the day
```

You should see something similar to the following - note that this has invoked the **todo_list** tool that is now enabled when you configured this experimental feature.

```
> I'll create a todo list for building a simple Python Flask app that returns a Yoda quote of the day.


🛠️  Using tool: todo_list (trusted)
 ⋮
 ● TODO:
[ ] Create requirements.txt with Flask dependency
[ ] Create app.py with Flask app and Yoda quotes list
[ ] Add route that returns random Yoda quote as JSON
[ ] Test the Flask app locally
 ⋮
 ● Completed in 0.5s


> Perfect! I've created a todo list with 4 essential steps to build your Yoda quote Flask app. The plan focuses on the
minimal requirements: setting up dependencies, creating the Flask app with quotes, adding the API endpoint, and testing it
works.

Ready to start implementing? I can work through these steps with you one by one.
```

We are not going to do that straight away. Exit your Kiro CLI session, and take a look at the new directory and file that has been created.

```
└── .amazonq
    ├── cli-agents
    └── cli-todo-lists
        └── 1758028067437.json
```

If you view that file, you see that it has been created in a specific format. This was the file created in the above example:

```
{"tasks":[{"task_description":"Create requirements.txt with Flask dependency","completed":false},{"task_description":"Create app.py with Flask app and Yoda quotes list","completed":false},{"task_description":"Add route that returns random Yoda quote as JSON","completed":false},{"task_description":"Test the Flask app locally","completed":false}],"description":"Create a minimal Python Flask app that returns a Yoda quote of the day","context":[],"modified_files":[],"id":"1758028067437"}%
```

Start your Kiro CLI session, and we are going to create another list. This time use the following prompt:

```
Create a todo list to generate a simple python flask app that will generate a simple fact checking application
```

Once this has completed, lets see how we can view the available lists by using the **"/todos view"** command:

```
> /todos view

? Select a to-do list to view: ›
  ✗ Create a minimal Python Flask app that returns a Yoda quote of the day (0/4)
  ✗ Build a simple Python Flask fact-checking application with basic UI and fact verification functionality (0/6)
```

You should see the two lists that we have created. The number at the end identifies the number of tasks, as well as the number of tasks that have been completed.

We use the **UP** and **DOWN** arrow keys to select a list, and then press ENTER when we want to "activate" a todo list. Select the first (Yoda quote of the day) and hit enter. It should now display that todo list.

```
> /todos view

Viewing: Create a minimal Python Flask app that returns a Yoda quote of the day

TODO:
[ ] Create requirements.txt with Flask dependency
[ ] Create app.py with Flask app and Yoda quotes list
[ ] Add route that returns random Yoda quote as JSON
[ ] Test the Flask app locally
```

To get Kiro CLI to start working iteratively through the tasks, we use the **"/todos resume"** command. From your **">"** prompt, enter the following:

```
/todos resume
```

You will be asked to specify which todo list - select the Yoda quote of the day and hit enter. You should see something like this to start with:

```
⟳ Resuming: Create a minimal Python Flask app that returns a Yoda quote of the day
⠸ Thinking...
```

Kiro CLI will then begin its usual cycle of building what these tasks have defined. As it completes one task, you should see the todo list get updated.

```
🛠️  Using tool: todo_list (trusted)
 ⋮
 ● TODO:
[x] Create requirements.txt with Flask dependency
[ ] Create app.py with Flask app and Yoda quotes list
```

After a few minutes, all the tasks should be completed and the application finished. We can check the status by using the **"/todos view"** command:

```
> /todos view

Viewing: Create a minimal Python Flask app that returns a Yoda quote of the day

TODO:
[x] Create requirements.txt with Flask dependency
[x] Create app.py with Flask app and Yoda quotes list
[x] Add route that returns random Yoda quote as JSON
[x] Test the Flask app locally
```

As we can see, all the tasks have been completed. We can remove completed todo lists by using the **"/todos clear-finished"** command.

```
> /todos clear-finished

✔ Cleared finished to-do lists!
```

If you now try and list views, you will notice that the Yoda todo list is no longer there.


*Deleting todo lists*

As well as creating todo lists, you can delete them using the **"/todos delete"** command. Lets delete the second todo list we created (the fact checking application). From the **">"** prompt, type:

```
> /todos delete
```

and from the list that appears, again use the UP and DOWN arrow. Select the fact checking application to do list and then press ENTER. It should display something similar to the following:

```
✔ Deleted to-do list: Build a simple Python Flask fact-checking application with basic UI and fact verification functionality
```

> You can also delete these by deleting the corresponding todo json file in the ".amazonq/cli-todo-lists" directory.

---

### Tangent Mode

In the previous labs, we have explored context engineering and looked at some of the techniques you have available to you to effectively manage your context. In this lab we are going to introduce a new experimental feature that provides some useful additional capabilities to help you minimise polluting your context. What does this mean? As your Kiro CLI sessions progress, you might need to use Kiro CLI to do adjacent tasks (for example, maybe troubleshoot an issue, or perhaps dive into a rabbit hole) but those tasks might not necessarily contribute to your objective and so you might not want to include the output of those interactions in your context. 

**Tangent mode** creates conversation checkpoints, allowing you to explore side topics without disrupting your main conversation flow. Enter tangent mode, ask questions or explore ideas, then return to your original conversation exactly where you left off.


**Task-14**

The first thing we need to do is enable this by using the /experiment mode ( [see the advanced section for more details if you missed that section out](/workshop/01b-advanced-setup-topics.md) ) - move down to "Tangent Mode" and then press space

```
? Select an experiment to toggle ›
  Knowledge          [ON]  - Enables persistent context storage and retrieval across chat sessions (/knowledge)
  Thinking           [OFF] - Enables complex reasoning with step-by-step thought processes
❯  Tangent Mode      [OFF]  - Enables entering into a temporary mode for sending isolated conversations (/tangent)
  Todo Lists         [OFF] - Enables Q to create todo lists that can be viewed and managed using /todos
```

After pressing space, you should see the followiing.

```
 Tangent Mode experiment enabled
```

From the Kiro CLI session, you can now run the command **"/tangent"**

```
> /tangent
```

You will notice that the **">"** prompt changes to **"↯ >"**, to let you know that you are now in tangent mode. What might you do when you are in tangent mode? Here are some example use cases:

* Explore alternatives approaches - you might have started along one path, but you can use tangents to explore alternatives so that they do not add to your existing context
* Getting help on Kiro CLI - if you need to ask Kiro CLI for help on some of its features, but dont want that to pollute your context
* Clarification - you might want to dive into some details to make sure that you have everything you need, but want to do this in a way that does not distract your existing context

You can check out some [additional examples use cases here](https://github.com/aws/amazon-q-developer-cli/blob/main/docs/tangent-mode.md#usage-examples)

> If you see the following when you run the command (**"/tangent"**) command, you have not enabled Tangent in the experimental model
> 
> ```
> > /tangent
> Tangent mode is disabled. Enable it with: q settings chat.enableTangentMode true
> ```

```
> /tangent

Created a conversation checkpoint (↯). Use ctrl + t or /tangent to restore the conversation later.
Note: this functionality is experimental and may change or be removed in the future.

↯ >
```

You can now use Kiro CLI in exactly the same way, knowing that your previous conversations have been check pointed and safe. Once you have finished, you can exit your session and return back to your previous conversation by running the **"/tangent"** command again.

```
↯ > /tangent

Restored conversation from checkpoint (↯). - Returned to main conversation.
```

You will notice that your prompt changes back, and you are now ready to carry on with your session.

You might be thinking that what if you wanted to include your tangent conversation back into the main chat conversation? A new feature added in v1.16 of Kiro CLI allows you to do this by using a new argument that will preserve the last conversation back in your main context. It works as follows - after initiating a new tangent session, after you have prompted and got a response, you can use the following command, **"/tangent tail"** which will then generate the following output.

```
↯ > /tangent tail

Restored conversation from checkpoint (↯) with last conversation entry preserved.
```

**Task-15**

Close and then start a new Kiro CLI session. From this new session, create a new tangent by running the following:

```
> /tangent
↯ >
```

Enter the following prompt:

```
Tell me about Python's Fast API
```

Watch the output. After it has finished, exit the tangent using **"/tangent"**

```
↯ > /tangent

Restored conversation from checkpoint (↯). - Returned to main conversation.
```

And now ask the following prompt:

```
What were we talking about?
```

It should return something similar to the following:

```
I don't have any record of previous conversations with you. Each chat session with Amazon Q starts fresh without memory of past interactions.
```

As we can see, tangent has done its job and has not brought back any of that back into your main chat session. Now repeat the above, but this time, after you have asked it about Python's FastAPI, instead of using **"/tangent"** use **"/tangent tail"** and then repeat the prompt. You should see something different this time.

```
↯ > /tangent tail

Restored conversation from checkpoint (↯) with last conversation entry preserved.

> what were we talking about

> We were discussing Python's FastAPI framework. I explained its key features like high performance, automatic documentation, type safety, and async support, along with a basic code example and installation instructions.
```


---





---

### Supporting Resources

Some additional reading material that dives deeper into this topic if you want to explore:

* [Kiro custom agent configuration file reference](https://kiro.dev/docs/cli/custom-agents/configuration-reference/)

### Completed!

Now that we have an idea of how Kiro CLI works and explored some of the features, we can take a look at some of the use cases in [Exploring Use Cases](03a-use-cases.md)


