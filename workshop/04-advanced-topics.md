![Kiro CLI header](/images/kiro-workshop-header.png)

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

**"PreToolUse"** and **"PostToolUse"** are used to configure hooks that run either before or after tools are run. You define these slightly different to the previous hooks. You use **"matcher"** to assign the hook to. When configuring matcher, you can configure it to select specific or a number of tools.

> **Note!** As Kiro CLI was previously Amazon Q CLI, some of the internals are still being updated. This means that whilst the tools we configured in earlier labs are correct, they map to other names:
> * write -> fs_write
> * read -> fs_read
> * shell -> execute_bash
> * aws -> use_aws
>

* "{tool}" - Exact match for a specific built in tools (for example, for "write" we would use "fs_write")
* "@{mcp}" - The "@" matches all tools from a given MCP server (for example, "@git" would be all tools from the Git MCP Server)
* "@{mcp}/{tool}" - Matches a specific tool from a MCP server (for example, "@git/status would match just the {status} tool from the @git MCP Server)
* "*" - All tools (built-in and MCP)
* "@builtin" - All built-in tools only
* No matcher - Applies to all tools

We will bring this to life by showing how we can we can extend our java-dev custom agent to illustrate this.

---

**Task-04**

We are going to add some pre and post hooks for running specific tools. In your Kiro CLI session, edit the java-dev custom agent configuration using the following command:

```
/agent edit --name java-dev
```

We are going to run a command, and log these out to a file so you can see that it actually does something. We need to add the following to our custom agent configuration file:

```
  "hooks": {
    "preToolUse": [
      {
        "matcher": "execute_bash",
        "command": "echo \"Running pre tool command, for all built in tools\" >> /tmp/pre-tool.log"
      }
    ],
    "postToolUse": [
      {
        "matcher": "fs_write",
        "command": "echo \"Running post tool command, only for write tool\" >> /tmp/post-tool.log"
      }
    ]
  },
```

When we run a shell command or write something to the file system, we will trigger a pre or post tool hook. We are directing the echo command to a file in /tmp/post-tool.log or /tmp/pre-tool.log which we will check to show that these have been triggered.

Your completed configuration file should look like this:

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
    "hooks": {
    "preToolUse": [
      {
        "matcher": "execute_bash",
        "command": "echo \"Running pre tool command, for all built in tools\" >> /tmp/pre-tool.log"
      }
    ],
    "postToolUse": [
      {
        "matcher": "fs_write",
        "command": "echo \"Running post tool command, only for write tool\" >> /tmp/post-tool.log"
      }
    ]
  },
  "toolsSettings": {},
  "useLegacyMcpJson": false,
  "model": null
}
```

After saving and exiting, you will need to restart your Kiro CLI session. Restart Kiro CLI with

```
kiro-cli --agent java-dev
```

Now from the "[java-dev] >" lets try and trigger this with the following prompt:

```
Write a simple html page that says hello world, and then use the shell ls command to list all files
```

You will be prompted to trust these commands, and you should see that a new file is written and then it displays the directory list. Once it has finished, review your file system to see if you have a /tmp/pre-tool.log and /tmp/post-tool.log.

Edit your java-dev custom agent configuration and change the matcher to use different tools and repeat the exercise.

---

## Fine grained permissions for your Tools

In previous labs we configured tools within custom agents, controlling which tools and the default permissions granted. Kiro CLI allows you to go a step further and configure tools permissions with more precision. For example, we can trust the **"shell"** tool that will allow you to run specific bash commands or perhaps you want to only trust specific directories for reading and writing. 

This is possible using the **"toolsSettings"** configuration within custom agents. It allows you to configure specific controls that a given tool provides. We do need to know for each tool though, what configuration options we can configure. For that we can refer to [this page on Github](https://github.com/aws/amazon-q-developer-cli/blob/main/docs/built-in-tools.md) that covers the Kiro CLI tools.

> **Important**
> 
> When you submit a prompt, if the activity falls within the constraints of what you have defined within the tool rule, Kiro CLI will carry out that task and you will not be prompted. However, if it falls outside of those constraints, then Kiro CLI will then trigger its standard warning message, ("Allow this action? Use 't' to trust (always allow) this tool for the session. [y/n/t]:"). The exception for this is where a tool provides an explicit deny option.

Using that reference, we can see that for the "read" and "write" tools, we can configure **"allowedPaths"** and **"deniedPaths"** to allow us to control which directories Kiro CLI has permission to read/write. You can configure explicit files, or use regex and wildcards.

> **Tip!** Some directories will be not behave as expected - any system directory will still trigger warnings despite being configured.

In the following example we can control what directories the custom agent will have read access to (without asking for permission), and which directories Kiro CLI will block us.

```
"toolsSettings": {
    "read": {
      "allowedPaths": ["~/green/*.md"],
      "deniedPaths": ["~/red/*.md"]
    }
  }
```

I created two directories (green and red) in my home directory, adding a single markdown file in each. From the Kiro CLI prompt, when I ask "how many lines are in" we see:

```
[java-dev] > how many lines are in ~/green/test.md

> I'll check how many lines are in the ~/green/test.md file for you.
Reading file: ~/green/test.md, all lines (using tool: read)
 ✓ Successfully read 8 bytes from /{path}/green/test.md
 - Completed in 0.2s

> The file ~/green/test.md contains 1 line.
```

You might be thinking this is no big deal - the read tool has default trusted permission. However, the trusted permission is only for the current working directory. You can see this when you run the /tools command:

```
- read            trust working directory
```

In the above example, we are reading from outside the working directory. With this configuration, we were able to add that additional trust.

Lets now try the red directory to see what happens.

```
[java-dev] > how many lines are in ~/red/test.md

> I'll check how many lines are in the ~/red/test.md file for you.
Command fs_read is rejected because it matches one or more rules on the denied list:
  - ~/red/*.md

> I understand the file read was rejected. You can check the line count of ~/red/test.md using
this command in your terminal:
```

Lets put this into practice now in the next lab.

**Task-05**

Lets configure our custom agent so that we limit what files Kiro CLI has access to, and explicitly block one command. Exit Kiro CLI for a moment, and go to your home directory (in most computers this is referred to as ~). Create two directories, "project-1", "project-2", and "project-3" and in each, create a dummy file called "README.md".

```
(home)
├── project-1
│   └── README.md
├── project-2
│   └── README.md
├── project-3
│   └── README.md
(all your other files)
```

From your Kiro CLI "[java-dev] >" prompt, edit your custom agent

```
/agent edit --name java-dev
```

replace the contents of your custom agent with the following:

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
  "toolsSettings": {
    "read": {
      "allowedPaths": ["~/project-1/**"],
      "deniedPaths": ["/~/project-2/**"]
    }
  }
}
```

After saving, exit Kiro CLI and restart with this custom agent

```
kiro-cli --agent java-dev
```

And now try the following prompts from your "[java-dev] >"

```
read the file in ~/project-1/README.md
read the file in ~/project-2/README.md
```

What happens? The first prompt should work, and Kiro CLI should not ask you for permission to read the file. When you try the second one, it should trigger an error like the following:

```
Command fs_read is rejected because it matches one or more rules on the denied list:
  - ~/project-2/**
  - ~/project-2/**
```

What happens when you try the following prompt:

```
read the file in ~/project-3/README.md
```

This time Kiro CLI brings up its trust prompt ("Allow this action? Use 't' to trust (always allow) this tool for the session. [y/n/t]:") - answer n to exit this. We can see that Kiro CLI is not trusted to automatically access this directory.

Edit the java-dev custom agent configuration, from your current Kiro CLI session use the following command:

```
/agent edit --name java-dev
```

Replace the configuration so that it looks like the following:

```
{
  "name": "java-dev",
  "description": "A custom agent that provides Java Spring Boot developer guidance for using Spring Boot v3.x",
  "prompt": "file://../shared/springboot.md",
  "mcpServers": {},
  "tools": [
    "read","write","shell"
  ],
  "toolAliases": {},
  "allowedTools": ["read"],
  "resources": [
    "file://AGENTS.md",
    "file://README.md"
  ],
  "hooks": {
  },
  "toolsSettings": {
    "read": {
      "allowedPaths": ["~/project-1/**"],
      "deniedPaths": ["~/project-2/**"]
    }
  },
  "useLegacyMcpJson": false,
  "model": null
}
```

Save, and then exit Kiro CLI and restart with the java-dev custom agent.

```
kiro-cli --agent java-dev
```

When it starts, you should see the following warning:

```
WARNING: You have trusted read tool, which overrides the toolsSettings: allowedPaths: ["~/project-1/**"]
```

Now try the same prompts again

```
read the file in ~/project-1/README.md
read the file in ~/project-2/README.md
read the file in ~/project-3/README.md
```

This time you should see that project-1 and project-3 work without asking you for permission, but project-2 is still blocked. 

As you start working with fine grain permission, be aware that "allowedTools" configuration is providing a blanket trust that is very broad. This is why the warning message is displayed. 

Now lets take a look at blocking certain commands from our Kiro CLI session. Edit the java-dev custom agent configuration, from your current Kiro CLI session use the following command:

```
/agent edit --name java-dev
```

Replace the configuration so that it looks like the following:



```
{
  "name": "java-dev",
  "description": "A custom agent that provides Java Spring Boot developer guidance for using Spring Boot v3.x",
  "prompt": "file://../shared/springboot.md",
  "mcpServers": {},
  "tools": [
    "read","write","shell"
  ],
  "toolAliases": {},
  "allowedTools": [],
  "resources": [
    "file://AGENTS.md",
    "file://README.md"
  ],
  "hooks": {
  },
  "toolsSettings": {
      "shell": {
            "allowedCommands": ["wc .*"],
            "deniedCommands": ["rm .*"],
            "autoAllowReadonly": true
      }
  },
  "useLegacyMcpJson": false,
  "model": null
}
```

Save and exit Kiro CLI. Before starting Kiro CLI, in the current project directory create a README.md file, and add some text to this file (it can be anything). Once you have created this file, restart Kiro CLI with the java-dev custom agent.

```
kiro-cli --agent java-dev
```

Try the following prompt:

```
use wc to count how many words are in the README.md
```

What happens? Kiro CLI should now use the shell to run the wc command and give you the answer. It has respected the allowedCommands, and not prompted us for trust.

Now try the next prompt:

```
rm the README.md file
```

What happens? Kiro CLI should block this. This is the error it displayed for me:

```
Command execute_bash is rejected because it matches one or more rules on the denied list:
  - \Arm .*\z
```

Before we finish this lab, lets see what happens if we had not configured wc as an allowed command. Edit the java-dev custom agent configuration, from your current Kiro CLI session use the following command:

```
/agent edit --name java-dev
```

Update the configuration so that you remove the allowedCommand, so that it looks like the following:

```
{
  "name": "java-dev",
  "description": "A custom agent that provides Java Spring Boot developer guidance for using Spring Boot v3.x",
  "prompt": "file://../shared/springboot.md",
  "mcpServers": {},
  "tools": [
    "read","write","shell"
  ],
  "toolAliases": {},
  "allowedTools": [],
  "resources": [
    "file://AGENTS.md",
    "file://README.md"
  ],
  "hooks": {
  },
  "toolsSettings": {
      "shell": {
            "deniedCommands": ["rm .*"],
            "autoAllowReadonly": true
      }
  },
  "useLegacyMcpJson": false,
  "model": null
}
```

Save and exit Kiro CLI, and then restart again with the java-dev custom agent.

```
kiro-cli --agent java-dev
```

Now repeat the prompt:

```
use wc to count how many words are in the README.md
```

What happens this time? This time you should be prompted for permission - as you can see, toolsSettings allows us to enable fine grain control of trust within your Kiro CLI sessions.

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

### Changing tool names using toolsAliases

In the previous section we shared how you can use **"toolsAliases"** to address namespace clashes. You can also use aliases to create shorter or more intuitive names for frequently used tools. For example, if you have some long tools you can shorten them with something like:

```
{
  "toolAliases": {
    "@aws-cloud-formation/deploy_stack_with_parameters": "deploy_cf",
    "@kubernetes-tools/get_pod_logs_with_namespace": "pod_logs"
  }
}
```

This is a nice developer experience feature that I am using with some of my custom agent tools.

---

## MCP Prompts

We have already looked at how Kiro CLI supports MCP Tools, but it also supports MCP Prompts. In earlier labs we looked at how you can use the **"/prompts"** command to list and use prompts that are stored on your machine, but you can also extend this to use prompts that are made available via MCP Servers.

To demonstrate this we are going to implement a simple MCP Server that provides Prompt resources, and then configure Kiro CLI to use this and show you how it works.

**Task-06**

Open a **new terminal window** and create a new directory (for example, "mcp-prompts").

From this window, create a file called **"mcp-server.py"** and add the following code. This will generate a simple MCP Server that provides a Prompt:

```
import asyncio
import sys

from mcp.server.fastmcp import Context, FastMCP

# Create an MCP server
mcp = FastMCP("KCLIPromptDemo")

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
        "KCLIPromptDemo": {
            "command": "uv",
            "args": ["--directory", "{path to directory}", "run", "--with", "mcp", "mcp", "run", "mcp-server.py"]
        }
    }
}
```

We now need to add this MCP Server to our custom agent JSON configuration. We will use the one we have been using throughout this lab (but feel to create a new one if you want). When I do a "pwd" I get the following working directory:

```
/{home}/{project workspace}/mcp-server
```

Which gives me the path I need for the MCP Server. I then edit this JSON configuration file so it looks like this (in the below configuration, the {add your path} will be need to be updated for your own machine)

```
{
  "name": "java-dev",
  "description": "A custom agent that provides Java Spring Boot developer guidance for using Spring Boot v3.x",
  "prompt": "file://../shared/springboot.md",
  "mcpServers": {
    "KCLIPromptDemo": {
          "command": "uv",
          "args": ["--directory", "{add your path}/mcp-server", "run", "--with", "mcp", "mcp", "run", "mcp-server.py"]
        }
  },
  "tools": [ "*" ],
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

You can see we have added the new MCP Server we created by adding the following to the previous custom agent configuration file. Again, yours will be slightly different as the directory you have created for mcp-server will be different.:

```
	"KCLIPromptDemo": {
          "command": "uv",
          "args": ["--directory", "{your path}/mcp-server", "run", "--with", "mcp", "mcp", "run", "mcp-server.py"]
        }
```

After saving this file, in the same directory, start an Kiro CLI session with the java-dev custom agent and confirm that the MCP Server has started ok:

```
[java-dev] > /mcp

KCLIPromptDemo
▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔
✓ KCLIPromptDemo loaded in 9.79 s

```

If you run **"/tools"**, what do you see? It should look exactly as previous. We have not added any new tools with this MCP Server, we have added prompts. So how do we see these? Simple, we use the **"/prompts"** command within our Kiro CLI Session.

From the **">"** prompt, type **"/prompts"** and hit return. You should see something like the following:

```
> /prompts list

Prompt                    Arguments (* = required)
▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔
KCLIPromptDemo (MCP):
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

Todo's are files that get created in your local filesystem, within your current project workspace. There is a new directory called **".kiro/cli-todo-lists"** that is created, and when you define your lists, they will appear here as json files.

Lets explore this new feature in the next lab.

**Task-07**

The first thing we need to do is enable this feature by using the /experiment mode ( [see the advanced section for more details if you missed that section out](/workshop/02-advanced-setup-topics.md) ) - move down to "Todo Lists" and then press space

```
? Select an experiment to toggle ›
  Knowledge          [OFF]  - Enables persistent context storage and retrieval across chat sessions (/knowledge)
  Thinking           [OFF] - Enables complex reasoning with step-by-step thought processes
❯  Tangent Mode      [OFF]  - Enables entering into a temporary mode for sending isolated conversations (/tangent)
  Todo Lists         [OFF] - Enables Q to create todo lists that can be viewed and managed using /todos
```

After pressing space, you should see the following.

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
- shell           * trust read-only commands
- read            * trusted
- write           * not trusted
- introspect      * trusted
- report          * trusted
- todo            * trusted
- aws             * trust read-only commands
- web_fetch       * not trusted
- web_search      * not trusted

```

Now that we have this enabled, lets try it out by creating our first Todo list.

From your Kiro CLI session, at the **">"** prompt, write the following:

```
Create a todo list to generate a simple python flask app that returns a Yoda quote of the day
```

You should see something similar to the following - note that this has invoked the **todo** tool that is now enabled when you configured this experimental feature.

```
> I'll create a todo list for building a simple Python Flask app that returns a Yoda quote of the day.


🛠️  Using tool: todo (trusted)
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

We are not going to implement this yet. In fact, we are going to create another todo list. Write the following in your Kiro CLI ">" prompt

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

---

### Tangent Mode

In the previous labs, we have explored context engineering and looked at some of the techniques you have available to you to effectively manage your context. In this lab we are going to introduce a new experimental feature that provides some useful additional capabilities to help you minimize polluting your context. What does this mean? As your Kiro CLI sessions progress, you might need to use Kiro CLI to do adjacent tasks (for example, maybe troubleshoot an issue, or perhaps dive into a rabbit hole) but those tasks might not necessarily contribute to your objective and so you might not want to include the output of those interactions in your context. 

**Tangent mode** creates conversation checkpoints, allowing you to explore side topics without disrupting your main conversation flow. Enter tangent mode, ask questions or explore ideas, then return to your original conversation exactly where you left off.


**Task-08**

The first thing we need to do is enable this by using the /experiment mode ( [see the advanced section for more details if you missed that section out](/workshop/02-advanced-setup-topics.md) ) - move down to "Tangent Mode" and then press space

```
? Select an experiment to toggle ›
  Knowledge          [ON]  - Enables persistent context storage and retrieval across chat sessions (/knowledge)
  Thinking           [OFF] - Enables complex reasoning with step-by-step thought processes
❯  Tangent Mode      [OFF]  - Enables entering into a temporary mode for sending isolated conversations (/tangent)
  Todo Lists         [OFF] - Enables Q to create todo lists that can be viewed and managed using /todos
```

After pressing space, you should see the following.

```
 Tangent Mode experiment enabled
```

From the Kiro CLI session, you can now run the command **"/tangent"**

```
> /tangent
```

You will notice that the **">"** prompt changes to **"↯ >"**, to let you know that you are now in tangent mode. What might you do when you are in tangent mode? Here are some example use cases:

* Explore alternatives approaches - you might have started along one path, but you can use tangents to explore alternatives so that they do not add to your existing context
* Getting help on Kiro CLI - if you need to ask Kiro CLI for help on some of its features, but do not want that to pollute your context
* Clarification - you might want to dive into some details to make sure that you have everything you need, but want to do this in a way that does not distract your existing context

You can check out some [additional examples use cases here](https://kiro.dev/docs/cli/experimental/tangent-mode/?trk=fd6bb27a-13b0-4286-8269-c7b1cfaa29f0&sc_channel=el)

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

You might be thinking that what if you wanted to include your tangent conversation back into the main chat conversation? Kiro CLI allows you to do this by using a new argument that will preserve the last conversation back in your main context. It works as follows - after initiating a new tangent session, after you have prompted and got a response, you can use the following command, **"/tangent tail"** which will then generate the following output.

```
↯ > /tangent tail

Restored conversation from checkpoint (↯) with last conversation entry preserved.
```

**Task-09**

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

### Context Usage Indicator

In earlier labs you learned about the importance of managing your context window, looking at how /context can help you know how close you are getting to your limit.

The Context Usage Indicator experimental features provides a visual cue, replacing the ">" prompt with an additional identifier for the percentage use of your context window. 

```
12% [java-dev] >
```

**Task-10**

From your Kiro CLI session, at the ">" prompt type:

```
/experiment
```

Move the arrow keys down to the "Context Usage Indicator" and press SPACE.

```
> /experiment

 Press (↑↓) to navigate · Enter(⏎) to toggle an experiment
  Knowledge                 [OFF] Enables persistent context storage and retrieval across chat sessions (/knowledge)
  Thinking                  [OFF] Enables complex reasoning with step-by-step thought processes
  Tangent Mode              [ON]  Enables entering into a temporary mode for sending isolated conversations (/tangent)
  Todo Lists                [ON]  Enables Kiro to create todo lists that can be viewed and managed using /todos
  Checkpoint                [OFF] Enables workspace checkpoints to snapshot, list, expand, diff, and restore files (/
❯ Context Usage Indicator   [ON]  Shows context usage percentage in the prompt (e.g., [rust-agent] 6% >)
  Delegate                  [OFF] Enables launching and managing asynchronous subagent processes
```

Once you have exited, your Kiro CLI prompt will change. If you want to revert back, just go back and toggle the same feature again and this will turn this experimental feature off.

---

### Delegate

(Coming Soon)

---

### Checkpoint

(Coming Soon)

---

### Knowledge

(Coming Soon)

---

### Supporting Resources

Some additional reading material that dives deeper into this topic if you want to explore:

* [Kiro custom agent configuration file reference](https://kiro.dev/docs/cli/custom-agents/configuration-reference/?trk=fd6bb27a-13b0-4286-8269-c7b1cfaa29f0&sc_channel=el)

### Completed!

Now that we have an idea of how Kiro CLI works and explored some of the features, we can take a look at some of the use cases in [Exploring Use Cases](03a-use-cases.md)


