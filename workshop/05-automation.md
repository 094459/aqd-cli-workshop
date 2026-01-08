![Kiro CLI header](/images/kiro-workshop-header.png)

# Automating developer tasks

In the previous labs ([Writing Code](/workshop/07-writing-code.md) and [Reviewing Code](/workshop/12-reviewing-code.md)) we built and reviewed a simple application using Kiro CLI. In this lab we are going to look at how we can use Kiro CLI to automate some of the developer tasks you might do once you have got to the stage where you are happy to ship your code.

You are going to learn:

* how to use Kiro CLI to package the application as a container image
* creating automation scripts using Kiro CLI that automate the package and building steps
* see how we can use Kiro CLI to automate the package and building of our application using GitHub actions
* automation using custom hooks

In addition to the tools you have already been using throughout these labs, you will also need to install and configure the following tool for your environment:

* GitHub - you will need a GitHub account as we will using this as our source control
* [gh](https://github.com/cli/cli) - the official CLI for GitHub, we will be using this to create a repo on our GitHub account
* Docker (or alternative like Finch, Podman, Rancher) - we will be running containers in this lab so we need the ability to run them with an appropriate tool

If you don't want to use GitHub then you can complete the first part of this lab and leave the last section.

## Overview

In this tutorial we are going to start by showing some of the capabilities within Kiro CLI that help with automation - both automating how you set the tool up, as well as automating some of the developer tasks.

Once we have finished going through those, we are going to take the application we created (customer survey app) and package this up so that it will run in a container. We will need to need to make a few changes to our code.

---

### Preparing your local project workspace

We will be checking out [this code repo here](https://github.com/094459/kiro-cli-workshop-customer-survey-app) as part of this lab. The code follows the previous [Writing Code](/workshop/07-writing-code.md) lab, and is the output I generated when I ran through the steps.

---

**Task-01**

If you have not completed the Writing Code lab and want to dive straight into this lab, carry out the following steps. If you **have** completed that lab and want to continue using the code and project workspace, skip to the next task.

From a terminal, create a new project directory that we will use during this lab. 

```
mkdir kiro-cli-automation
cd kiro-cli-automation
```

Now check out the repo we will be working with

```
git clone https://github.com/094459/kiro-cli-workshop-customer-survey-app.git
cd kiro-cli-workshop-customer-survey-app
```

Your project workspace (kiro-cli-workshop-customer-survey-app) is now ready. Throughout this lab when we refer to the project workspace, this is the directory you need to be in.

---

## Using Kiro CLI to make this application ready for production

In previous labs you might have noticed the following message appear when you started the customer survey application:

```
WARNING: This is a development server. Do not use it in a production deployment. Use a production WSGI server instead.
```

We are going to ask Kiro CLI to update the project so that it will run with WSGI, and more specifically gunicorn which is a well known approach to doing this. We are going to configure our Flask application to use Gunicorn, a common approach for running Flask applications in production. We need to do this to make sure that when we package up the application within the container, the application is able to run properly. You might have noticed that our current application is bound to a local port (127.0.0.1:5000) which is fine for local development, but would not work very well in a container.

---

**Task-02**

From your project directory, open a new terminal session, and then start Kiro CLI using this command:

```
kiro-cli --agent customer-survey
```

Once Kiro has started, from the "[customer-survey]>" prompt type the following:

```
Update the project so that it uses gunicorn to run.
```

Follow along with Kiro as it asks you for permission to make changes to some files. You should see that it updates a few files and creates some new ones. Once finished it will provide you with a summary - whilst yours may be different, it should be along similar lines to the following:

```
> The project is now configured to use Gunicorn. Here's what I've added:

1. Added Gunicorn dependency using uv add gunicorn

2. Created gunicorn.conf.py with production-ready settings:
   - Binds to 0.0.0.0:8000 for external access
   - Uses 2 workers for better performance
   - Includes timeout and request limits

3. Updated README.md with instructions for both development and production modes

You can now run the application with:
- **Development**: python run.py (Flask dev server on port 5000)
- **Production**: uv run gunicorn -c gunicorn.conf.py run:app (Gunicorn on port 8000)

The Gunicorn configuration is minimal but production-ready with appropriate worker count, timeouts, and request limits.
```

From a new terminal window, test this out and make sure that both the development and production approaches to starting the application work. You should see something like this:

```
uv run gunicorn -c gunicorn.conf.py run:app
[2025-12-11 12:52:11 +0000] [40260] [INFO] Starting gunicorn 23.0.0
[2025-12-11 12:52:11 +0000] [40260] [INFO] Listening at: http://0.0.0.0:8000 (40260)
[2025-12-11 12:52:11 +0000] [40260] [INFO] Using worker: sync
[2025-12-11 12:52:11 +0000] [40261] [INFO] Booting worker with pid: 40261
[2025-12-11 12:52:11 +0000] [40263] [INFO] Booting worker with pid: 40263
^C[2025-12-11 12:52:30 +0000] [40260] [INFO] Handling signal: int
[2025-12-11 12:52:30 +0000] [40263] [INFO] Worker exiting (pid: 40263)
[2025-12-11 12:52:30 +0000] [40261] [INFO] Worker exiting (pid: 40261)
[2025-12-11 12:52:30 +0000] [40260] [INFO] Shutting down: Master
```

To exit, press CTRL+C which should return you to the prompt.

---

## Containerizing our application

We have prepared the application so that it is container friendly, but there are still a few more things we need to do. When containerizing applications, it is often a good idea to create an entrypoint script (entrypoint.sh) so we will see how Kiro CLI can help us with that.

**Task-03**

From our Kiro CLI terminal session, at the "[customer-surve]>" prompt, lets ask Kiro CLI to help generate the entrypoint.sh script.

```
This project is going to run via containers. Create an entrypoint.sh script that I can use.
```

Follow along and review the script that is created. Kiro CLI will ask for permission to create files and make them executable.

Given the non deterministic nature of how tools like Kiro CLI work, you might find that it only creates this script. However, it may also create additional artifacts, such as a Dockerfile or Docker Compose files. This is a snippet of the output generated when I ran this:

```
> Created entrypoint.sh with minimal functionality:

- **Database initialization**: Creates tables if database doesn't exist
- **Gunicorn startup**: Uses the existing configuration
- **Error handling**: set -e ensures script fails on errors
- **Executable permissions**: Ready for container use

The script handles the common container scenario where the database needs to be initialized on first run, then starts the application with Gunicorn.
```

We are now ready to containerise this application.

---

**Task-04**

With everything set, lets ask Kiro to containerise this application with the following prompt:

```
Create a Dockerfile to containerise this application. Ensure the Dockerfile uses an optimized, small base image.
```

This should create a Dockerfile, and generate a report. This is what was output when I asked Kiro CLI to do this:

```
> Created an optimized Dockerfile with:

- **python:3.12-slim**: Minimal Python base image (~45MB vs ~380MB for full image)
- **Multi-stage uv installation**: Copies uv binary from official image
- **Dependency caching**: Copies dependency files first for better layer caching
- **Frozen dependencies**: Uses --frozen --no-dev for production builds
- **Non-root user**: Runs as appuser for security
- **Port exposure**: Exposes Gunicorn's port 8000
- **Entrypoint**: Uses the existing entrypoint script

The image will be significantly smaller and more secure than using a full Python image.
```

This looks good. One thing that we can do is to use Kiro CLI to help us explain our Dockerfile. This can be helpful if you are new to writing Dockerfiles or want to know what a particular configuration does. We can ask questions like:

```
Will this package up the local database?
```

Kiro CLI will review and then provide you with an answer - in my specific example, it realized that it should have provided a .dockerignore file to exclude some of the artifacts in my project.

```
> Yes, the current Dockerfile will package the local survey.db file since the COPY . . command copies all files from the current directory.

For production containers, you typically want to exclude the local database. I'll create a .dockerignore file to prevent this:
I'll create the following file: /{project directry}/kiro-cli/workshop/kiro-cli-workshop-customer-survey-app/.dockerignore (using tool: write)
Purpose: Create .dockerignore to exclude local database and other files

+     1: survey.db
+     2: .venv/
+     3: .git/
+     4: .coverage
+     5: htmlcov/
+     6: __pycache__/
+     7: *.pyc
+     8: .env
+     9: .pytest_cache/
+    10: tests/

Creating: /{project directry}/kiro-cli/workshop/kiro-cli-workshop-customer-survey-app/.dockerignore
 - Completed in 0.2s

> Now the container will:
- **Exclude** the local survey.db file
- **Create** a fresh database on first run via the entrypoint script
- **Exclude** other development files like .venv, tests, and cache directories
```

You can also ask Kiro CLI to help you optimize the Dockerfile.

```
how can this Dockerfile be improved
```

Kiro CLI will now suggest a number of enhancements. After re-engineering my Dockerfile, it provided me with a summary of the changes it made.

```
> Key improvements made:

1. Multi-stage build: Separates dependency installation from runtime, reducing final image size
2. Virtual environment copying: Only copies the .venv from builder stage, excluding uv binary
3. Combined RUN commands: Reduces layers and image size
4. Explicit permissions: Ensures entrypoint script is executable
5. Health check: Monitors application health (requires a /health endpoint)
6. PATH optimization: Uses virtual environment directly
```

I think we are done for now, but bear this in mind as you use Kiro CLI to create your own application build artifacts.

Finally, we can ask it how we can test this with a simple prompt:

```
how can I test this Dockerfile?
```

Kiro CLI should now provide you with some commands which you can run in a new terminal to test. Review the commands and make sure that:

* you can build your container locally
* you are able to run your container locally

> **Stuck?** If you run into issues in this section, then use Kiro CLI to help you troubleshoot. Typical problems you might run into include conflicting ports on your local machine, or perhaps errors with the Dockerfile which cause issues during the build process. When troubleshooting with Kiro, remember to provide as much context as possible. Provide the stack trace or error message, together with details about where if fails and other information that you notice.

In a few minutes I have built a local container image, and then run it in my local container runtime (I am using [Finch](https://github.com/runfinch/finch) rather than Docker) - here is the output I get.

```
finch run -p 8000:8000 survey-app
Initializing database...
[2025-12-11 13:16:10 +0000] [1] [INFO] Starting gunicorn 23.0.0
[2025-12-11 13:16:10 +0000] [1] [INFO] Listening at: http://0.0.0.0:8000 (1)
[2025-12-11 13:16:10 +0000] [1] [INFO] Using worker: sync
[2025-12-11 13:16:10 +0000] [9] [INFO] Booting worker with pid: 9
[2025-12-11 13:16:10 +0000] [10] [INFO] Booting worker with pid: 10

finch ps -a

CONTAINER ID    IMAGE                                    COMMAND                   CREATED          STATUS                       PORTS                     NAMES
6710f3ac13ab    docker.io/library/survey-app:latest      "./entrypoint.sh"         2 minutes ago    Exited (0) 29 seconds ago    0.0.0.0:8000->8000/tcp    survey-app-6710f

```

I was able to open up a browser and confirm that I could register and then use this application.

---

## Automating tasks with scripts

We now have a working application packaged in a container. Kiro CLI has generated a Dockerfile and have been able to test it. Whilst this is great, there are lots of moving parts so lets show how we can create a script to help automate this.

**Task-05**

From the "[customer-survey]>" prompt in Kiro, use the following prompt. You will probably need to use "/editor" to add this detailed prompt.

```
Create a container build script that automates the task of building and tagging the container image build. It should do the following:

- Pass in as an argument to this script the tagged release version (for example, 1.0.0, 1.0.1, 1.1.0, etc)
- Do not tag with :latest
- Ensure SEMVER is used for correct version numbering
- Build multi-architecture container images (x86 and arm64)
- Define which container runtime tool is being used (that supports the Docker command options) e.g Docker, Finch, Podman, etc
- Do NOT use Docker specific arguments (like buildx)
- Provide an option to list existing container images built as well as ALL available container images
- Provide a help option to display all the possible options
- Support an optional argument to push the container to a container image repository
- Delete a local container image, providing the specific version
```

Follow Kiro as it prompts you for permission to create the script and make it executable. After a few minutes I have a new script called "build-container.sh" which is ready to go. The script you get will be different from my output, but it should still have similar characteristics. From a new terminal window, try your script out. 

```
./build-container.sh --help
Usage: ./build-container.sh [OPTIONS] VERSION

Build and tag container images with SEMVER versioning.

ARGUMENTS:
  VERSION           SEMVER version (e.g., 1.0.0, 1.2.3)

OPTIONS:
  -p, --push        Push image to registry after build
  -l, --list        List existing local images
  --list-all        List all local container images
  -d, --delete      Delete image by version
  -h, --help        Show this help message

ENVIRONMENT VARIABLES:
  CONTAINER_RUNTIME Container runtime (default: docker)
  REGISTRY         Container registry URL (optional)

EXAMPLES:
  ./build-container.sh 1.0.0                    # Build version 1.0.0
  ./build-container.sh --push 1.0.1            # Build and push version 1.0.1
  ./build-container.sh --list                  # List existing images
  ./build-container.sh --delete 1.0.0          # Delete version 1.0.0
  REGISTRY=myregistry.com ./build-container.sh --push 1.2.0

```

and I can use the "--list-all" command to quickly list my local images

```
 ./build-container.sh --list-all
All local container images:
REPOSITORY                                                    TAG       IMAGE ID        CREATED              PLATFORM       SIZE       BLOB SIZE
survey-app                                                    0.0.1     297078130849    About an hour ago    linux/arm64    189MB      50.69MB
704533066374.dkr.ecr.eu-north-1.amazonaws.com/fact-checker    1.0.1     8d06669222c2    8 days ago           linux/arm64    284.1MB    98.75MB
fact-checker                                                  local     81dc1220b3f4    9 days ago           linux/arm64    250MB      92.82MB
fact-checker                                                  1.0.2     8d06669222c2    9 days ago           linux/arm64    284.1MB    98.75MB
```

We will now test this by using this to build our container image. 

From this same terminal window, use the script that Kiro CLI created for you and run it to do the following:

* build the container image, tagging with a version 1.0.1
* list the container image
* delete the container image with the same version number 1.0.1

We can also add or change our script after it has been created. Lets add the ability to run a container image from this script. Switch back to the Kiro CLI session, and at the "[customer-survey]>" prompt, type the following:

```
add a new option in the script to run a container image by specifying the version number
```

Follow Kiro CLI as it makes changes, and then switch to the other terminal and try the following:

* run the --help command to list all the options to make sure this new feature has been added
* build a container image and then run it

Automating developer tasks with Kiro CLI becomes a breeze, and you can easily and quick add new features. 

---

## Automating with GitHub Actions

In this final task, we are going to see how Kiro CLI can help you automate the building of this container image as you add new features to your code. We are going to use GitHub for this, so make sure you have installed the required tools mentioned at the beginning of this lab.

**Task-06**

The first thing we need to do is create a new GitHub repo. I am going to use the command line, but if you prefer to use the GitHub web interface you can do that.

> **Note!** Before using gh you need to use "gh auth login" to authenticate against your GitHub account

Exit your Kiro CLI session, and from the terminal type in the following:

```
gh repo create kiro-customer-survey --public --description "Kiro Customer Survey workshop"
```

Which should generate output like this:

```
gh repo create kiro-customer-survey --public --description "Kiro Customer Survey workshop"
✓ Created repository 094459/kiro-customer-survey on github.com
  https://github.com/094459/kiro-customer-survey
```

If you originally clones this repo from then we need to change this. To make sure, run the following command:

```
git remote -v
```

It should display:

```
origin	https://github.com/094459/kiro-cli-workshop-customer-survey-app.git (fetch)
origin	https://github.com/094459/kiro-cli-workshop-customer-survey-app.git (push)
```

If it displays nothing, then skip then next command.

```
git remote remove origin
```

We now need to reset the git origin using the following command, replacing {repo} with the name of your new repo 

```
git remote add origin {repo}
```

using the example above, mine was "https://github.com/094459/kiro-customer-survey" so the command I would use would be "git remote add origin https://github.com/094459/kiro-customer-survey"

> **Help!** If you get the error "error: remote origin already exists" then you did not run the remove origin command.


We can now push our repo to GitHub using the usual commands:

```
git add .
git commit -m "kiro cli workshop"
git push -u origin main
```

If you switch back to the GitHub web application, you should now see your project appear. Whilst you are there, take a look at the Actions menu - you should see that it is currently empty with no actions configured. Lets change that.

We are now ready to get Kiro CLI to help us create some GitHub actions, so start Kiro CLI using the following:

```
kiro-cli --agent customer-survey
```

And once started, using "/editor" submit the following prompt:

```
- Create a GitHub action that will build/update a container image with new commits.
- Use the commit hash when tagging the container
- NEVER tag with :latest
- Build multi-architecture container images (x86 and arm64)
- Use GitHub's Container Registry
- Display a link to the container image 
```

Kiro will start working on building the GitHub Actions, and you should see output similar to the following. Kiro will prompt you for permissions as it goes along.

```
> I need to create the .github/workflows directory first:
I will run the following command: mkdir -p .github/workflows (using tool: shell)
Purpose: Create GitHub workflows directory

Allow this action? Use 't' to trust (always allow) this tool for the session. [y/n/t]:
```

After a few moments, you should see a status update. Here is the outout Kiro CLI generated for me:

```
> Done! The GitHub Actions workflow is now configured to use GitHub Container Registry (ghcr.io). Key changes:

- Uses ghcr.io registry instead of Docker Hub
- Authenticates with GITHUB_TOKEN (no additional secrets needed)
- Requires packages: write permission
- Tags images as ghcr.io/owner/repo:commit-hash
- Still builds for both x86 and arm64 architectures
- Never uses :latest tag as requested
- Displays link to the container image
```

The next step is to now push this code to GitHub, so exit Kiro CLI and from the terminal push these updates using the following:

```
git add .
git commit -m "Added GitHub actions"
git push
```

As this is a simple update it should be quick. Go back to the web interface of GitHub and from the project repo, check the Actions menu. This is what I see, you should see something similar.

![github actions overview](/images/github-actions-overview.png)

When you click on the build job, it will display the breakdown of the tasks and provide logs to help you troubleshoot any issues. If I want to find the link for the container image that has been published on GitHub's container repository, I click on it and you see see that a link has been generated (which you can see for yourself, [here](https://github.com/094459/kiro-customer-survey/pkgs/container/kiro-customer-survey))

![github actions run](/images/github-actions-run.png)


To confirm this is working, from the terminal I can now run the following command:

```
finch run -p 8000:8000 ghcr.io/094459/kiro-customer-survey:674f4bee68246f4081d2277998406b3a6ffd4d39
```

And in a few seconds, I now have the application up and running on my local machine. 

Try this yourself:

* review the output from your GitHub Actions and make sure it looks good
* grab the link of the container image that was built
* run in in your local machine, using the following command (replacing container-image-link with the output from your setup)

```
docker run -p 8000:8000 {container-image-link}
```

Once you have checked its running by opening up a browser and navigating to http://localhost:8000, you can exit by using CTRL + C

---

**Task-07**

With our automation in place, when we make changes to our application, our container image will automatically be updated. We can see this in action by making a quick addition to the application.

From the terminal you have just been working in, start Kiro CLI with the following:

```
kiro-cli --agent customer-survey
```

At the "[customer-survey]>" prompt, use the following to add a new simple feature to our amazing customer survey application.

```
Add two new features to this application: 

1 - Add a "Help" page that displays how to use this application from an end user perspective
2 - Add a "Contact Us" page that displays a simple telephone number and email address to contact in the event of problems
```

Follow Kiro as it creates these new files in the project, and when this has finished push these changes to the GitHub repo.

```
git add .
git commit -m "new features"
git push
```

Switch back to GitHub and watch as the new update triggers the rebuild of the container image. Once completed, grab the new container image link and run this on your local machine. This is the updated container image when I ran this:

```
finch run -p 8000:8000 ghcr.io/094459/kiro-customer-survey:44c585e9b2fbfecdb56d9471908eba9a258289dd
```

When I opened up my local browser, I can now see that I have the new features in the application.

![new features up and running](/images/updated-app.png)

---

### Automating Kiro CLI configuration

Before we finish this section, I want to cover one last aspect of automation - how you automate your Kiro CLI setup. As you have been running through these labs, you will have been creating custom agents (a json configuration file), creating steering documents (markdown docs which we have copied from a GitHub repo). For the purpose of the lab, it is important that we do this so you can walk through the process. But as you start to move beyond the basics, you will want to automate your setup.

In the [Advanced Topics](/workshop/04-advanced-topics.md) you looked at [Context Hooks](https://kiro.dev/docs/cli/hooks/?trk=71546b8e-c969-4ead-aa9f-9cd06f6d8610&sc_channel=el). Go back and review that section again before proceeding.

Context Hooks provide a powerful automation primitive that you should explore as you look to take your automation tasks to the next level. For example, one of the things I have setup when I create custom agents is to automatically pull down some steering documents that I always use:

```
{
  "name": "my-dev",
  "description": "Python custom agent",
  "prompt": "You are an experienced Python 3 software developer.",
  "mcpServers": {},
  "tools": [
    "*"
  ],
  "toolAliases": {},
  "allowedTools": [],
  "resources": [
    "file://AGENTS.md",
    "file://.kiro/steering/**/*.md"
  ],
  "hooks": {"agentSpawn": [
      {
        "command": "cd .kiro/steering && wget -N https://raw.githubusercontent.com/094459/aqd-cli-workshop/refs/heads/main/resources/python-dev.md && wget -N https://raw.githubusercontent.com/094459/aqd-cli-workshop/refs/heads/main/resources/ip.md"
      }
    ]
  },
  "toolsSettings": {},
  "useLegacyMcpJson": false,
  "model": null
}
```

When I start Kiro CLI, this triggers the agentSpawn event which then runs these commands to copy some steering documents down to my local global steering files. I will always have the latest version as long as I keep the version in source control up to date.

Context hooks work at other events too - before and after tools are used for example. 

* agentSpawn - when Kiro CLI starts
* UserPromptSubmit - after every prompt that you enter in Kiro CLI
* Stop - at the end of processing a given prompt
* PreToolUse - before Kiro CLI uses a tool (which you define)
* PostToolUse - after Kiro CLI uses a tool (again, which you have defined)

Explore the [Advanced Topics](/workshop/04-advanced-topics.md) section again on how to use these.

---

## Summary

In this lab we have seen how we can use Kiro CLI capabilities to help with automation. We just scratched the surface of what is possible, and automation tasks like the ones in the labs you completed are just simple examples.

You can [proceed back to the README](/README.md).
