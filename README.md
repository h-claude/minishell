# 🐚 Minishell

A simple, lightweight Unix shell implementation written in C. This project recreates basic functionality of bash, featuring command parsing, execution, pipes, redirections, and built-in commands.

> A 42 School project focused on learning Unix process management, signal handling, and building a functional command-line interpreter.

---

## 📋 Table of Contents

- [About](#about)
- [Technologies](#technologies)
- [Features](#features)
- [Installation](#installation)
- [Usage](#usage)
- [Built-in Commands](#built-in-commands)
- [Project Structure](#project-structure)
- [Authors](#authors)

---

## 📖 About

Minishell is a minimal shell implementation that mimics the behavior of bash. It provides an interactive command-line interface where users can execute commands, navigate directories, manage environment variables, and chain commands using pipes and redirections.

This project demonstrates understanding of:
- Process creation and management (fork, exec, wait)
- File descriptors and I/O redirection
- Signal handling (SIGINT, SIGQUIT)
- Lexical analysis and parsing (Abstract Syntax Tree)
- Memory management with custom garbage collector

---

## 🛠️ Technologies

| Technology | Description |
|------------|-------------|
| ![C](https://img.shields.io/badge/C-00599C?style=flat&logo=c&logoColor=white) **C Language** | Core programming language (C99 standard) |
| ![Make](https://img.shields.io/badge/Make-427819?style=flat&logo=gnu&logoColor=white) **GNU Make** | Build automation tool |
| ![Readline](https://img.shields.io/badge/GNU_Readline-4EAA25?style=flat&logo=gnu&logoColor=white) **GNU Readline** | Command-line editing and history |
| ![Termcap](https://img.shields.io/badge/Termcap-333333?style=flat&logoColor=white) **Termcap** | Terminal capability library |

### Libraries Used

- **libft** - Custom implementation of standard C library functions
  - String manipulation functions (ft_strlen, ft_strdup, ft_split, etc.)
  - Memory management functions (ft_calloc, ft_bzero, ft_memcpy, etc.)
  - Linked list operations (ft_lstadd_back, ft_lstnew, etc.)
  - Custom garbage collector for automatic memory management
  - get_next_line for reading file descriptors

- **readline** - GNU library for command-line input
  - Line editing capabilities
  - Command history with arrow key navigation
  - Custom prompt display

- **termcap** - Terminal capability database
  - Terminal-independent screen handling

### Compiler & Flags

```
Compiler: cc (gcc/clang)
Flags: -Wall -Wextra -Werror -g3
```

---

## ✨ Features

### Shell Features
- ✅ Interactive command prompt with custom display
- ✅ Command history (up/down arrows)
- ✅ Signal handling (Ctrl+C, Ctrl+D, Ctrl+\)
- ✅ Exit status codes ($?)
- ✅ Environment variable expansion ($VAR)

### Command Execution
- ✅ Execute commands from PATH
- ✅ Execute commands with absolute/relative paths
- ✅ Handle command arguments

### Pipes & Redirections
- ✅ Pipes (`|`) - Chain commands together
- ✅ Input redirection (`<`) - Read from file
- ✅ Output redirection (`>`) - Write to file
- ✅ Append redirection (`>>`) - Append to file
- ✅ Here-document (`<<`) - Read until delimiter

### Quoting
- ✅ Single quotes (`'`) - Prevent interpretation
- ✅ Double quotes (`"`) - Allow variable expansion

---

## 🚀 Installation

### Prerequisites

Ensure you have the following installed:
- GCC or Clang compiler
- GNU Make
- GNU Readline library
- Termcap library

**On Debian/Ubuntu:**
```bash
sudo apt-get update
sudo apt-get install build-essential libreadline-dev
```

**On macOS (with Homebrew):**
```bash
brew install readline
```

### Build

1. **Clone the repository:**
```bash
git clone https://github.com/h-claude/minishell.git
cd minishell
```

2. **Compile the project:**
```bash
make
```

3. **Clean object files:**
```bash
make clean
```

4. **Full clean (remove executable):**
```bash
make fclean
```

5. **Rebuild:**
```bash
make re
```

---

## 💻 Usage

### Starting the Shell

```bash
./minishell
```

You will see an interactive prompt:
```
DEDSEC ❋ /current/directory$ > 
```

### Example Commands

```bash
# Simple command
DEDSEC ❋ ~$ > ls -la

# Environment variables
DEDSEC ❋ ~$ > echo $HOME

# Pipes
DEDSEC ❋ ~$ > ls -la | grep minishell | wc -l

# Redirections
DEDSEC ❋ ~$ > echo "Hello World" > output.txt
DEDSEC ❋ ~$ > cat < input.txt

# Append redirection
DEDSEC ❋ ~$ > echo "New line" >> output.txt

# Here-document
DEDSEC ❋ ~$ > cat << EOF
> Line 1
> Line 2
> EOF

# Chaining with pipes and redirections
DEDSEC ❋ ~$ > cat file.txt | grep pattern | sort > sorted.txt
```

---

## 🔧 Built-in Commands

| Command | Description | Usage |
|---------|-------------|-------|
| `echo` | Display text | `echo [-n] [string...]` |
| `cd` | Change directory | `cd [path]` |
| `pwd` | Print working directory | `pwd` |
| `export` | Set environment variable | `export [name=value]` |
| `unset` | Remove environment variable | `unset [name]` |
| `env` | Display environment variables | `env` |
| `exit` | Exit the shell | `exit [status]` |

---

## 📁 Project Structure

```
minishell/
├── Makefile                    # Build configuration
├── README.md                   # This file
├── includes/
│   └── minishell.h             # Main header file
├── libft/                      # Custom C library
│   ├── Makefile
│   ├── libft.h
│   ├── ft_*.c                  # Standard library reimplementations
│   ├── get_next_line.c         # File reading function
│   └── garbage_collector/      # Memory management
│       ├── garbage_collector.c
│       └── call_functions.c
└── srcs/
    ├── builtins/               # Built-in command implementations
    │   ├── cd/
    │   │   ├── cd.c
    │   │   ├── cd_utils.c
    │   │   └── cd_utils_env.c
    │   ├── echo.c
    │   ├── env.c
    │   ├── exit.c
    │   ├── export.c
    │   ├── pwd.c
    │   └── unset.c
    ├── execution/              # Command execution
    │   ├── exec.c
    │   ├── exec_utils.c
    │   ├── redirect.c
    │   ├── utils.c
    │   └── exec_pipe/          # Pipe handling
    │       ├── exec_pipe.c
    │       ├── manage_pid.c
    │       ├── ast_to_list.c
    │       ├── redirection_pipe.c
    │       └── utils_for_pipe.c
    ├── parsing/                # Lexer and Parser
    │   ├── main.c              # Entry point
    │   ├── lexer.c             # Tokenization
    │   ├── lexer_utils.c
    │   ├── parser.c            # AST generation
    │   ├── parsecmd.c
    │   ├── vars.c              # Variable expansion
    │   └── utils.c
    └── utils/                  # Utility functions
        ├── utils.c
        ├── utils2.c
        ├── copy_env.c
        ├── signals.c
        └── print_errors.c
```

### Architecture Overview

```
┌─────────────────────────────────────────────────────────────┐
│                        User Input                           │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                     Readline Library                        │
│              (Command editing & history)                    │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                         Lexer                               │
│        (Tokenize input into words, operators, etc.)         │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                        Parser                               │
│           (Build Abstract Syntax Tree - AST)                │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                       Executor                              │
│    (Execute commands, handle pipes & redirections)          │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                        Output                               │
└─────────────────────────────────────────────────────────────┘
```

---

## 👥 Authors

| Author | GitHub |
|--------|--------|
| **Hippolyte Claude** | [@h-claude](https://github.com/h-claude) |
| **Mohamed Ali Ajili** | [@moajili](https://github.com/ajilidali) |

---

## 📝 License

This project is part of the 42 School curriculum.

---

## 🙏 Acknowledgments

- [42 School](https://42.fr/) for the project subject
- [GNU Readline](https://tiswww.case.edu/php/chet/readline/rltop.html) documentation
- [Bash Reference Manual](https://www.gnu.org/software/bash/manual/bash.html)

---

<p align="center">
  Made with ❤️ at 42 School
</p>
