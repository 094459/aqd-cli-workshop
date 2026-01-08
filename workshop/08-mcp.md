![Kiro CLI header](/images/kiro-workshop-header.png)

# What is Model Context Protocol (MCP)

At its core, MCP follows a client-server architecture where a host application can connect to multiple servers. It has three key components: Host, Client, and Server

* MCP Host represents any AI app (Kiro CLI for example) that provides an environment for AI interactions, accesses tools and data, and runs the MCP Client. The host manages the AI model and runs the MCP Client.
* MCP Client within the host and facilitates communication with MCP servers. It's responsible for discovering server capabilities and transmitting messages between the host and servers.
* MCP Servers exposes specific capabilities and provides access to data, like,

![MCP Overview](/images/mcp-response-diagram.png)

> **When is a server not a server?** Although they are called "MCP Servers", you might be thinking that this is a machine running a process or endpoint you need to connect to. MCP Servers can actually be a local process (program, executable) that you run on your local machine, and in-fact, most MCP Servers as of writing this workshop (May, 2025) run locally. When folk mention MCP Servers, they often mention the method of running those locally as STDIO.

## What does your MCP Server do?

<add links ot AWS docs>


## Running locally or remote

MCP Servers will typically document not only what they do (the services they provide) but also provide how to connect to them. You can connect to MCP Servers in two ways: running them locally, or connecting to a remote endpoint via https.

### Native or Container

I have seen two common ways that people are integrating MCP Servers locally using STDIO: they are either installing/running libraries or executables, or they are running the same libraries/executables but through a container image. Most of the examples (including my original post) showed the first way (running direct libraries). You might be thinking why would you use one method over another. It depends, but when you run MCP Servers locally you need to make sure that you meet all the dependencies. You might encounter some MCP Servers that have a lot of dependencies (libraries, binaries, etc) or maybe dependencies that clash with your setup. In these circumstances then running those MCP Servers in a container is probably a good approach.

Another good reason to run your MCP Servers in a container is to limit its access to your local machine. When I configure my dev tools to use MCP Servers, I am always using the container option, and I have written about this in some blog posts (see additional resources section at the end).

# Security

**SECURITY ALERT!!**

Be very very careful when exploring MCP Servers. They represent a new attack vector as you are enabling executables to run in your environment. Key security considerations when using MCP:
 
* Only install servers from trusted sources
* Review tool descriptions and annotations before approving
* Use environment variables for sensitive configuration
* Keep MCP servers and the Q CLI updated
* Monitor MCP logs for unexpected activity

The MCP security model in Amazon Q Developer CLI is designed with these principles:

1. Explicit Permission: Tools require explicit user permission before execution
2. Local Execution: MCP servers run locally on your machine
3. Isolation: Each MCP server runs as a separate process
4. Transparency: Users can see what tools are available and what they do


----

### Supporting Resources

Some additional reading material that dives deeper into this topic if you want to explore:

* [Kiro MCP documentatin](https://kiro.dev/docs/cli/mcp/?trk=71546b8e-c969-4ead-aa9f-9cd06f6d8610&sc_channel=el)
