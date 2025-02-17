# sos-test_system

An SOS-System for testing SOS-Toolkit.

Simple Flask frontend and a mock FastAPI backend to output text to a web client.

The purpose is to demonstrate the internal workings of SOS-Toolkit.


# Requirements
- [SOS-Toolkit](https://github.com/vtp-one/sos-toolkit)

Follow the instructions to get a working SOS-Toolkit virtual environment. 

Installation instructions assume you already have an activated virtual environment. 

All Commands in the environment are prefaced with: `(sos)$`


# Current Release Target
```
v0.0.1
```

# Installation
These steps will pull, install, and run SOS-Test_System. 

You should start from your SOS-Toolkit root directory: `/sos` or it's equivalent in your environment

:warning: WARNING :warning:
```
A SYSTEM CAN RUN ANY TERMINAL COMMAND AND ANY ARBITRARY PYTHON CODE INCLUDING DYNAMICALLY GENERATED CODE
DO NOT INTERACT WITH SYSTEMS THAT YOU DO NOT KNOW, TRUST, OR HAVE THE CAPABILITY TO MANUALLY VERIFY THEIR OPERATION
```


### 1. Pull a Remote System

This will use SOS-Toolkit to clone this git repository using the global installation of git via [gitpython](https://github.com/gitpython-developers/GitPython). 
The global installed version of git can utilize your pre-existing git settings including saved passwords and accounts. 
While SOS-Toolkit should not have access to this information, that is dependent on the security of the gitpython package. 
```
(sos)$ sos-toolkit pull https://github.com/vtp-one/sos-test_system.git --remote-branch v0.0.1
(sos)$ cd sos-test_system
```
If you are only intending to use a system, not develop it, you should always include a remote-branch which is the Current Release Target for the system.
Without this switch added to the command, SOS-Toolkit will try to pull the main branch.
While this should be the same as the Current Release Target, that is dependent on how the developers are maintaining the repository.

### 2. Setup the System

This command will generate an SOSContext object for the system and save it to an `sos-context.yaml` file in the current directory.

```
(sos)$ sos-toolkit setup
```

The SOSContext is generated by working with multiple files: 
 - `sos-system.yaml` - the current system information file located in the pulled repository
 - `sos-root.yaml` - SOS-Toolkit root information located in the SOS-Toolkit source tree
 - `sos-local.yaml` - system local information file located in the pulled repository directory
 - `sos-user.yaml` - user local information file - currently not implemented 
 - `sos-service.yaml` - SOS-Toolkit services that provide platform dependent functionality located in the SOS-Toolkit source tree

Together the generated SOSContext represents the current state of the system in your environment. 
It can be edited manually or by using SOS-Toolkit commands. 
Manually editing can lead to problems, so the preferred method is using actions defined by the system, or using SOS-Toolkit commands.

As part of the setup process, a system is able to include additional actions that are run when the SOS-Toolkit setup command is called. 
These could include downloading containers, data files, building directory structures, and any other command line or python code. 

For this system this includes using the SOS-Toolkit local_data service to create a local_data directory which will be stored in the internal directory of the system.

Before running any system, you can open the `sos-system.yaml` file in a text editor which will allow you to see what the actions do. 

More information about the format of this file and the way it is used can be found: [SOS-Toolkit](https://github.com/vtp-one/sos-toolkit)

A simple explanation is that an SOSContext defines actions, and these actions are run when you use SOS-Toolkit commands.
```
sos-toolkit setup 
  => generate: SOSContext object
  => run: action.sos_setup 
  => run: each object defined in the sos_setup action
```

If you attempt to call the setup command when there is already an `sos-context.yaml` file in the output location, it will cause an error.
By default, the command will not overwrite a pre-existing context.

In order to do that call the command using the --overwrite switch.
```
(sos)$ sos-toolkit setup --overwrite
```
This will delete the previous file, so be sure to make a copy of it if you want to keep it.


### 3. Install the System

After generating the `sos-context.yaml` file for the system, this command will install the system using that SOSContext. 
```
(sos)$ sos-toolkit install
```
Running the install command should result in a system that does not require any further steps for complete operation. 
That means no other downloading, building, baking, or any other tools required.
This is a goal for SOS-Toolkit - to compress the steps needed to install digital intelligence systems into a three step process for an end-user: 
 - setup 
 - install 
 - run

That is the case for this Test System, but still not entirely functional for more complex systems.

For this system, this command preforms two steps. 

First it clones the sub-repositories into the system's internal data directory: 
 - [sos-test_system-builder](https://github.com/vtp-one/sos-test_system-builder)
 - [sos-test_system-backend](https://github.com/vtp-one/sos-test_system-backend)
 - [sos-test_system-builder](https://github.com/vtp-one/sos-test_system-frontend)

Second, it uses the information from the builder repository along with the local information provided by the context to use docker buildx to build two containers: one for the backend and one for the frontend.

In the future this will instead only need to pull containers instead of pulling sub-repositories and building containers locally, but they currently aren't prebuilt for testing purposes.


### 4. Start the System

This command will utilize docker-compose to start containers that use dynamic variables pulled from the SOSContext object including local directories and dynamic configuration options.

These variables and settings are generated during the initial setup, modified during the installation, and can be changed using system actions or SOS-Toolkit commands.
```
(sos)$ sos-toolkit up
```

For this system, starting will output to the terminal a status object for the system: 
```
{'terminal.print_object': 'SERVER RUNNING ON: http://X.X.X.X:XXXXX'}
```
You can open that link in a web browser, and see the system in action.
```
FROM BACKEND: {'message': 'Hello World!', 'debug_name': 'Test System', 'time': XXXXXXXXXX.XXXXXX}
```
Nothing fancy, just a simple test system showing a frontend communicating with a backend. 

When the frontend Flask server receives a request, it makes a request to the backend FastAPI server, and prints the result to the browser.

If you tried to run this command before running install, it will error. 
You need to install a system before you can run it.


### 5. Stop the System

This command will use docker-compose to stop and remove the running containers.
```
(sos)$ sos-toolkit down
```


# System Configuration
This Test System includes a simple example of a configuration action.

If you already ran the system once, you can notice that the output to the browser included a "debug_name" value. 
The included configuration action will allow us to change that.
```
(sos)$ sos-toolkit config
Input Debug System Name:
foobar
```
Enter any name you want, press enter to submit it, and start/restart the system:

```
(sos)$ sos-toolkit up
```
or
```
(sos)$ sos-toolkit restart
```

If you check the output of the server:
```
FROM BACKEND: {'message': 'Hello World!', 'debug_name': 'foobar', 'time': XXXXXXXXXX.XXXXXX}
```
You used an action to modify a value in the SOSContext.
That value was injected into the docker compose environment, and was used by the backend server.


Since we are playing with configuration, here is a direct method to accomplish the same change.
In the SOSContext, the debug_name value is stored like this:
```
context.namespace.debug_name
```
Instead of using the pre-built action, we can directly modify the value using a command:
```
(sos)$ sos-toolkit context --target=namespace.debug_name --value=foobar_from_command
```
If we now restart the system:
```
(sos)$ sos-toolkit restart
```
Then check the frontend:
```
FROM BACKEND: {'message': 'Hello World!', 'debug_name': 'foobar_from_command', 'time': XXXXXXXXXX.XXXXXX}
```
The new value worked.

Remember finally to shut-down the system when done.
```
(sos)$ sos-toolkit down
```


# System Development
Running a system is nice, but there is of course a long development process required before even a simple system like this works.

SOS-Toolkit is designed for development first, and while it still needs quite a few tools to meet that purpose; it already has some useful functionality.

For this demonstration we are going to start from a clean state.

This is a dangerous command. It will be deleting files, and there are currently no safeties or guardrails for that. 
If you've modified the system or if something is misconfigured, this could delete anything. 
It's intended primarily for development purposes and shouldn't be used outside of a system where you know exactly what it's going to do.
There is also has a glitch where it isn't able to delete files created by a container that is running as the root user.
```
(sos)$ sos-toolkit clean
```
For our Test System here is what it does:
 - shutdown any running containers
 - delete the system's docker images: backend, frontend
 - delete the internal repositories: internal/builder, internal/frontend, internal/backend
 - delete the local_data for the system: internal/local_data
 - delete the `sos-context.yaml` file

These actions are defined in the `sos-system.yaml` file. 
A developer needs to define these actions for them to work.

The targets for the actions are pulled from the SOSContext, and that is why the command is dangerous. 
If you've changed the values these actions use, it will run the actions with whatever values it finds.
If those are set incorrectly, that could mean deleting anything. 
Instead of targeting the internal/builder directory, it could attempt to instead delete your root directory. 
If it is a concern, a safer option would be to manually delete the system's directory, pull the system again, and start from a clean state that way.


Assuming now that you are in a clean Test System repository, the first thing we want to do is generate a new SOSContext.
```
(sos)$ sos-toolkit setup
```

Now instead of using the install command, we are going to use the dev command to setup our development environment.
```
(sos)$ sos-toolkit dev
```
This command is intended to setup a development environment. 

This could include pulling other git repositories or setting whatever configuration options are needed.
For this Test System, the main difference is that it sets the SOSContext profile to dev instead of default.
This profile variable is saved in the SOSContext under the `meta` object.
The profile setting is used as a flag to modify how the system works, such as selecting the dev profile from the docker-compose file.
Using the dev profile enables us to use the build command.
```
(sos)$ sos-toolkit build
```
Running this command will force a build of the docker containers using docker buildx bake. 
You can look at internal/builder/bakefile.hcl to see what that does.

This command is used in dev mode instead of install. 
After running this command, the system should be fully functional.

We can now start the system in the same way as before
```
(sos)$ sos-toolkit up
```
The difference here is that the command selects the dev profile instead of the default profile from the docker-compose.yaml file. 
This profile maps the internal folders for the backend and frontend into the running containers as volumes. 
This will enable us to edit them and see the results.
You can look at the docker-compose.yaml file to see how that is done.

If you now go to the frontend web address in a browser, it should appear the same as the install version. 

Let's try some actual development!

We first need to stop the system because hot-reloading of the server components doesn't work yet due to docker limitations.
```
(sos)$ sos-toolkit down
```
Now we can go and edit something. 
We are going to change the message the backend replies to the frontend. 
To do that we want to edit the main.py file located in the backend directory

```
(sos)$ nano internal/backend/server/main.py

from fastapi import FastAPI
from fastapi.responses import FileResponse

import time
import os

DEBUG_NAME = os.environ.get("DEBUG_NAME", "DEBUG_SYSTEM")

app = FastAPI(
    title="test_system-backend",
    version="0.0.0")
favicon_path = 'favicon.ico'

@app.get("/")
async def root():
    return {"message":"Hello World!",
            "debug_name":DEBUG_NAME,
            "time":time.time()}

@app.get('/favicon.ico', include_in_schema=False)
async def favicon():
    return FileResponse(favicon_path)
```

We want to add something to it's response from the root:
```
@app.get("/")
async def root():
    return {"message":"Hello World!",
            "debug_name":DEBUG_NAME,
            "time":time.time()}
```

Let's add an additional object to that dictionary:
```
@app.get("/")
async def root():
    return {"message":"Hello World!",
            "debug_name":DEBUG_NAME,
            "development_name":"foobar",
            "time":time.time()}
```
Save the file, start the system, and check the output.

```
(sos)$ sos-toolkit start
```
```
FROM BACKEND: {'message': 'Hello World!', 'debug_name': 'Test System', 'development_name': 'foobar', 'time': XXXXXXXXXX.XXXXXX}
```
There is our change.

We now want to commit our changes, so first we can call the status command to see what has changed.
```
(sos)$ sos-toolkit status
```
This command will output the git status of the different system components. 
We should see that the internal/backend repo is marked as dirty since we edited a file inside it.

The next step would be to commit our changes. 
Currently SOS-Toolkit does not have the tools required to switch to different remote repository, so the commit action won't work for you.
If you had forked the Test System repo or if you owned the repo for the system, you would be able to use the commit command:
```
(sos)$ sos-toolkit commit
```
This command would commit the changes while asking you for a commit message. 

Since you don't have permissions, it will fail; but as long as your local repo is in a dirty state, it will not be overwritten. 
It is still possible to delete a directory and any changes made either manually or using the `clean` command.

In the event of an upstream update, it currently requires manual resolution of conflicts, but future tools intend to work around that.


# Forking
Forking the repository yourself to test changes is currently not as simple as it will be. 
The problem is due to the separation into multiple repositories. 

You need to fork all the sub-repositories the system uses: 
 - sos-test_system-builder
 - sos-test_system-backend
 - sos-test_system-frontend
 
This is due to the system currently using a single root git target url for the repositories. 

Future versions will utilize individual targets for each component, but this early version does not support that. 
You need to keep the names of the sub-repositories the same: sos-test_system-backend, sos-test_system-frontend, and sos-test_system-backend. 

If you do fork all three repositories, you can modify the system to target your repositories using the `sos-local.yaml` file. 
These changes need to be made before running the setup and dev commands. 

The values you need to change are: 
 - namespace.git_root
 - namespace.branch_target
 - namespace.org_target

Now you need to create an `sos-local.yaml` file in the system directory:
```
[ sos-test_system/sos-local.yaml ]

namespace:
  git_root: <YOUR GIT ROOT>
  branch_target: <YOUR BRANCH TARGET>
  org_target: <YOUR ORG TARGET>

action:
  sos_status:
    root:
      disabled: True

  sos_commit:
    root:
      disabled: True
```
When you run the setup command, SOS-Toolkit will load the `sos-local.yaml` file and merge it with the `sos-system.yaml` to generate the `sos-context.yaml` file. For more information about how that works, you can read the SOS-Toolkit documentation.

The namespace values are combined by the system actions to create the target url for each action:
```
git pull https://<YOUR GIT ROOT>/<YOUR ORG TARGET>/sos-test_system-builder.git --branch <YOUR BRANCH TARGET>
git pull https://<YOUR GIT ROOT>/<YOUR ORG TARGET>/sos-test_system-frontend.git --branch <YOUR BRANCH TARGET>
git pull https://<YOUR GIT ROOT>/<YOUR ORG TARGET>/sos-test_system-backend.git --branch <YOUR BRANCH TARGET>
```

The additional values provided for the action objects `sos_status.root` and `sos_commit.root` will disable running the status and commit actions for the system's root repository.
This needs to be done since you didn't fork the sos-test_system repository and if the actions try to run, they will fail since the repository doesn't exist from your new git target.

Using the SOS-Toolkit setup command will now use your provided values from the `sos-local.yaml` file and all subsequent actions will target the new locations. 

Convoluted, but functional. 
This will be improved in latter SOS-Toolkit versions.


# Capabilities
Command actions are not limited to git. 

By defining SOS-Toolkit actions you can include any tool you want and define the usage however you need.
This can automate complex steps like creating releases, bug-fixes, pull requests, pushing docker images, building resources, downloading data, generating runtime configurations, and really anything you want. 
SOS-Toolkit includes a plugin system that allows you to create custom python functions that can be automatically included in your system and used as tools by your actions.

By using the `sos-local.yaml` file and the other SOS-Toolkit configuration methods, you can create local environments that allow actions to utilize dynamic location targets, accounts, passwords, secrets, and any information.


# More Information
This system is currently used as a test system for SOS-Toolkit, not as a full demonstration of it's features.

To see more of what SOS-Toolkit can do, check out some of the other systems or the SOS-Toolkit documentation:

 - [SOS-Systems](https://github.com/vtp-one/sos-systems)
 - [SOS-Toolkit](https://github.com/vtp-one/sos-toolkit)


# License
LICENSE WILL CHANGE

CURRENT LICENSE: VTP-LICENSE-v0.0.1

FOR APPROVED USE: POLYFORM - STRICT
