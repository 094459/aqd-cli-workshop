![Kiro CLI header](/images/kiro-workshop-header.png)

## Setting up, and getting starting with Kiro CLI

We are now ready for the first hands on part of this lab, installing and setting up Kiro CLI. In this lab we are going to:

* Register a Builder ID which we will need to use Kiro CLI
* Download and install Kiro CLI
* Explore Kiro CLI configuration options

Kiro also supports logging in with GitHub and Google credentials (for Linux or MacOS users). For headless setups (Windows via WSL) you will need to use a Builder ID or Identity Centre (Idc) account.

### Creating your Builder ID

(If you already have a Builder ID, you can skip this step)

[Creating a Builder ID](https://s12d.com/builder-id) is the first step in being able to use Kiro CLI. All you need is a valid email address to create a Builder ID. Head over to the [Builder ID page](https://s12d.com/builder-id) and click on the "Sign in with Builder ID". This will pop up a browser window where you can now create your Builder ID, using your email address. You can follow [this short video](https://youtu.be/i870XzFCyGI) to help you understand the flow if you get stuck.

---

### Installing Kiro CLI

Now that we have our Builder ID we can download, install, and then configure Kiro CLI. Before we do that, you need to talk about terminals and shells.

**Terminals and shells**

The command line is a text based user interface that you can use to interact with your system. You can run commands to make the system do something – run a tool, start an application, edit a file. These commands interact with the computer operating system, and nearly all computer operating system has some kind of command line.

When you use the command line, you use a piece of software called a terminal. When you open up that terminal it will run something called a shell (whether that is Bash or Zsh on Linux/Mac or command prompt or Power Shell on Windows). Kiro CLI runs in a terminal, configured and enabled via the shell. It supports most terminal applications (for example iterm2, Ghostty, tux - you can check out the source code for the complete list), and supports three shells today: bash, zsh, and fish.

---

**Installation options**

**Windows**

To install Kiro CLI on Windows, you will need to use Windows Subsystem for Linux (wsl). I have put together an installation guide you can follow - [Installing Kiro CLI in wsl](https://dev.to/aws/the-essential-guide-to-installing-amazon-q-developer-cli-on-windows-lmh)

**MacOS**

To install Kiro CLI on MacOS, [follow the instructions on the Kiro.dev documentation site](https://kiro.dev/docs/cli/installation/?trk=71546b8e-c969-4ead-aa9f-9cd06f6d8610&sc_channel=el#macos)


**Linux options**

To install Kiro CLI on Linux, [follow the instructions on the Kiro.dev documentation site](https://kiro.dev/docs/cli/installation/?trk=71546b8e-c969-4ead-aa9f-9cd06f6d8610&sc_channel=el#linux-appimage). This covers Ubuntu and other distributions you might be using.

---

### Logging in

The final step after we have installed Kiro CLI is to now login. Kiro CLI supports a number of different login methods, but for this workshop we are going to use the Builder ID we created earlier.

From the command prompt we do this by entering:

```
kiro-cli login
```

If you are using a GUI Linux system or a MacOS, a browser window will pop up and display the login options.

![Kiro CLI login options](/images/kiro-login.png)

Click on Builder ID option. You will now be asked to login with your Builder ID (if you created one for this workshop and left the session open, it will just skip). After entering your credentials, you will be asked to confirm that you are happy to provide Kiro with access.

![Kiro Builder ID login flow](/images/kiro-builder-id-login.png)

If you are using a headless setup (Linux or Windows WSL) you will see something different. In the terminal you will be presented with the following options.


```
? Select login method ›
❯ Use for Free with Builder ID
  Use with Pro license
```

Select the first option (Use for Free with Builder ID) and you will be provided with a web url which you can use to login, prompting you to login with your Builder ID.  

```
? Select login method ›
❯ Use for Free with Builder ID
  Use with Pro license

Confirm the following code in the browser
Code: PNDN-QVKB

Open this URL: https://view.awsapps.com/start/#/device?user_code=PNDN-QVKB
▰▰▱▱▱▱▱ Logging in...
```

Accept the pop ups that appear, and once you have completed those steps (it was say you can now close that browser window), returning to the terminal you should now see

```
Logged in successfully
```

Kiro CLI supports some more advanced login flows which you can see when you run the "kiro-cli login --help" command. For example, you can use the following command to simplify the login experience for users using AWS Identity Centre credentials.

```
kiro-cli login --use-device-flow --license pro --identity-provider {your IdC login URL} --region {your region}
```


**Logging out**

You can log out at any time by running the following command:

```
kiro-cli logout
```

---

### Confirming installation

To confirm that Kiro CLI is installed correctly, you can use the "q doctor" command. Run it in a new terminal window, and if everything is configured correctly you should get something like

```
kiro-cli doctor

✔ zsh ~/.zshrc integration check
✔ Everything looks good!

  Kiro CLI still not working? Run kiro-cli issue to let us know!
```

You can confirm the version you are running by typing the following:

```
kiro-cli -V
```

which should produce the following. I run this just post Kiro CLI launch, so by the time you are using this workshop you might have a newer version

```
kiro-cli 1.23.0
```


---

#### Updating Kiro CLI

You can update Kiro CLI to the latest version by running the following command, which will check for updates.

```
kiro-cli update
```

 You will be prompted to upgrade (Yes/No) which will then install the latest version for you automatically (if you proceed). As Kiro CLI is an active project, make sure to check for updates and make use of the new capabilities as they are released.

You can also view the latest changelog from the command line by typing the following command:

```
kiro-cli version --changelog
```

or if you want to dive into a specific release

```
kiro-cli version --changelog 1.20.0
```

---

#### Finding your Kiro CLI chat history

Kiro CLI records each command you enter in a history file that is located in the **"~.kiro"** directory. This file is called **".cli_bash_history"** and you can view this to see all the previous prompts and commands you issued.

---

#### Creating alias for Kiro CLI

You can create alias files in your shell to make accessing Kiro CLI easier. Here is what I have in mine:

```
alias k='kiro-cli'
alias kh='cat ~/.kiro/.cli_bash_history'
```

This allows me to simply type "k" to access Kiro, or "kh" to list my chat history.


---

### Supporting Resources

Some additional reading material that dives deeper into this topic if you want to explore:

* [Official Kiro documentation on kiro.dev](https://kiro.dev/docs/cli/?trk=71546b8e-c969-4ead-aa9f-9cd06f6d8610&sc_channel=el)

* [Upgrading from Amazon Q CLI to Kiro](https://kiro.dev/docs/cli/migrating-from-q/?trk=71546b8e-c969-4ead-aa9f-9cd06f6d8610&sc_channel=el)

---

### Completed!

You can optionally dive deeper into [more advanced setup options](/workshop/02-advanced-setup-topics.md).

Now that you have set everything up, we can proceed to the next step which is [exploring how to get started with Kiro CLI](03-getting-started.md)
