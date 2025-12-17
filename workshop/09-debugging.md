![Kiro CLI header](/images/kiro-workshop-header.png)

# Debugging and Troubleshooting with Kiro CLI

Did you know that according to [getdx.com](https://getdx.com/guide/ai-assisted-engineering/)(signup required), debugging issues with code is the top use case of AI coding assistants. In this lab we will look at how to use Kiro CLI to debug and troubleshoot issues.

## Overview

The first part of this lab is theory, where we share some tips and good practices. To finish we will try and implement some of these in the customer survey application that this workshop has been working with (which you can build from the [Writing Code](/workshop/07-writing-code.md) part of this workshop, or check out a version of this in GitHub [here](https://github.com/094459/kiro-customer-survey))

Using tools like Kiro CLI to debug and troubleshoot comes down to:

* Describing the problem with the right level of precision and detail
* Providing supporting information (context)

In the following sections we will explore both. 

**Fixings vs Helping**

Before you proceed, there is one final thing to consider. When using tools like Kiro CLI to debug and troubleshoot, think about whether you want it to fix the problem, or whether what you really need is help you understand what the issue is. AI coding assistants can do both, but in my experience I have seen better results pivoting from asking "Fix it!", to asking it, "What is causing this issue?" and then getting additional information that can help me understand how plausible a solution might be (and then going ahead and fixing it). 

## 1. Define the problem

Using Kiro CLI to help you debug issues works best when you provide structure. In the same way that we have learned how to structure context, break down problems into smaller tasks, and use AI coding assistants to write code, we need to learn how to take a similarly structured approach when using them to debug and troubleshoot.

The first step in using tools like Kiro CLI to help you debug and resolve errors is to understand the actual issue. This is critical as it will allow you to provide the right context and information, which is essential for a good response.

If you are debugging some issues with code (either new code or changes to existing), then you need to provide the right framing. For example: 

* Describe what happens to generate the error
* When did the issue start?
* What changed right before things broke?
* What was the expected vs. actual behavior - “Expected JSON, got HTML.”
* Is it affecting one user or many?
* Is the issue in the code or something external that it depends on?
* Ask for reasoning - “What could cause this?” instead of “Fix it.”

This will allow you to craft a detailed prompt that will be more effective in troubleshooting. (see next section)

If you are debugging more complex issues or systems, then you can use Kiro CLI to help you with structured troubleshooting approaches - for example Root Cause Analysis. You can create a custom agent where we define a specific purpose using system prompts. We will take a look at how to do that in the next hands on activity.

---

**Task-xx**

<create a custom debug agent>


---

### Understanding your codebase

Add a section here about this as a baseline for debugging
refer to the understanding your code / project lab

I cannot tell you how many times I am sitting there, slinging obscenities at my poor AI assistant, saying: "UNDERSTAND THE CODE BEFORE YOU TRY TO FIX IT." 

PROBLEM: An activity like debugging sometimes requires you to look at a lot of files. 




## 2. Prompt design

Prompt design matters. When trying to fix errors that come up, it is critical to structure the prompt so that it provides the right direction for Kiro CLI.

I have found that I re-use certain prompts/phrases that seem to work well. This is useful in conjunction with providing the right context (next section). 





be specific

Initial Prompt: "Here is my code and the error. What are some potential causes?"
Follow-up Prompt: "You suggested it might be a type mismatch. Can you explain where in the code that mismatch could be happening?"
Final Prompt: "Okay, I see the issue. Can you suggest a more efficient way to refactor this function to prevent this error in the future?"

leverage examples

Bad Prompt: "Fix this broken function."
Good Prompt: "Here is a broken function calculateTotal(). It's not working correctly. For reference, here is a similar function, calculateSubtotal(), which works perfectly. Please compare the two and explain the error in calculateTotal()."


Chaining Thoughts: For complex analyses, ask the AI to “Think step-by-step before answering.” (e.g., “First, identify the error type. Second, pinpoint the line. Third, suggest a fix.”)

Iterative Refinement: If the first response isn’t great, refine your prompt and try again, perhaps providing examples of good output.


One final thought. Think about how you are going to manage your prompts - there is nothing worse than needing to work on an urgent issue, and then lose time trying to figure out where you kept all you great debugging prompts and resources.

Make sure you start to collect as an organization useful prompts that work within your developer communication channels. If you don't have anything today, why not try [promptz.dev](https://www.promptz.dev/), a community developed tool that is a great starting point.

---

## 3. Debug tasks one at a time

When you have several bugs that might come at you at once, the temptation is to just ask Kiro to "fix the bugs". Whilst this might be effective in small, less complex issues, your results will vary as size and complexity increase. The trick is to use your skill as a developer and identify discrete issues and then debug those one at a time.

A good way to manage this is by creating an "issue-log.md" file and then using this to drive your debugging activities. I have provided an example [issue-log.md](/resources/issue-log.md) that I use. I create the initial file in a suitable location, and add this to the context of the debugging session I have started. During testing, I add issues to this doc. I then use this to drive the troubleshooting and debugging efforts (one at a time).

It is important to time box debugging activity. If you are spending more than 30 mins on an issue, you should pause and take a step back. If you have already tried the other techniques in this doc, then you might want to use checkpoints or git to revert code changes (if debugging is down to changes you have mode).


## 4. Adding instrumentation

A good technique depending on the code base you are working on is to get Kiro CLI to help you add debugging instrumentation to the existing code base. Adding structured logging within the code is something that can be done in minutes, allowing you to capture additional information that might help you identify the issue, or be something that can be used by Kiro to understand the issue and then suggest a fix.

Model Context Protocol (MCP) Servers are potentially useful in this situation. There are many MCP Servers that provide tools to help you debug and troubleshoot issues, sometimes by directly accessing systems (for example, PostgreSQL MCP servers that connect to a database with a set of limited access credentials and then help you resolve issues you have). If you take this approach, creating specific custom agents that are dedicated to trouble shooting make sense. MCP Servers love to eat through your context window, so keep this as a separate persona, using it as you need.

> **Note!** Kiro CLI supports a useful experimental feature called "/tangent" that is very useful for these kinds of debugging use cases, and help you reduce overloading your context window by spawning a new conversation with a few context window.





## 5. Provide the right context

The most effective way to get better debugging results from using Kiro is to make sure you are providing the right context. Here are some pointers.

They described the setup precisely, including language, version, framework, and the point at which the error occurs.


### Getting more context

You do not have to limit yourself to these. You can try meta prompting, asking Kiro for "What information do you need to debug this issue", or "What are likely reasons why I would get XXXXX"?


### x. Integrate with a Language Server Protocol (LSP)

### x. Create a custom agent for debugging


### x. Adding context




- What is happening when the error occurs - the biggest failure from using Kiro is to provide only part of the information needed to help provide a good solution. For example, just providing a stack trace and asking for Kiro to fix the issue. You should always provide situational awareness when asking for a fix. For example, "when I do x in the web browser, I expect y, but get z and this is an error/stack trace that is produced". The same applies for non user issues, for example "when the system x is running, y event happens, and this stack error is produced".

- Provide details about changes you made - If you made changes, then add the change you made as context. Be specific: what files did you change and why. Add the previous state if you can (this is where providing git diff as context can be helpful)

- Provide the right context within your project - Kiro provide lots of ways to provide specific context about the projects you are working on. These are very effective and helping Kiro prioritise where to look. For example, open up the specific file in the editor if you suspect that is where the issue is. Sometimes it might be easier to copy down logs and make them available, other times and if possible, you can use MCP servers to "connect" to a system. (e.g. CloudWatch)

```
k --agent debug
"review the cloudwatch log group "kiro-clo-debugging-demo" for any anomalies in eu-west-1"
```

- What was the expected behaviour - Make sure you add what the expected behaviour is supposed to be as context. I often see this as a miss.

- Kiro/Q love patterns - we no longer need to print to console.log to help us debug. We have over instrument our applications (use your favorite logging framework), and make it configurable.  Whilst you don't want to go through all logs generated, AIs love this as context. Observability is very useful in this use case, and you can turn it on/off as needed. This extends to debugging systems that might centralise logs (CloudWatch)


- Use more than logs and stack traces to help you debug issues - You can  attach screenshots (and annotate them), providing text to explain what the issue is. This can be useful when other ways of trying to explain an issue are imprecise.

- Don't forget browser logs! - Using browser debugging tools, you can use the output of these (Inspector/Console) to provide lots great context and errors that might be missing from the server side.

- Clearing context and starting again - Sometimes context can get confused, so open a new chat window and try again. There can be lots of reasons for this, so using /clear and re-starting is something to try if everything else has failed. This technique tends to work better when combined with other tips in this doc (for example, maintaining an issue log)


---

## Try different models

Kiro offer different models. If you are struggling to get to a resolution, try different models. Starting with Auto, work to increasingly more sophisticated models as these have different reasoning capabilities as well as context windows.

---

## x. How to undo code changes successfully

During debugging you may end up in a situation where you have spent more time than you have to resolve the issue. At this point you will need to think about how to go back to your working state and start again.

* Use source control - one way you can revert changes is to use git (or your preferred tool) to revert back to the previous working code. When using AI coding assistants, this is why you should work on discrete tasks and small blocks of code. This will limit the amount of change and allow you to revert back and then try again.

* Use Checkpoints and Restore - Kiro provides a feature (currently experimental) called checkpoints that allows you to manage project workspace checkpoints, essentially allow you create checkpoints and then to quickly restore back to a checkpoint 

**Do not rely on your AI coding assistant to undo changes via the prompt - this is too unreliable.**

---

**Task-xx**



---

## x. Other kinds of debugging

Deciphering Vague Errors: Turning a cryptic error code into a plain-English explanation.
Spotting "Code Blindness": Identifying obvious typos or syntax errors that you've stared at so long you can no longer see them.
Tracing Complex Logic: Walking through a function's execution path to find where the reasoning went wrong.

 "Rubber Ducking" with AI:
Classic "rubber duck debugging" involves explaining your code, line-by-line, to an inanimate object. The act of verbalizing your logic often reveals the flaw. AI is the ultimate rubber duck because it can talk back.

Prompt: "I am trying to debug this complex function. I'm going to explain my thought process for how it's supposed to work. Please listen to my explanation and point out any logical fallacies, incorrect assumptions, or potential edge cases I might be missing."


---

## Completed!

Head back to the [homepage](/README.md) and see what other labs you might want to try out.

