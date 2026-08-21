# Effective Terminal: Comprehensive Cheatsheet

## 1. Terminal Stack Architecture & Setup

The terminal environment consists of three distinct layers:

| Layer                      | Responsibility                                                    | Recommended Stack                                                                                           |
| :------------------------- | :---------------------------------------------------------------- | :---------------------------------------------------------------------------------------------------------- |
| **1. Terminal Emulator**   | Graphical window rendering text and capturing input               | **macOS:** iTerm2 / Kitty<br>**Linux:** Alacritty / Kitty / GNOME Terminal<br>**Windows:** Windows Terminal |
| **2. Shell Interpreter**   | Parses commands, manages processes, expands variables             | **Zsh** (rich completion, advanced globbing) or **Bash** (server default)                                   |
| **3. Framework / Plugins** | Adds themes, git integration, syntax highlighting, autocompletion | **Oh My Zsh**, **Prezto**, or **Zimfw** + `zsh-autosuggestions`                                             |

* **GPU Acceleration:** Prefer GPU-accelerated emulators (e.g., Alacritty, Kitty, WezTerm) over legacy software-rendered consoles to prevent rendering bottlenecks during large log outputs.
* **Windows Parity (WSL2):** Use **WSL2 (Ubuntu)** instead of legacy pseudo-consoles (Git Bash, ConEmu, Cygwin) to ensure full POSIX compliance and Linux filesystem performance.

---

## 2. Core Text Utilities & Modern Replacements

### Classic vs. Modern Drop-in Replacements

| Legacy Tool    | Modern Drop-in         | Key Advantages                                        | Example Command              |
| :------------- | :--------------------- | :---------------------------------------------------- | :--------------------------- |
| `grep`         | **`rg`** (ripgrep)     | Multi-threaded, respects `.gitignore`, 10x faster     | `rg "Error" /var/log`        |
| `find`         | **`fd`**               | Intuitive syntax, ignores hidden/git files by default | `fd -e tsv`                  |
| `cat`          | **`bat`**              | Syntax highlighting, line numbers, git modifications  | `bat config.yaml`            |
| `sed`          | **`sd`**               | Cleaner regex syntax, simplified `find replace`       | `sd "HTTP" "HTTPS" api.conf` |
| `ls`           | **`eza`** / **`lsd`**  | Tree hierarchy, file icons, inline git status         | `eza -la --git --tree`       |
| `cat *.tsv`    | **`tv`** (Tidy Viewer) | Column alignment, alternating row colors for TSV/CSV  | `tv spells.tsv`              |
| `top` / `htop` | **`btop`**             | Visual resource monitor (CPU, memory, disk, network)  | `btop`                       |

### Core Line & Stream Tools
* `cat <file>`: Dumps raw file content.
* `nl <file>`: Numbers every line of output (useful for indexing and referencing line numbers).
* `wc -l <file>`: Returns line count instantly.
* `head -n <N>` / `tail -n <N>`: Inspect first/last $N$ lines (`tail -n +2` skips header line).
* `cut -f<N>`: Extracts field $N$ (tab-delimited by default).
* `sort` / `uniq -c`: Sorts lines and counts unique occurrences (always run `sort` before `uniq`).

---

## 3. Standard Streams & I/O Redirection

Every Unix process runs with three standard file descriptors:
* `0 (stdin)`: Standard input stream.
* `1 (stdout)`: Standard output stream.
* `2 (stderr)`: Standard diagnostic/error stream.

```bash
cmd > file.txt          # Overwrite stdout to file
cmd >> file.txt         # Append stdout to file
cmd 2> error.log        # Redirect stderr to file
cmd 2>&1                # Merge stderr into stdout
cmd | cmd2              # Pipe ONLY stdout (Port 1) to cmd2
cmd > /dev/null 2>&1    # Completely silence execution (stdout + stderr discarded)
```

> **Note:** The pipe operator (`|`) redirects **only stdout**. Any output on `stderr` bypasses downstream pipe stages unless merged via `2>&1 |`.

---

## 4. Batch Execution with `xargs`

`xargs` takes lines from `stdin` and converts them into arguments for subsequent commands, avoiding explicit `for` loops.

```bash
# Basic usage: create directories from distinct values in a TSV column
cat spells.tsv | cut -f3 | tail -n +2 | sort -u | xargs -I {} mkdir -p "spell_{}"

# Run subshells per item to generate isolated files
cat spells.tsv | cut -f3 | tail -n +2 | sort -u | xargs -I {} sh -c 'rg "{}" spells.tsv > "spell_{}/spells.txt"'

# Parallel batch execution (-P defines concurrency limit)
cat urls.txt | xargs -P 8 -I {} curl -O {}
```

---

## 5. Navigation, History & Lookups

* **`z` / `zoxide`**: Frecent directory navigation (tracks frequency + recency).
  ```bash
  z project-name        # Direct teleport without typing paths
  ```
* **`fzf`**: Interactive fuzzy finder for files, history, and processes.
  ```bash
  Ctrl + R              # Interactive fuzzy history search
  alias goto='cd $(fd -t d | fzf)'  # Interactive fuzzy directory jumping
  ```
* **In-Terminal Documentation**:
  ```bash
  tldr tar              # Practical, real-world cheatsheet examples
  rg --help / -h        # Fast flag reference
  man fd                # Comprehensive manual
  ```

---

## 6. Dotfile Architecture & Automation

Maintain an operational environment that is reproducible across machines:

1. **Version Control:** Store `~/.zshrc`, shell aliases, and tool configs in a single Git repository (`~/dotfiles`).
2. **Symlinking:** Symlink repository files to user home directories (`ln -s ~/dotfiles/.zshrc ~/.zshrc`).
3. **The Rule of Three:** If you execute the same multi-step command sequence three times, convert it into an alias or custom shell function.
4. **Vim Keybindings:** Master basic motion keys (`h, j, k, l`, `w, b`, `/`) to navigate pagers (`less`), terminal prompts, and command history instantly.

---

## 7. Structured Data & Log Processing (JSON, YAML, CSV)

| Utility            | Primary Function                                      | Common Syntax                                       |
| :----------------- | :---------------------------------------------------- | :-------------------------------------------------- |
| **`jq`**           | JSON parser, filter, and transformer                  | `cat data.json \| jq '.[].name'`                    |
| **`yq`**           | YAML/XML processor with `jq`-compatible syntax        | `yq '.services.web.image' docker-compose.yml`       |
| **`gron`**         | Flattens JSON into line-by-line greppable assignments | `gron data.json \| rg "status" \| gron --ungron`    |
| **`yj`**           | Ultra-fast data format converter                      | `yj -jy < data.json > data.yaml`                    |
| **`mlr`** (Miller) | Relational SQL-like operations on CSV/TSV/JSON        | `mlr --t2j cat data.tsv`                            |
| **`youplot`**      | Visual CLI plotting engine for metrics and logs       | `youplot histogram -n 10 -t "Latency Distribution"` |

### Real-World Processing Pipelines

```bash
# 1. Filter TSV, convert to JSON on the fly, and structure with jq
cat spells_catalog.tsv \
  | rg --invert-match "Necromancy" \
  | mlr --t2j cat \
  | jq 'group_by(.type)[] | {(.[0].type): (map(.name))}' \
  | jq -s '.'

# 2. Extract metrics from application logs and generate an in-terminal histogram
cat app.log \
  | rg --only-matching '\d+\.\d+ seconds' \
  | sd " seconds" "" \
  | youplot histogram -n 10 --ylabel seconds -t "Application Starting Times"
```

---

## 8. Specialized & Unobvious Utilities

* **`cal -3`**: Monospace calendar displaying previous, current, and next month side-by-side.
* **`trans`** (*Translate Shell*): Inline translation engine directly inside the terminal (`trans :pl "Starting process"`).
* **`pngpaste`**: Pastes image data directly from system clipboard to a disk file.
* **`tesseract`**: Terminal OCR engine (`tesseract screenshot.png stdout`).
* **`diskbloom`**: Interactive, visual disk space analyzers.
* **`btop`**: Process manager TUI, modern alternative of `top`

---

## 9. Java on the Command Line: JBang vs. Vanilla Java 21+

Java 21 supports executing single `.java` files directly (`java Script.java`), but vanilla Java is limited when used for CLI automation and production scripting.

### Why Use JBang Instead of Standard Java 21?

| Feature                   | Vanilla Java 21 (`java App.java`)                                            | JBang (`jbang App.java`)                                                            |
| :------------------------ | :--------------------------------------------------------------------------- | :---------------------------------------------------------------------------------- |
| **Dependency Management** | ❌ None (requires full Maven/Gradle project or manual classpath construction) | **`//DEPS` annotations** pull transitive artifacts directly from Maven Central      |
| **JDK Management**        | ❌ Uses globally configured `$JAVA_HOME`                                      | **`//JAVA` directives** download and isolate required JDK runtimes automatically    |
| **CLI Execution Model**   | ❌ Must invoke via `java Script.java`                                         | **Native Shebang support** (`#!/usr/bin/env jbang`) runs scripts as `./script.java` |
| **Execution Caching**     | ⚠️ Recompiles source on every execution                                       | Caches compiled bytecode and resolved JAR artifacts                                 |
| **Remote Execution**      | ❌ Local files only                                                           | Executes scripts directly from URLs / GitHub Gists                                  |
| **Argument Parsing**      | ❌ Manual `args[]` index parsing                                              | Built-in native integration with **Picocli**                                        |

### Complete Executable JBang Template

Save as `filter_spells.java`, grant executable permissions (`chmod +x filter_spells.java`), and run in pipelines:

```java
///usr/bin/env jbang "$0" "$@" ; exit $?
//JAVA 21
//DEPS com.fasterxml.jackson.core:jackson-databind:2.17.0
//DEPS info.picocli:picocli:4.7.5

import com.fasterxml.jackson.databind.ObjectMapper;
import picocli.CommandLine;
import picocli.CommandLine.Command;
import picocli.CommandLine.Option;

import java.io.BufferedReader;
import java.io.InputStreamReader;
import java.util.concurrent.Callable;

@Command(
    name = "filter_spells",
    mixinStandardHelpOptions = true,
    description = "Pipes and processes stream data via JBang"
)
class FilterSpells implements Callable<Integer> {

    @Option(names = {"-p", "--prefix"}, description = "Prefix tag", defaultValue = "LOG")
    private String prefix;

    public static void main(String... args) {
        int exitCode = new CommandLine(new FilterSpells()).execute(args);
        System.exit(exitCode);
    }

    @Override
    public Integer call() throws Exception {
        ObjectMapper mapper = new ObjectMapper();

        // Native standard input stream consumption
        try (BufferedReader reader = new BufferedReader(new InputStreamReader(System.in))) {
            String line;
            while ((line = reader.readLine()) != null) {
                if (!line.isBlank()) {
                    System.out.println("[" + prefix + "] " + line.trim().toUpperCase());
                }
            }
        }
        return 0;
    }
}
```

```bash
# Execute standalone directly in a Unix pipeline
cat spells_catalog.tsv | ./filter_spells.java --prefix="SPELL"
```
