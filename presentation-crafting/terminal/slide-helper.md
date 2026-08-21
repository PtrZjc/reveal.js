### 1. Title Slide: Effective Terminal

Hi everyone! Welcome to today's Lunch and Learn session.

I am really exited to be here with you as I will present you the topic which is very close to my heart - the command line, which we often recall to as terminal.

I have some conceptual overviews on the slides but we'll also quite often jump to the terminal.
We have 60 minutes and plenty of content, so let's get right into it.

### 4. Session Overview

Before we start, a short note on agends. I've promised there will be something for everyone. We'll start with the basics which wi'll be a repeat for many of you, but I ask for some patience of advances users when we speak about the foundations of unix command line, as there will be some cool things later on, I promise. I will show you some shiny tools as well as my some of my work habits.

### 3. Why the Terminal?

I would like to start with sharing a bit of personal story here.

My journey with terminal-first workflows started back in 2019 after seeing a live terminal demo on Devox conference by Dawid Pokusa. 

Seeing someone manipulate systems, transform data, and navigate effectively without single mouse click looked like a magic. I was amazed by what I witnessed. I came home and started tinkering which lasts till this day.

That brings me to term I love here
Working in terminal is frictionless. There is no friction resulting from necessity of clicking though interfaces and windows. It allows fast, keyboard-driven daily workflow.
Additionally moving away from UI-based tooling to CLI workflows reduces context switching.

That brings me to the last point.
Command-line literacy is one of the highest-ROI technical skills an IT engineer can get. Whether you are debugging a container, executing tasks on remote servers via SSH, or automating repetitive tasks, the terminal is the universal interface. It opens myriad of great new tools which are just waiting to be used one command away.


### 6. Knowledge & Operational Skill

Engineering mastery relies on two distinct pillars:

The knowledge is domain, technology, frameworks etc.
The skills is how proficiently we operate the knowledge. 

We'll focus entirely on that latter part.
Let's try to be like this swordsmith, hone our skills and craft a gear we'll use later on.

### 7. The Three Layers of the Terminal

Before we start using it, let's stop for a moment clarifying the terms as
People often confuse terminal terminology. There are three distinct layers:

1. **Terminal Emulator:** The graphical window application that captures keystrokes, interacts with  OS, and renders characters
2. **Shell Interpreter:** The command language engine running inside that emulator, responsible for parsing commands, expanding variables, and executing processes (
3. **Framework:** Lies on the top of shell and makes it configuration easier, allows for easy extending new functionalities. The frameworks provides stuff like autocompletion, git integration, syntax highlighting, and themes 

### 8. Example Stack

Once you understand the three-tier separation, the immediate question is how to compose this stack on your machine.

Here are shows the default choices on on different systems.
Rather than sticking with factory OS defaults, I recommend to make a research and try something that will suit you.
For terminal emulator you should look for modern GPU-accelerated emulators, as legacy terminal often rely on CPU software rendering.
What I can recommend is if you are using macos, iTerm2 emulator is good starting point.


Next thing, the shell. While Bash remains the default for headless production servers, Zsh provides superior interactive capabilities rich tab completion, advanced globbing, and syntax highlighting and it's near fully compatible. 

On Windows in order to utilize what we show today, you must use WSL2 with linux like Ubuntu. Avoiding legacy pseudo-consoles and PowerShell quirks is critical for Unix tooling.
WSL2 provides an actual Linux kernel running in a lightweight VM, giving you full POSIX standard compliance, direct Linux file system performance, and exact parity with production environments.


I can't comment other frameworks than oh-my-zsh which I use from the very beginning, but here are the names of the more modern popular zsh frameworks that exist.

### 9. Part III: Core Text Processing & Piping

Let's dive into live stream manipulation. For these demos, we will use a fantasy dataset: `spells_catalog.tsv`.

### 10. Core File & Text Processing Tools

In the Unix ecosystem, the core philosophy is simple: write tools that do one thing and do it well. Because Unix treats almost everything as a simple stream of text, standard utilities process data line by line.
#### echo

#### cat
The first tool is cat. We use it by using filename as argument.

(Show cat data.tsv on screen)

It just prints to the terminal. Nothing fancy. No colors. Just plain text.
Since it’s hard to read, let me show you the file content using a different, installed externally... or Tidy Viewer.

(Show tv data.tsv on screen)

Much better. Now we actually have proper column alignment, alternating row colors, and a clean 
tabular layout so we can see what we're working with.
### nl
Next up is nl. While cat dumps raw text, nl passes the file through and numbers every line. It’s incredibly useful for quick indexing or referencing line numbers in output.nl
#### wc 
Moving on, we have wc, short for word count. While it can count words and bytes, its most popular flag by far is -l. Running wc -l instantly gives you the line count—making it the fastest way to answer, "How many records are in this file?"
```sh
wc spells_catalog.tsv
wc -l spells_catalog.tsv
```

#### head tail
Next, we have head and tail. When you're dealing with massive log files, you don't want to dump millions of lines at once. head shows you the top few lines to inspect headers, while tail gives you the end of the file. 

#### grep
Now, what if you need to find a needle in a haystack? That’s where grep comes in. grep searches through text line by line using regexes. Whether you’re looking for a specific error code, an IP address, or a username, grep filters out the noise and returns only the matching lines.

```sh
grep Fire spells_catalog.tsv
grep -r Fire
```
#### sed
And finally, we have sed, the stream editor. While grep only finds text, sed lets you modify it on the fly. You can search, replace and edit text on the fly. But since sed has terrible API (it's very old) i show you modern alternative `sd`
```sh
sd -p Fire Water spells_catalog.tsv
```

### 11. Unix Pipelines

Each of these tools is lightweight and specialized on its own, but the real magic happens when you start chaining them together with pipes.

We are going to chain these single-purpose tools together using the pipe operator (`|`). Each command receives the output of the preceding command on its standard input.

**DEMO: Multi-Stage Processing Pipeline**

We are going to count the types of all spells in the catalogue:

```bash
cat spells_catalog.tsv  
  | cut -f3  # 2 spaces to prevent history
| tail -n +2 
| sort 
| uniq -c 
| sort -r
```
### 12. Standard Streams & Redirection

Lets see at pipe. We have producer, then ....

Depending what we use at the end, the output is redirected.

if tool raises error, the output comes through different exit and the rest of pipe is bypassed. 
We will not go deeper into that now, but keep this in mind.

Every Unix process opens three standard I/O file descriptors:

* `0 (stdin)`: Standard Input (keyboard or incoming pipe).
* `1 (stdout)`: Standard Output (screen or outgoing pipe).
* `2 (stderr)`: Standard Error (diagnostic messages and error logs).

> Remember: The pipe operator (`|`) redirects **only stdout (Port 1)**. Error logs on `stderr` bypass standard pipes unless explicitly merged via `2>&1`.

There are also other redirections and let me show you (from last exercies)
```sh
> spell_types.txt
>> spell_types.txt
```

* `>`: Overwrite file with `stdout`.
* `>>`: Append `stdout` to file.
* `2>`: Redirect `stderr` to file.
* `2>&1`: Merge `stderr` into `stdout`.
* `> /dev/null 2>&1`: Completely silence execution output.

### 13. Modern Replacements for Classic Tools

Classic coreutils were built decades ago for constrained hardware. Modern alternatives written in Rust/Go offer multi-threading, sensible defaults (ignoring `.git` and `.gitignore` targets out of the box), nice api and some nice stuff like  syntax-highlighted outpu

| Legacy Tool | Modern Drop-in     | Key Advantages                                           |
| ----------- | ------------------ | -------------------------------------------------------- |
| `grep`      | **`rg` (ripgrep)** | Multi-threaded, respects `.gitignore`, $10\times$ faster |
| `find`      | **`fd`**           | Clean syntax, ignores hidden/ignored paths by default    |
| `cat`       | **`bat`**          | Syntax highlighting, line numbers, Git diff integration  |
| `sed`       | **`sd`**           | Cleaner regex syntax, simplified string replacement      |
| `ls`        | **`eza` / `lsd**`  | File icons, tree hierarchy views, inline Git statuses    |

**DEMO: Legacy vs Modern Replacements**

```bash
# Search files
grep -rn "Fireball" .
rg "Fireball"

# Find files
find . -name "*.tsv"
fd -e tsv

```

Takeout - look for modern alternatives. Some are obvious choice like the ones showed here, but others often have multiple alternatives to choose from.

---

### 14. Fast Command Lookups
Ok,  we already know some tools, we also know how to chain them. We also started discovering new tools. 
How we should know how to use them. Of course you can ask onyx, but there are faster alternatives.

one can make efficiently check how to use the particular tool? 

we can do either is shown on the left, digging into massive grimoure, or on the right, looking exactly at what we need.
Never break focus to search command syntax in a browser:

* `man <command>`: Comprehensive reference manual.
* `<command> --help` / `-h`: Fast flag and option lookup.
* `tldr <command>`: Community-maintained cheat sheet focused on practical usage patterns.
* `zsh-autosuggestions`: As-you-type completions matching history.

**DEMO: In-Terminal Help**

```bash
man fd     # Full manual
tldr tar    # Practical examples
rg --help   # Instant flag lookup

```
Show autosuggestions for fd, rg, sd
### 15. Navigation & Interactive Fuzzy Finding
Let's speak about navigation.
In the old days I've hated terminal navigation. 
I felt like this guy over there looking at all the muddy valleys down without knowing there are some portals that allow to just teleport in blink of eye.

Manual folder traversal (`cd ../../../`) slows you down. We replace it with two utilities:

* `z` / `zoxide`: Frecent directory jumping based on frequency and recency (`z grimoire`).
* `fzf`: General-purpose interactive fuzzy finder for files, history, git commits, and processes.

**DEMO: Fuzzy Navigation Helper**

```bash
# Jump directly with z
z spells

# Interactive directory hopper combining fd and fzf
alias goto='cd $(fd -t d | fzf)' #$() = Command Substitution
goto
```




* **History Search:** Use `Ctrl + R` (enhanced with `fzf`) to recall complex one-liners instantly instead of retyping.

### 16. Dotfile Architecture & Custom Helpers
We know to move now. We have some new shiny tools. We have some initial aliases that allow us to navigate efficiently. 
How to persist it? this is where the concept of dotfiles comes in.
Centralize the config and make it version-controlled in Git, and synchronized across environments.

PLAN: 
```sh
cat ~/.zshrc (autosuggestion points to dotfiles)
z dot
cat zsh/custom_oh-my-zsh/git.zsh
```

### 17. Batch Execution with xargs

`xargs` converts standard input into arguments for other commands, enabling batch operations and parallel execution without manual `for` loops.

**DEMO: Directory and File Batch Generation**

```bash
# 1. Generate directories from unique spell schools
cat spells_catalog.tsv | cut -f3 | tail -n +2 | sort | uniq | sort -r | xargs -I {} mkdir spell_{}

# 2. Populate each folder with matching school spells
cat spells_catalog.tsv | cut -f3 | tail -n +2 | sort | uniq | sort -r | xargs -I {} sh -c "cat spells_catalog.tsv | rg {} > spell_{}/spells.txt"

```

### 18. Data Processing

Now let's move to the most interesting part of today presentation - the data processing.

look at batch operations, argument construction, and structured data handling (JSON/YAML/CSV).
### 19. Structured Data Processing

Engineering workflows constantly interact with structured formats: **JSON, YAML, CSV, TSV**. Web formatters and heavy GUI apps often crash or choke on large production payloads and log exports. CLI processors handle these streams deterministically and fast.

**Slide Bullet Points**

* **Modern Formats:** Modern workloads interact heavily with structured formats: JSON, YAML, CSV, TSV.
* **GUI Limitations:** GUI applications and web viewers often struggle with raw large-scale data exports and API responses.
* **CLI Processors:** Dedicated CLI processors enable filtering, restructuring, and aggregating large-scale data efficiently.

---

### 20. CLI Data Transformation Utilities

The essential toolkit for handling structured formats:

* `jq`: Query, slice, filter, and transform complex JSON trees.
* `yq`: YAML/XML processor mirroring `jq` syntax.
* `gron`: Flattens JSON into discrete, line-by-line greppable assignments.
* `yj`: High-speed format conversion between YAML $\leftrightarrow$ JSON $\leftrightarrow$ TOML.
* `miller` (`mlr`): Relational data processor for CSV, TSV, and tabular JSON (runs SQL-like verbs directly on streams).

### 21. Live Data Transformation

Let's see these processors in action on real payloads.

I am going to show you how easily group the spells by type into a json. But without necromancy. We don't want dark arts in out newly grouped spells.

**DEMO: Transforming JSON and Log Streams**

```bash
# 1. Query nested objects with jq (Filter base_stat > 100)

cat spells_catalog.tsv 
| rg --invert-match Necromancy
| mlr --t2j cat
| jq

rura -c 'cat spells_catalog.tsv | rg --invert-match Necromancy | mlr --t2j cat | jq'

# '.[].type' -r
# 'group_by(.type)[] | length
# 'group_by(.type)[] | {(.[0].type):(map(.name))}' | jq -s
```

show gron / yj -jy

2 - 
[logs-link](https://sportradar.grafana.net/a/grafana-lokiexplore-app/explore/namespace/ld-qa-prediction-markets/logs?var-filters=namespace%7C%3D%7Cld-qa-prediction-markets&var-filters=app%7C%3D%7Cpm-server&patterns=%5B%5D&from=now-30d&to=now&timezone=browser&var-lineFormat=&var-ds=cen5g1miwlaf4d&var-fields=&var-levels=&var-metadata=&var-jsonFields=&var-all-fields=&var-patterns=&var-lineFilterV2=&var-lineFilters=caseInsensitive,0%7C__gfp__%3D%7CStarted%20PmApplication&urlColumns=%5B%5D&visualizationType=%22logs%22&displayedFields=%5B%22formattedMessage%22,%22message%22,%22exception%22%5D&userDisplayedFields=true&sortOrder=%22Descending%22&wrapLogMessage=true&prettifyLogMessage=true)

I could use positive lookahead
```bash
cat ~/Downloads/Logs-2026-08-16\ 23_01_57.txt 
 | rg --only-matching '\d+\.\d+ seconds' 
 | sd " seconds" "" 
 | youplot histogram -n 10 --ylabel seconds -t "Application starting times"
```


---

### 22. Unobvious Terminal Utilities

A quick rundown of specialized tools that solve distinct operational bottlenecks:

* `cal -3`: Displays previous, current, and next month side-by-side in monospace.
* `trans` (Translate Shell): Command-line translation engine directly inside stdout.
	  - `which en pol`
* `pngpaste` / `icat`: Paste images directly to disk from the system clipboard or render graphics in-terminal.
* `tesseract` (OCR): Extract text from image files on the CLI.
* `dust` / `diskbloom`: Visual interactive disk space analyzers.
* btop

---

### 23. Bonus: Java on the Command Line

Most of you may know that since 21 we can make one file java scriptrs.

One limitation is that if you want everything self-contained like this you can’t have any dependencies on external jars



You don't need a full Maven or Gradle project structure to run Java utilities or quick automation scripts.

**Zero-Boilerplate Execution with JBang (`jbang`):**

* Run single-file `.java` source files directly as executables.
* Inline dependency resolution via `//DEPS` comments (pulls from Maven Central automatically).
* Direct Shebang support (`#!/usr/bin/env jbang`).
* Native stdin/stdout stream handling for seamless integration into Unix pipelines.

**Slide Bullet Points**

* **//DEPS Declarations:** Declare external libraries directly inside the `.java` header.
* **Shebang Scripting:** Execute Java files as standalone CLI binaries.
* **Zero Setup & Caching:** Automatic JDK management and cached artifact compilation.
* **Unix Stream Pipelines:** Read from `System.in` and write to `System.out` via Java streams.

---

### 24. Summary & Key Takeaways

To turn your terminal into an effective daily engine, focus on four principles:

1. **Maintain a Dotfiles Repository:** Store your configurations in Git for reproducible setups across machines.
2. **Adopt Modern Replacements Incrementally:** Swap one legacy tool at a time (e.g., switch from `grep` to `rg` first).
3. **Learn Core Vim Keybindings:** Basic navigation keys (`h, j, k, l`, `w, b`, `/`) translate directly across CLI tools, pagers, and prompts.
4. **Automate Common Sequences:** If you execute the same multi-step command sequence three times, convert it into an alias or shell function.

---

### 25. Outro / Q&A

*(Closing Slide)*