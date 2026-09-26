# minishell

> A POSIX-subset command interpreter written in C: lexes, parses into an AST, and executes pipelines of external and built-in commands with I/O redirection, heredocs, and signal-aware job control.

---

## 📌 Problem Overview & Classification

- **Domain / Problem Category:** Low-Level Systems Programming / Unix Process & Inter-Process Communication (IPC)
- **Theoretical Problem:** Shell Grammar Interpretation (Lexer → Recursive-Descent Parser → AST evaluator), combined with the classic *fork/exec/wait* process-control model and a *pipe-based Producer-Consumer* pattern for heredocs and multi-stage pipelines.
- **Core Objective:** Translate a raw input line into an executable process graph — a linked chain of `fork`ed processes connected by anonymous pipes, each with its file descriptors correctly redirected — while keeping built-in commands (which must mutate the parent's own environment/cwd) running in-process.
- **Key Constraints & Challenges:**
  - Correct file-descriptor bookkeeping across N-stage pipelines (no leaked read/write ends, no double-close).
  - Distinguishing commands that must run in the parent (`cd`, `export`, `unset`, `exit`, …) from those that require `fork`+`execve`.
  - Heredoc (`<<`) input must be collected in a child process so `SIGINT` during collection doesn't kill the shell, while still feeding the result back through a `pipe(2)`.
  - Signal behavior differs by context: interactive prompt (`SIGINT` redraws the line), heredoc collection (`SIGINT` aborts the heredoc only), and executing children (`SIGINT`/`SIGQUIT` reset to `SIG_DFL`).
  - Deterministic memory/FD cleanup on every exit path (`exit` builtin, `Ctrl-D`, fatal error) via a single garbage-collector-backed `ft_exit`.

---

## ⚙️ Architecture & Implementation Details

- **Modular Design:**
  - `srcs/parsing/` — `lexer.c` / `lexer_utils.c` tokenize the input line into `TOKEN_WORD`, `TOKEN_VARIABLE`, `TOKEN_PIPE`, `TOKEN_OPERATOR`, `TOKEN_STRING`, `TOKEN_EMPTY`, `TOKEN_EOF` (`t_token`, `t_lexer` in [minishell.h](includes/minishell.h)). `parser.c` / `parsecmd.c` implement a recursive-descent parser (`parse_pipeline` → `parse_command`) producing a `t_astnode` tree of type `AST_COMMAND`, `AST_PIPELINE`, or `AST_ERR`. `vars.c` handles `$VAR` / `$?` expansion (`replace_variables`, `parse_variable`).
  - `srcs/execution/` — `exec.c` walks the AST: single commands call `is_local_fct` (builtin dispatch) or `fork`+`execve`; `redirect.c` resolves `<`, `>`, `>>`, `<<` into real file descriptors per node before execution.
  - `srcs/execution/exec_pipe/` — for `AST_PIPELINE` nodes, `ast_to_list.c` flattens the (left-associative) pipe tree into a `t_lst_cmd` linked list, `exec_pipe.c` forks one process per stage wiring `pipe(2)` fds between consecutive commands, `manage_pid.c` tracks/reaps every spawned PID.
  - `srcs/builtins/` — one file per built-in (`cd/`, `echo.c`, `env.c`, `exit.c`, `export.c`, `pwd.c`, `unset.c`), each operating directly on the shell's live `t_env` list so state (cwd, exported variables) persists across commands.
  - `srcs/utils/` — `signals.c` (signal handlers per shell mode), `copy_env.c` / `utils2.c` (the `t_env` singly-linked list, exposed process-wide through the `give_envp`/`give_mini` static-pointer accessors), `print_errors.c` (centralized bash-style diagnostics).
  - `libft/` — a from-scratch libc subset plus `garbage_collector/`, a reference-less allocation tracker (`t_gc` linked list) used by every heap allocation in the project.

- **Key Primitives & Mechanisms:**
  - `fork` / `execve` / `waitpid`: one child per pipeline stage ([exec.c](srcs/execution/exec.c), [exec_pipe.c](srcs/execution/exec_pipe/exec_pipe.c)); exit status extracted via `WEXITSTATUS`/`WTERMSIG`.
  - `pipe(2)` + `dup2(2)`: connects pipeline stages and implements heredocs — `<<` forks a dedicated collector process that writes each `readline`-free line (via `get_next_line`) into the write end, and the consumer `dup`s the read end onto its `fd_in`.
  - `sigaction` / `signal`: three distinct handler sets swapped in via `setup_signal_handler(flag)` — interactive prompt (`handler`, redraws the readline buffer on `SIGINT`), heredoc collection (`handle_sigint_heredoc`), and executing-child mode where both signals are restored to `SIG_DFL` right before `execve`.
  - `access(X_OK)` + custom `find_path`: resolves absolute/relative paths directly, otherwise walks `$PATH` entries manually before falling back to `ER_CMD_NOT_FOUND` (exit 127) / `ER_PERM_DENIED` (exit 126), matching bash's own exit-code convention.
  - `open(2)` with `O_WRONLY|O_CREAT|O_TRUNC` (`>`) or `O_APPEND` (`>>`), `O_RDONLY` (`<`): redirections are resolved into `node->fd_in`/`fd_out` before the fork, and the parent's original stdio is preserved via `dup(STDIN_FILENO)`/`dup(STDOUT_FILENO)` and restored after the command completes.
  - `getcwd` / `chdir`: back `cd`, `cd -` (via a synthetic `OLDPWD`/`PWD` pair kept in the env list) and the dynamic prompt (`make_prompt` reads a hidden `PWD_HIDE` env entry).

- **Lifecycle & Resource Management:** All heap allocations go through `ft_malloc`, registered in a process-global `t_gc` list ([garbage_collector.c](libft/garbage_collector/garbage_collector.c)); `ft_free` detaches and frees a single node, `ft_free_gb` walks and frees the whole list. Every exit path — the `exit` builtin, `Ctrl-D` (`EOF` from `readline`), and fatal internal errors — funnels through `ft_exit(status)` ([call_functions.c](libft/garbage_collector/call_functions.c)), which frees the garbage-collector list, `close`s file descriptors `0..1023`, then calls `exit(status)`, guaranteeing no dangling FDs or unfreed tracked allocations regardless of exit reason. Per-command redirections save/restore the shell's own stdin/stdout with `dup`/`dup2` so a redirection on one command never leaks into the next.

---

## 🛠️ Stack & Tooling

| Category | Tools / Technologies |
| :--- | :--- |
| **Language / Standard** | C (C99 — uses `<stdbool.h>`), POSIX.1-2001 system calls |
| **System Primitives** | `fork`, `execve`, `waitpid`, `pipe`, `dup2`, `sigaction`, `open`/`close`, `chdir`/`getcwd` |
| **External Libraries** | GNU `readline` (line editing, history), `termcap` |
| **Compiler & Flags** | `cc` (gcc/clang) with `-Wall -Wextra -Werror -Wno-unused-function -g3` |
| **Build System** | GNU Make (recursive build into `libft/`) |

---

## 🚀 Getting Started

### Prerequisites
- Linux or macOS with a C compiler (`gcc`/`clang`) and GNU Make.
- GNU `readline` + `termcap` development headers.
  - macOS (Homebrew): `brew install readline` — the top-level [Makefile](Makefile) links against the Intel Homebrew prefix `/usr/local/opt/readline` explicitly; on Apple Silicon (`/opt/homebrew`) or Linux, adjust `-I`/`-L` in the `Makefile` link rule accordingly.
  - Debian/Ubuntu: `sudo apt-get install build-essential libreadline-dev`

### Compilation
```bash
make
```

### Usage
```bash
./minishell
```
Drops into an interactive prompt reflecting the current directory:
```
DEDSEC ❋ /current/directory$ >
```

*Example:*
```bash
DEDSEC ❋ ~$ > echo "$HOME" | grep -i user | wc -l
DEDSEC ❋ ~$ > cat << EOF > out.txt
> hello $USER
> EOF
DEDSEC ❋ ~$ > cd - && pwd
```

### Build Targets
- `make`: Compiles `libft` then the `minishell` binary.
- `make clean`: Removes intermediate object files (`.objs/`, `libft`'s own objects).
- `make fclean`: Runs `clean` and additionally removes the compiled `minishell` binary.
- `make re`: Equivalent to `fclean` followed by `all`.

---

## 🧪 Testing & Reliability Verification

The repository ships no automated test harness; the checks below are the manual verification commands the implementation is designed to pass.

### Memory Leak & Concurrency Checks
```bash
# Verify zero memory leaks (each ft_exit() path frees the garbage-collector list)
valgrind --leak-check=full --show-leak-kinds=all --trace-children=yes ./minishell

# Confirm signal handling under interactive use
./minishell
# then press Ctrl-C (SIGINT) at the prompt, Ctrl-\ (SIGQUIT), and Ctrl-D (EOF)
```

- [x] All allocations tracked through `ft_malloc`/`ft_free`/`ft_free_gb`, released on every `ft_exit()` path.
- [x] `ft_exit()` closes file descriptors `0`–`1023` unconditionally before terminating.
- [x] `SIGINT` at the prompt redraws the line and sets `$?` to `1`; inside a heredoc it aborts collection; during child execution it falls back to default behavior.
- [x] `SIGQUIT` (`Ctrl-\`) prints `Quit: 3` and sets exit status `131`, mirroring bash.
- [x] `exit` builtin validates numeric arguments (rejects non-digit input, "too many arguments"), and wraps values `> 255` modulo `256`.
- [x] Command lookup distinguishes "command not found" (127) from "permission denied" (126), matching POSIX shell exit-status conventions.

---

## 👤 Author

- **Hippolyte Claude** — [@h-claude](https://github.com/h-claude)
- **Mohamed Ali Ajili** — [@ajilidali](https://github.com/ajilidali)
