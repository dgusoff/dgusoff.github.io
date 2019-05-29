---
layout: post
title: PowerShell profile breaks local debugging for Azure Functions 
---

I came across a very strange issue this week trying to locally debug TypeScript Azure Functions in Visual Studio Code.  The F5 debugger invocation was throwing a failure with the message "No such file or directory". 

![screenshot](/images/vscode-funcs/console.png "Console Screen shot")

At first I thought it was an environmental issue, so I set up a clean new dev environment in an Azure VM, installed the latest versions of everything, and got the same error. What was even more frustrating was that the Microsoft documentation talks about how easy all this was.

Note that this terminal is opened when you initiate a debugging session, and it doesn't offer a prompt to cd into the correct directory.

The first clue to the nature of the problem was in the error message itself. In the screen shot, the console is looking for a package.json in the "C:\code" directory. This is the directory I set with a CD statement in my PowerShell profile.

Here's the contents of my PowerShell profile:

![screenshot](/images/vscode-funcs/profile.png "Console Screen shot")

So it seems to me that what's happening is, the F5 opens the console at the correct directory, then the profile loads, including the cd command, and finally the npm commands execute after the directory change happens and the context is all messed up, resulting in a broken build.

I was able to get my Azure Function to debug locally by removing that CD command from my profile and restarting VS Code. This is supremely annoying, because I'm used to my PowerShell sessions starting in the place where I'm most likely to be using them.

## The fix

This is not ideal for me, bacause I do love my PowerShell, but the solution I came up with was to set the default shell at the project level to use Git Bash. You could probably also get away with setting cmd.exe.

Each VS Code project has a local file called settings.json. If you add the following line to settings.json it'll use Git Bash as the default shell, and the PowerShell profile scripts will be irrelevant.

```json
"terminal.integrated.shell.windows": "C:\\Program Files\\Git\\bin\\bash.exe"
```

Of course this assumes you have Git Bash installed at the specified location.

Once this settings file is updated, a restart of VS Code allows the trouble-free debugging experience all the Microsoft documentation promises.

I'd love to know if anyone else has experienced this and what they did to rectify it.
