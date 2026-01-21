# MyShell
**Author:** Tushar Cora Suresh

---

## Overview

MyShell is a custom Unix shell implementation developed as part of a systems programming course (CS214). It provides a command-line interface that replicates core shell functionality while adding advanced features like wildcard expansion and conditional command execution. The shell emphasizes compliance with standard Unix shell behavior and demonstrates understanding of process management, file descriptors, and inter-process communication.

---

## Project Structure

```
MyShell/
├── mysh.c          # Main shell implementation
├── Makefile        # Build configuration
├── batchtests.sh   # Batch mode test script
├── mysh            # Compiled executable
└── TestCases/      # Test files and scenarios
```

### File Descriptions

**mysh.c**
- Core shell implementation with command parsing and execution
- Process management using `fork()` and `exec()`
- I/O redirection and pipeline handling
- Built-in command implementations
- Wildcard expansion logic
- Conditional execution based on exit status

**Makefile**
- Compiles `mysh.c` into the `mysh` executable
- Includes build and clean targets

**batchtests.sh**
- Comprehensive batch mode test script
- Tests conditional execution (`then`/`else`)
- Validates built-in commands and navigation
- Verifies batch mode functionality

**TestCases/**
- Directory containing test files for various scenarios
- Input/output files for redirection testing
- Sample executables for testing

---

## Features

### 1. Dual Operating Modes

**Interactive Mode**
- Displays a prompt for user input
- Processes commands one at a time
- Automatically detected using `isatty()`
- Provides immediate feedback

**Batch Mode**
- Reads commands from a script file
- Executes commands sequentially
- No prompt display
- Useful for automation and testing

Usage:
```bash
# Interactive mode
./mysh

# Batch mode
./mysh batchtests.sh
```

---

### 2. Built-in Commands

MyShell implements essential built-in commands that execute directly without forking:

**`cd [directory]`**
- Changes the current working directory
- Supports relative and absolute paths
- Handles `..` (parent directory) and `.` (current directory)

**`pwd`**
- Prints the current working directory
- Outputs absolute path

**`which [command]`**
- Locates the path to an executable
- Searches directories in PATH
- Returns the full path if found

**`exit`**
- Terminates the shell
- Cleans up resources before exiting

Examples:
```bash
cd /home/user
pwd
which gcc
cd ..
exit
```

---

### 3. Command Execution

**Path Resolution**
- Searches for executables in PATH
- Supports absolute paths (`/bin/ls`)
- Supports relative paths (`./myprogram`)
- Handles bare command names (`ls`, `gcc`)

**Argument Parsing**
- Tokenizes command strings into arguments
- Supports up to 1,000 tokens per command
- Each token can be up to 1,000 characters
- Maximum command length: 10,000 characters

**Process Management**
- Uses `fork()` to create child processes
- Executes commands with `execv()`
- Parent waits for child completion
- Tracks exit status for conditional execution

---

### 4. Input/Output Redirection

MyShell supports standard I/O redirection using `<` and `>` operators.

**Output Redirection (`>`)**
- Redirects stdout to a file
- Creates file with permissions `0640` (rw-r-----)
- Overwrites existing files

**Input Redirection (`<`)**
- Redirects stdin from a file
- Reads file content as command input

**Implementation**
- Uses `dup2()` for file descriptor manipulation
- Opens files with appropriate flags
- Properly closes file descriptors after use

Examples:
```bash
ls > output.txt
cat < input.txt
./program > results.txt
sort < data.txt > sorted.txt
```

---

### 5. Pipelines

MyShell supports **single-level piping** between two commands using the `|` operator.

**Pipeline Implementation**
- Creates unnamed pipe using `pipe()`
- Forks two child processes
- First process writes to pipe
- Second process reads from pipe
- Parent waits for both children

**Limitations**
- Supports exactly one pipe per command
- No chained pipes (e.g., `cmd1 | cmd2 | cmd3`)

Examples:
```bash
ls | grep .txt
cat file.txt | wc -l
ps aux | grep mysh
./test | grep Hello
```

---

### 6. Combining Redirection and Pipes

MyShell handles complex commands with both redirection and pipes.

**Precedence Rules**
- Redirection takes precedence over pipelines
- Input redirection overrides pipe input
- Output redirection overrides pipe output

Examples:
```bash
# Pipe with output redirection
ls | grep .txt > results.txt

# Pipe with input redirection (input overrides pipe)
cat < input.txt | grep "pattern"

# Combined input and output redirection with pipe
sort < data.txt | uniq > output.txt

# Input file overrides pipe input
ls | grep "pattern" < input.txt
```

---

### 7. Wildcard Expansion

MyShell implements wildcard (`*`) expansion for file matching.

**Wildcard Behavior**
- Matches files in the current directory
- Expands `*` to all matching filenames
- Supports multiple wildcards in one command
- Matches zero or more characters

**Limitations**
- Wildcards cannot immediately follow redirection symbols
- Only matches files in the current directory
- No recursive directory matching

Examples:
```bash
ls *.txt
cat *bar.txt
rm test*
gcc *.c -o program
```

---

### 8. Conditional Execution

MyShell implements conditional execution based on the exit status of previous commands.

**`then` Command**
- Executes only if the previous command **succeeded** (exit status 0)
- Similar to `&&` in bash

**`else` Command**
- Executes only if the previous command **failed** (exit status ≠ 0)
- Similar to `||` in bash

**Implementation**
- Global status variable tracks last command's exit status
- Checked before executing `then`/`else` commands

Examples:
```bash
cd /valid/path
then echo "Success!"
else echo "Failed!"

mkdir newdir
then cd newdir
then pwd

rm nonexistent
else echo "File not found"
```

---

### 9. Special Token Handling

MyShell preprocesses input to ensure special characters are recognized as individual tokens.

**Special Characters**
- `<` - Input redirection
- `>` - Output redirection  
- `|` - Pipeline

**Token Processing**
- Special characters are always separate tokens
- Whitespace around special characters is optional
- Commands like `ls>file` and `ls > file` are equivalent

Examples:
```bash
# All these are equivalent:
ls>output.txt
ls> output.txt
ls >output.txt
ls > output.txt
```

---

## Building and Running

### Compilation

```bash
make
```

This produces the `mysh` executable.

### Running MyShell

**Interactive Mode:**
```bash
./mysh
```

**Batch Mode:**
```bash
./mysh batchtests.sh
```

### Cleaning Build Artifacts

```bash
make clean
```

---

## Testing

MyShell includes comprehensive testing to ensure correctness and compliance with Unix shell behavior.

### Test Categories

#### 1. Basic Functionality Tests

**Built-in Commands**
```bash
pwd
cd test
cd ..
which cd
which touch
exit
```

**Simple Command Execution**
```bash
touch testfile
rm testfile
gcc test.c -o test
mv test newName
./test
./a.out
```

**Nested Shell Execution**
```bash
./mysh
```

---

#### 2. Redirection Tests

**Output Redirection**
```bash
./test > test_output
pwd > pathCheck
echo "Hello World" > greeting.txt
```

**Combined Redirection**
```bash
./test > output < input
sort < unsorted.txt > sorted.txt
```

**File Permissions**
- Verifies output files are created with mode `0640`
- Checks file creation and overwriting behavior

---

#### 3. Pipeline Tests

**Simple Pipes**
```bash
ls | grep .txt
ps aux | grep mysh
cat file.txt | wc -l
```

**Combined Redirection and Pipes**
```bash
cat < input | grep "pattern" > output
sort < data.txt | uniq > results.txt
ls | grep "pattern" < input
ls | grep "pattern" > output
```

**Precedence Verification**
- Input redirection overrides pipe input
- Output redirection captures final output
- Validates correct precedence handling

---

#### 4. Wildcard Tests

**Setup**
```bash
touch file1.txt
touch file2.txt
touch bar.txt
touch foobar.txt
```

**Wildcard Expansion**
```bash
ls *.txt
ls *bar.txt
cat *.c
rm test*
```

**Multiple Wildcards**
```bash
cp *.txt *.backup
```

---

#### 5. Batch Mode and Conditional Tests

**batchtests.sh Script**
```bash
echo Hello!
echo Test2
cd testcases
./test
cd ..
cd
then cd testcases
pwd
```

**Conditional Execution**
```bash
# Success then success
mkdir testdir
then cd testdir

# Failure then else
cd nonexistent
else echo "Directory not found"

# Success chain
touch newfile
then ls newfile
then echo "File created successfully"
```

---

#### 6. Comparison with Bash

To ensure compliance, MyShell's output is compared with bash for:
- Command execution results
- Exit status handling
- File creation behavior
- Redirection output
- Pipeline behavior
- Wildcard expansion results

---

### Test Execution

**Manual Testing**
```bash
./mysh
# Enter commands interactively
```

**Automated Batch Testing**
```bash
./mysh batchtests.sh
```

**Comparison Testing**
```bash
# Run in MyShell
./mysh < test_commands.txt > mysh_output.txt

# Run in Bash
bash < test_commands.txt > bash_output.txt

# Compare
diff mysh_output.txt bash_output.txt
```

---

## Implementation Details

### Low-Level I/O Operations

**Command Reading**
- Uses `read()` for command input
- Handles line buffering manually
- Maximum command length: 10,000 characters

**Prompt Display**
- Uses `write()` for prompt output
- Only displayed in interactive mode
- Detected using `isatty(STDIN_FILENO)`

### Command Processing Pipeline

1. **Input Parsing**
   - Read command string
   - Preprocess special characters
   - Tokenize into arguments

2. **Token Analysis**
   - Identify built-in commands
   - Detect redirection operators
   - Locate pipe operators
   - Expand wildcards

3. **Execution**
   - Built-ins: Execute directly
   - External: Fork and exec
   - Redirection: Setup file descriptors
   - Pipes: Create pipe and fork twice

4. **Status Tracking**
   - Store exit status globally
   - Enable conditional execution

### Executable Path Resolution

1. Search for absolute paths
2. Search for relative paths (contains `/`)
3. Search PATH directories for bare names
4. Return full path if found, NULL otherwise

### File Descriptor Management

**Redirection**
```c
// Output redirection
int fd = open(filename, O_WRONLY | O_CREAT | O_TRUNC, 0640);
dup2(fd, STDOUT_FILENO);
close(fd);

// Input redirection
int fd = open(filename, O_RDONLY);
dup2(fd, STDIN_FILENO);
close(fd);
```

**Pipelines**
```c
int pipefd[2];
pipe(pipefd);

// Child 1: Write end
dup2(pipefd[1], STDOUT_FILENO);

// Child 2: Read end
dup2(pipefd[0], STDIN_FILENO);
```

---

## Design Decisions and Constraints

### Limitations

1. **Single-Level Piping**
   - Only one pipe (`|`) per command
   - No support for `cmd1 | cmd2 | cmd3`

2. **Wildcard Constraints**
   - Wildcards cannot immediately follow `<` or `>`
   - Only matches current directory
   - No recursive directory matching

3. **Token Limits**
   - Maximum 1,000 tokens per command
   - Maximum 1,000 characters per token
   - Maximum 10,000 characters per command

4. **Redirection Precedence**
   - Redirection always takes priority over pipes
   - Input redirection overrides pipe input

### Design Choices

**Why `read()` and `write()` instead of `fgets()` and `printf()`?**
- Provides low-level control over I/O
- Demonstrates understanding of system calls
- Avoids buffering issues

**Why track exit status globally?**
- Enables conditional execution (`then`/`else`)
- Simulates shell variables like `$?` in bash

**Why limit to single-level pipes?**
- Simplifies implementation for educational purposes
- Focuses on core piping mechanism understanding
- Still covers the fundamental concept

**Why special token preprocessing?**
- Ensures consistent parsing regardless of whitespace
- Handles edge cases like `ls>file` vs `ls > file`
- Improves user experience

---

## Educational Value

This project demonstrates understanding of:

1. **Process Management**
   - `fork()`, `exec()`, `wait()`
   - Parent-child relationships
   - Exit status handling

2. **File Descriptors**
   - Standard I/O (stdin, stdout, stderr)
   - `dup2()` for redirection
   - File descriptor inheritance

3. **Inter-Process Communication**
   - Unnamed pipes
   - Pipe read/write ends
   - Process coordination

4. **System Calls**
   - Low-level I/O (`read()`, `write()`, `open()`, `close()`)
   - Directory operations (`chdir()`, `getcwd()`)
   - Process control

5. **String Processing**
   - Tokenization
   - Pattern matching for wildcards
   - Command parsing

6. **Shell Behavior**
   - Built-in vs external commands
   - PATH resolution
   - Exit status semantics

---

## Future Enhancements

Potential improvements could include:

- Multi-level piping (`cmd1 | cmd2 | cmd3`)
- Background processes (`&`)
- Job control (`fg`, `bg`, `jobs`)
- Environment variable expansion (`$VAR`)
- Command history and editing
- Tab completion
- Signal handling (Ctrl+C, Ctrl+Z)
- Recursive wildcard matching (`**`)
- Append redirection (`>>`)
- Error redirection (`2>`, `2>&1`)
- Command substitution (`` `cmd` ``)
- Scripting features (loops, conditionals, functions)

---

## Known Issues and Edge Cases

**Edge Cases Handled:**
- Empty commands
- Commands with only whitespace
- Missing files for redirection
- Non-existent executables
- Invalid directory for `cd`

**Assumptions:**
- At most one pipe per command
- Wildcards don't immediately follow redirection
- Commands fit within size limits
- Files exist for input redirection
- Valid syntax for redirection and pipes
## Acknowledgments

Developed as part of CS214 (Systems Programming) to demonstrate understanding of process management, file I/O, and Unix shell behavior.
