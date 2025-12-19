![Kiro CLI header](/images/kiro-workshop-header.png)

## Writing good Prompts

There is a temptation when using AI Coding Assistants like Kiro CLI to dive right in and think that you do not need to invest time in writing Prompts. I would encourage readers to rethink this, and spend time BEFORE using these tools to write code. Why? Time invested here will help you better understand what it is you are trying to do, and as a result better direct AI to help produce good output.

**Good Prompts are not enough**

Before we look at Prompts however, it is important to note that good Prompts alone are not enough! This is a fundamental and critical point of understanding if you want to maximize the alignment and quality of what you get from these tools. So if Prompts themselves are not enough, what do we need? The answer is a lot more simple than you might think.

> **Prompt** + **Context** = **Output**

The effectiveness of AI assisted coding is strongly dependent the quality of your Prompt, and how skillfully you can provide the right supporting context to the underlying large language model (LLM).

Whilst this looks simple written down, as you start using these tools with your own code and projects, knowing how to manage these is a journey that you will need to take. Learning by doing, by experimenting is key. This section will share some of the things you can use to bootstrap your learning, and get a head start.

>
>**The dangers of too much Context**
>
> You might be thinking that if Context is so important, why don't I just give the AI Coding Assistant everything and let it figure it all out. Whilst that might be an option in the future, where we are today is that we are limited in how much we can provide them. Both from a capacity perspective (for example, Amazon Q CLI has a context window of 200K tokens), but also from a practical perspective - currently, LLMs have a habit of going down rabbit holes when you provide too much context. The key is balance, and that is something that you will get an intuition for over time as you use these tools more regularly.
>

## Fundamentals

A well crafted Prompt centres on clarity. AI Coding Assistants produce better results when given precise instructions and a clearly defined scope. Consider these essential elements when designing your prompts:


**1. Specificity and clarity**

AI Coding Assistants want you to provide as much detail and clarity about the task you want to it do do.

* **Provide detailed and un ambiguous direction** - you should provide as much detail as you can, making sure you provide specific details. The more **relevant** details you leave out, the more you are leaving for the LLM to fill in the blanks.
* **Break down problems into smaller tasks** - there is a temptation to ask AI Coding Assistants to deliver the end result you are looking for. For larger problems, you will get better results in trying to break down that problem into a series of tasks, and then tackle them one at a time.
* **Create Prompts to tackle specific tasks** - you will find over time that certain tasks can be achieved very effectively with the same Prompt, so you should look to re-use these across your teams. Typically these might be tasks like creating documentation in a specific way, or generating diagrams of your projects, but there are other common Prompts you will find yourself returning to.
* **Iterate in a linear style** - the output of previous tasks (and the Prompt used) is provided in subsequent requests, which provides useful Context and helps ensure that the output generated aligns and makes sense. Do not clear context or use new sessions to carry on working on tasks (unless you encounter issues or hallucinations that require a reset)
* **Define your constraints** - outline any restrictions, for example: "Do not implement sections outside the specified requirements."
* **Role** - Specify the required expertise or persona. For instance: "You are an experienced software engineer with specialized knowledge in Python."

Things to avoid:

* **Brief and ambiguous Prompts** - the less detail you provide, the more "Vibe" the response will be and the less in control. This might be useful in certain situations (divergent thinking, for example when trying to come up with novel, new ideas or approaches), so make sure that you understand when and why you might use this approach
* **Too broad a question or problem** - tackling too large a problem will tend to generate unpredictable output, depending on factors such as novelty of the problem, details that are domain specific, or where there are too many "it depends" type questions that you might typically ask a person when asking the same question or problem.


**2. Providing supporting information - Context, Context, Context**

Depending on the tool you are using, you might need to provide Context within your prompt, or provide it as a document (typically in markdown format) within your project workspace. When providing Context, you can look at this as both functional and non functional. Your output is critically dependant on your context. Make sure you:

* **Describe your situation** - you should provide clear background details that describe your project, look at and explain your functional and non functional requirements, and outline your constraints.
* **Provide additional context that might not be in the code** - AI Coding Assistants cannot read minds (yet!) so do not make any assumptions and provide any additional information and Context that you think they might need. A good example of this is when working with new libraries you might want to use. You can add these to your local project workspace as Context - check out the supporting resources at the end of this lab for a deep dive on this
* **Define project standards via scaffolding documents** - defining key pieces of information in markdown documents, which I call scaffolding docs, help root the AI Coding Tools suggestions and recommendations. Refine these over time based on your own or organizational needs. Re-use these across your organization, either directly through the tooling or through third party tools and resources that are emerging to help.
* **Markdown is your new friend** - Context tends to be written in markdown documents within your project workspace - you are going to be writing a lot of these docs. Plugins to convert from Word/Google docs to markdown exist to enable you to re-use existing documentation.
* **Review MCP Server** - MCP Servers are becoming a critical way of providing Context to your AI Coding Tool, so make sure you have a plan on how to use these sensibly and with the right guardrails. MCP Servers to potentially allow you to run un-sanitized code, so you should plan accordingly.
* **What is your data model?** - Data models provide a great foundation for Context, so think about providing the right level of detail for the task you have to hand.

**Avoid**

It is tempting to include as much information as possible, but remember, Context is a finite resource and the more information that the AI Coding Tools needs to look at, the more potential there is for missing relevant information to be provided. 

* **Do not include information not relevant to the task at hand** - this will consume valuable capacity from your Context window.
* **Too much Context can make things worse** - be selective.
* **Check limits** - AI Tools have context limits (or costs) so beware.
* **Large projects** - When working with large coding projects, this can lead to limits being exceed. Zoom into specific part of the project directory structure if you can.

**3. Provide examples of expected inputs and outputs**

The final good practice when thinking about Prompts is to provide good examples that will help your AI Coding Assistant to follow what **you WANT** not what **IT** wants. Specifically:

* **What are your expected Output?** - If the task you are asking the AI Coding Assistant to help with requires something specific, then you need to articulate this. A good example here is the project layout - if you have a desired or required structure you want it to follow, you will need to provide these as examples.
* **Specify how you want the information presented** - You might have some specific need to ensure that the code you generate is presented in a certain way (for example, what a return statement needs to provide, ensuring that any new code matches existing code in your project, or perhaps the format and structure of data within your application). Unless you provide examples, you will be left at the mercy of the LLM and whatever prior art was used in its training data. Provide an example of how you want output generated to guide and align the LLM on what you want.
* **Provide example input and output data formats** - Be explicit in providing expected inputs for your code, but also what you expect the code to provide as output. Providing an example will help you shape and control your applications data formats. This extends to including data formats and types as well.
* **Provide sample code snippets or examples you want it to follow** - AI Coding Assistants are great at taking existing example code you have, and using that as a template. Create samples of how you want the code to look - include doc-strings for documentation, style preferences, variable names standards, etc.

You need not include every element in every prompt. Use this as checklist to help create well-formed prompts that yield accurate and consistent results.

---

**Examples**

We want to deploy our application on a container on AWS. Lets look at good and bad examples of what a Prompt (and supporting Context) would look like

 *Bad Example*

> How do I deploy a container on AWS?



 *Better Example*

> I need to deploy a containerized Node.js e-commerce application that handles 50,000 daily users with peak loads during promotional events.
> Requirements:
> - High availability across multiple regions
> - MongoDB for persistence
> - Auto-scaling capabilities
> 
> Please provide:
> 1. AWS architecture diagram
> 2. List of required services with configurations
> 3. Security best practices
> 4. Operational monitoring recommendations

---

## Practical Guidelines

The following provides some practical information that you might find helpful in improving your Prompts.

**Meta-Prompting**

**Meta Prompting** is where you ask the AI Coding Assistant (or any other LLM) to refine or improve your Prompt.This is a highly effective technique, and my colleague Dennis Traub has written about this in his post, [AI-Powered Prompt Engineering: Create the Perfect Prompt - Every Single Time!](https://community.aws/content/2hVZaVgpovhzdi5ijY12ZKDPGBc/ai-powered-prompt-engineering-create-the-perfect-prompyt-every-single-time?trk=fd6bb27a-13b0-4286-8269-c7b1cfaa29f0&sc_channel=el). Apply this approach to validate and test your Prompts.

This technique is also useful for generating variants of your Prompts. When you are uncertain about the optimal phrasing of your Prompt, you can request multiple versions of the prompt. For example, put your prompt in a markdown document and then use something like "Provide three alternative prompts for the Prompt in xxxx.". You can also switch to using different Models (for example, switching between Sonnet 3.5, 3.7 and 4.0 which Kiro CLI allows you to do) to see if you get different (improved) output.

**Narrow the Scope as Complexity Increases**

As tasks or codebases become more complex, narrow your prompt to focus on specific components or objectives. Reference particular documents or files within your prompt instead of relying solely on the model’s internal retrieval capabilities.

**Think Multi-modal**

Kiro is able to understand information from a variety of formats, from markdown documents within your project workspace, to architecture diagrams. Check out the supporting resources below for a list of curated content that dives deep into this topic.


---
## Managing your prompts

There are tools that help developers manage their Prompts. AWS Community Builder, **Christian Bonzelet** has created an incredible resource  called [Promptz](https://www.promptz.dev/). You can login and save your own prompts, as well as see what prompts others have created and try these out. Even better, you can connect to the MCP Server for tools and prompts. We use this in the Getting Start lab for MCP Servers.

Check this incredible resource out - learn from the community, and be sure to share back too. Whether you use Promptz or something else, the key thing here is to make sure that you figure out ways in which you can maximize the return on investment in creating good Prompts.

