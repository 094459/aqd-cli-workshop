![Kiro](images/kiro-workshop-header.png)

## Introduction - Next Generation Developer Tools

*A hands on guide to working with Kiro CLI. Made by DevRel with 💖.*

This is a hands on tutorial that will help you explore and get hands on with Kiro CLI. You will learn lots, and hopefully see how the next generation of developer tooling will help make our jobs as developers more enjoyable and productive.

> **You do not have to have an AWS account or be an AWS user to go through this tutorial**. 
 
This hands on tutorial is split into a number of different labs which you can run either within a controlled workshop, or at your own pace. Feel free to contact me if you run into any issues.

*Amazon Q CLI is now Kiro CLI - this workshop has been updated to reflect the change*

**What is Kiro CLI?**

How often **do** you use your terminal or shell? As a developer, I spend probably most of my time in the terminal when I am not writing code in my IDE. If you are not a developer, maybe you work as a sysadmin, or perhaps a devops engineer, you might spend all of your time in the terminal - writing scripts to automate stuff, running various tools to help with your job, and helping to fix things when they go wrong. The terminal is an essential part of our job, and so we need to make sure we are 'command line confident' as I like to say.

To many folk though, using the command line can be daunting, for many reasons. For example, trying to remember which commands are available to do certain tasks, or perhaps trying to recall the different command line parameters that command line tools require (as they change all the time).

If only we could get some help with this.

[Kiro CLI](https://kiro.dev/download?trk=71546b8e-c969-4ead-aa9f-9cd06f6d8610&sc_channel=el) brings IDE-style autocomplete and agentic capabilities to your terminal. It features:

* Auto Completion: IDE-style completions to hundreds of popular CLIs like git, npm, docker, and aws.
* Natural Language Chat: Interact with your terminal using natural language to ask questions, debug issues, or explore the codebase.
* Contextual Awareness: Integrates context from your local development environment, so answers are tailored to your specific code and setup.
* Agentic Execution: Kiro CLI can take action: generate code, edit files, automate Git workflows, resolve merge conflicts, and more — with your permission.
* Tool Integration: Kiro CLI can execute AWS CLI commands, interact with your local filesystem, and leverage specialized tools through MCP extensions, orchestrating these tools intelligently to accomplish tasks.
* Specialized Agents: Create custom AI assistants tailored for specific workflows, domains, or security requirements, with configurable tools and resources.
* Security Model: All commands that modify your system or AWS resources require your explicit approval before execution, giving you control while benefiting from AI assistance.

![splash page for Kiro CLI](/images/kiro-splash.png)

It is pretty awesome, and in this workshop you will get hands on with it so that you can learn the basics,  and give you the confidence so you can apply what you have learned to your every day activities.

---

**What you will need**

To follow along this workshop, you are going to need:

* A Windows (with Windows Subsystem for Linux - wsl configured), MacOS, or Linux machine with the ability to install software
* A supported terminal (most are supported) and supported shell - bash, zsh, or fish
* You will need some standard developer tools: git, [uv](https://docs.astral.sh/uv/getting-started/installation/), npx
* You will need Python 3.10.x or later
* (optional) A code editing tool (Zed, VSCode, etc) - this will make it easier to work with files that are generated, but vim also works



You will doing the following as part of this workshop:

* Create a Builder ID account, which will allow you to use Kiro CLI for free
* Learn how to install, configure, and understand the many features of Kiro CLI
* Use Kiro CLI to work through a number of use cases (writing code, automation, etc)

---

**Overview**

This workshop will first of all get you up and running with Kiro CLI - installing, configuration, and understanding the different parts that make up Kiro CLI. Once we have had some time to explore its capabilities, we will use that knowledge against a number of typical use cases where you might use it.


## Installation and Setup

[Follow this link](workshop/01-setup.md) to get started with installing Kiro CLI. You do not need an AWS account to use Kiro CLI. You can use the free tier by [creating a Builder ID](https://s12d.com/builder-id), and then using that Builder ID to login. You can also use Kiro with your GitHub or Google account.

You can then (optionally) [explore more advanced setup topics](/workshop/02-advanced-setup-topics.md) to get a deeper understanding of how Kiro CLI works, looking at important directories and files as well as how to configure various settings.

---

## Learning and exploring Kiro CLI capabilities

Once we have Kiro CLI installed, setup, and configured, we are ready to explore its capabilities. In this lab we are going to go over the features in detail and provide your first opportunity to get hands on.

[Follow this link](workshop/03-getting-started.md) to explore the wonderful world of Kiro CLI, including getting hands on with some exercises.

Once you have learned the basics, you can dive deeper and [explore more advanced capabilities](workshop/04-advanced-topics.md).

---

## Writing code with Kiro CLI 

In this lab we are going to use some of the things we have learned to help use write some code using Kiro CLI.

[Follow this link](workshop/07-writing-code.md) to get started.

**Spec Driven Development**

If you are interested in more hands on tutorials on how to use AI coding assistants to write code, I have written [a spec driven development](https://github.com/094459/sdd-workshop) workshop that dives into using Kiro IDE to go through both greenfield and brownfield development use cases.

---

## Reviewing code

In this lab we will take a look at how you can use Kiro CLI to review your project. Whether this is using Kiro CLI to help you review code, run third party scanning and review tools more efficiently, we can use Kiro CLI to be our second pair of eyes. We are going to build from the previous lab, although you can follow along with your own projects if you prefer. 

[Follow this link](/workshop/12-reviewing-code.md) to get started.

---

## Automation

In this lab we are going to look at automation use cases, exploring some of the ways you can use Kiro CLI in your day to day activities.

[Follow this link](workshop/05-automation.md) to start diving into automation.

---

## Documenting Codebases

In this lab we will explore how we can use Kiro CLI to help with documentation tasks. Beyond just documenting the project you are working on, you will see how you can use it to get a better understanding of any code base you want to work with.

[Follow this link](workshop/06-understanding.md) dive into this lab.

---

## Debugging

When you write code, you are going to encounter problems. As good as Kiro is at writing code, issues will come up and need your help to fix. In this lab we will look at effective debugging and troubleshooting techniques when using Kiro CLI.

[Follow this link](workshop/09-debugging.md) dive into this lab.

---

## Feedback!

Before you go, if you found this tutorial useful/helpful, or have ideas how we can make it better, please please provide some feedback using [this link](https://pulse.aws/survey/1DM5TAZU).

-

-

[view changelog](/workshop/changelog.md)
