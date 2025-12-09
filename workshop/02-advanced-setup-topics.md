![Kiro CLI header](/images/kiro-workshop-header.png)

# Diving deeper into Kiro CLI setup

This optional section provides a deeper look at how Kiro CLI is setup.

## Exploring Kiro CLI configuration (Optional)

This is an optional section for those of you who want to dive deeper at Kiro CLI, looking at how you can tune and configure it to your preferences. Feel free to skip this section if you want.

### Directory structure

There are two key directories you need to be aware of when working with Kiro CLI.

* Kiro CLI uses the "~.kiro" in your home directory called to store global configuration settings that are used across all your Kiro CLI sessions.

* Kiro CLI can also create project specific settings that are used just within the session you are working on. When you launch Kiro CLI from a terminal window, the current directory is called the **"Project Workspace directory"** and within this directory you might find a ".kiro" directory used to store local Kiro CLI configuration files 

Don't worry too much at this stage though, you do not need to do anything yet. I am just providing you with this background info so you understand how you can tailor and configure Kiro CLI to provide the best output. We will see how later in this workshop.

---

### Using Kiro CLI behind a Proxy

If you are running Kiro CLI behind a proxy, you should check out the [official Kiro documentation](https://kiro.dev/docs/cli/installation/?trk=fd6bb27a-13b0-4286-8269-c7b1cfaa29f0&sc_channel=el) that provides details on how you can configure this.

---

### Debugging options

When starting Kiro CLI, you can set an environment variable (KIRO_LOG_LEVEL) which will alter the level and detail output of logs which will appear in the **"$TMPDIR/kiro-log/kiro-chat-log"** log file.

```
export KIRO_LOG_LEVEL=trace
kiro-cli
```

Which will provide the most detailed level of logging. Use carefully as this will likely slow down your normal operation of Kiro CLI. The options you have are "trace", "debug", "warn", "info", and "error".

---

### Settings

The Kiro CLI settings file lives in the "~.kiro/settings" directory, and is json file that you can edit.

Kiro CLI allows you to set a number of different configuration parameters so that you can tailor how it works. You can make changes to your Kiro settings using the **"kiro-cli settings"** command, or by directly updating the configuration file.

For example, if I wanted to change the default model that Kiro CLI uses, I can use the following command from the terminal.

```
kiro-cli settings chat.defaultModel claude-4-sonnet
```

You can also open up the settings file using the command **"kiro-cli settings open"** which will open up the settings file in your default editor.

```
{
  "chat.defaultModel": "claude-4-sonnet",
  "chat.editMode": "vi",
  "mcp.loadedBefore": true
}
```

You can view your current settings, using the command **kiro-cli settings all**:

```
kiro-cli settings all

chat.defaultModel = "claude-4-sonnet"
chat.editMode = "vi"
mcp.loadedBefore = true
```

You can view the settings of individual settings using the format **kiro-cli settings {setting}**, for example:

```
kiro-cli setting chat.Mode
```

You can view the various options by using the "help" option when running this:

```
kiro-cli settings help
```
---

### Getting diagnostic info from your Kiro CLI installation

"kiro-cli diagnostic" is a very handy option that provides a summarized output of your installation setup.

```
kiro-cli diagnostic
```

Your output will be different from mine, but this is what I got when I ran this:

```
[kiro-details]
version = "1.20.0"
hash = "ac0d8dec437123a79962295fdad4122ef9642da3"
date = "2025-11-16T23:27:14.134786Z (2d ago)"
variant = "full"

[system-info]
os = "macOS 15.7.2 (24G325)"
chip = "Apple M1 Pro"
total-cores = 10
memory = "32.00 GB"

[environment]
...
...
```

---

### Experimental features in Kiro CLI

From time to time there will be features released that are initially only available if you enable them using Kiro CLI's beta/experimental mode. You can easily do this from within your Kiro CLI session by using the **"/experiment"** command, which will then allow you to toggle any of the experimental features that are currently available.

When you enter this command, you will see the following (you might see a different set of experimental features if you are using a later version of Kiro CLI)

```
> /experiment

⚠ Experimental features may be changed or removed at any time

? Select an experiment to toggle ›
> /experiment

 Press (↑↓) to navigate · Enter(⏎) to toggle an experiment
❯ Knowledge                 [ON]  Enables persistent context storage and retrieval across chat sessions (/knowledge)
  Thinking                  [ON]  Enables complex reasoning with step-by-step thought processes
  Tangent Mode              [ON]  Enables entering into a temporary mode for sending isolated conversations (/tangent)
  Todo Lists                [ON]  Enables Kiro to create todo lists that can be viewed and managed using /todos
  Checkpoint                [ON]  Enables workspace checkpoints to snapshot, list, expand, diff, and restore files (/checkpoint)
  Context Usage Indicator   [OFF] Shows context usage percentage in the prompt (e.g., [rust-agent] 6% >)
  Delegate                  [ON]  Enables launching and managing asynchronous subagent processes
>
```


You can use the **UP** and **DOWN** arrow keys and then press **SPACE** to toggle the status of the experimental feature between ON or OFF. As you can see above, I have enabled the most of these experimental features, with the exception of Context Usage Indicator" which has remained disabled.

> What this feature is doing is simplifying configuration to your Kiro CLI settings file which we have covered above. As you toggle these different experimental features you will see that they are updated in your settings.json file.

We dive into some of these features in the [advanced topics](/workshop/04-advanced-topics.md) part of this workshop.

---

**Alerting when Kiro CLI completes a task**

As you start using Kiro CLI to do more complex tasks, you may run into the situation where you leave it running in a terminal, but then have to periodically check in to see if its finished. To make this task more efficient, you can set an alert so that when Kiro CLI has completed its task, it will use your terminal alert to notify you. To enable this, you just need to run the following command from the terminal.

```
kiro-cli settings chat.enableNotifications true
```

When a task finishes, your terminal will now notify you. Depending on the terminal you are using, you can configure your alert to let you know when to come back to your Kiro CLI session.

If you want to disable this at any time, just run the same command but use "false"

---

### Supporting Resources

Some additional reading material that dives deeper into this topic if you want to explore:

* [Kiro CLI Experimental features](https://kiro.dev/docs/cli/experimental/?trk=fd6bb27a-13b0-4286-8269-c7b1cfaa29f0&sc_channel=el)


---


### Completed!

Now that you have set everything up, we can proceed to the next step which is [exploring how to get started with Kiro CLI](03-getting-started.md)
