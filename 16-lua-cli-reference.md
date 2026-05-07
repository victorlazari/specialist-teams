# Lua Command Line Interface (CLI) Reference

## Introduction to Lua CLI

Lua is a lightweight, high-level, multi-paradigm programming language designed primarily for embedded use in applications. Its simplicity and efficiency have led to widespread use in various domains, such as game development, web services, and configuration management. The Lua Command Line Interface (CLI) serves as a powerful tool for running Lua scripts, interacting with the Lua interpreter, and managing Lua environments.

The Lua CLI enables users to execute Lua code files, interact with the Lua interpreter in a REPL (Read-Eval-Print Loop) environment, and customize execution via command-line flags and environment variables. This document provides an exhaustive reference to the Lua CLI, detailing each command, flag, argument, and environment variable, alongside comprehensive examples and best practices.

## Basic Usage

The Lua CLI is invoked via the `lua` command, followed by various options, scripts, and arguments. At its simplest form, executing Lua from the command line involves:

```sh
lua script.lua
```

This command runs the Lua script `script.lua`. If no script is provided, the interpreter enters an interactive mode, where Lua code can be executed line by line.

```sh
lua
```

In this mode, users can experiment with Lua code snippets interactively.

## Command Line Options

The Lua CLI supports several command line options that modify its behavior. These options allow users to execute code directly from the command line, load libraries, control the execution environment, and more.

### `-e <stat>`

The `-e` option executes the given Lua statement. This is particularly useful for running short scripts or setting a global variable.

```sh
lua -e "print('Hello, World!')"
```

This command directly executes the `print` statement, outputting "Hello, World!" to the console.

### `-l <name>`

The `-l` option loads the specified module using `require`. This is useful for initializing libraries or setting up dependencies before executing a script.

```sh
lua -l math -e "print(math.sqrt(16))"
```

This command loads the `math` library and calculates the square root of 16.

### `-i`

The `-i` option forces the interpreter into interactive mode after executing the specified script or command. This is useful for debugging or exploring the environment after script execution.

```sh
lua -i -e "x = 5"
```

After setting `x` to 5, the interpreter enters interactive mode, allowing further exploration with `x` available in the environment.

### `-v`

The `-v` option outputs the version information of the Lua interpreter. This is valuable when debugging or ensuring compatibility with specific Lua versions.

```sh
lua -v
```

This command outputs the version number of the Lua interpreter.

### `-E`

The `-E` option prevents the execution of environment-related scripts, such as those specified in the `LUA_INIT` variable. This ensures a clean environment for script execution.

```sh
lua -E script.lua
```

By using `-E`, the script is executed without any pre-defined environment modifications.

### `-W`

The `-W` option turns warnings on. Lua typically suppresses warnings, but enabling them can be useful for debugging or ensuring code quality.

```sh
lua -W script.lua
```

This command runs `script.lua` with warnings enabled, providing additional information about potential issues.

## Environment Variables

Lua's behavior can be customized further using environment variables, which allow for dynamic configuration without altering the code directly.

### `LUA_INIT`

The `LUA_INIT` variable can specify a string of Lua code or a file to be executed before the interpreter runs any script. This is useful for setting up environment configurations or loading essential libraries.

```sh
export LUA_INIT='print("Lua Initialization")'
lua
```

This setup prints "Lua Initialization" every time the Lua interpreter starts.

### `LUA_PATH`

The `LUA_PATH` variable defines the search path for Lua scripts. It uses a semicolon-separated list of patterns that the interpreter looks through to find required modules.

```sh
export LUA_PATH='./?.lua;/usr/local/share/lua/5.4/?.lua'
```

This configuration allows the interpreter to locate Lua files in the current directory and a system-wide directory.

### `LUA_CPATH`

Similar to `LUA_PATH`, the `LUA_CPATH` variable defines the search path for C libraries used by Lua, typically in the form of shared libraries or DLLs.

```sh
export LUA_CPATH='./?.so;/usr/local/lib/lua/5.4/?.so'
```

This setup helps the interpreter locate compiled C libraries for Lua extensions.

## Interactive Mode (REPL) Deep Dive

The Lua interpreter can operate in an interactive mode, which is a REPL environment. This allows users to dynamically execute Lua statements, test functions, and experiment with code snippets.

### Starting the REPL

Simply executing `lua` without any arguments starts the REPL:

```sh
lua
```

In this mode, users can enter Lua statements, and the interpreter will evaluate and print the results immediately.

### Features of the REPL

- **Immediate Evaluation:** Enter Lua statements directly, and the interpreter evaluates them on the fly.
  
  ```lua
  > print("Hello, REPL!")
  Hello, REPL!
  ```

- **Variable Persistence:** Variables defined in the REPL persist across commands.

  ```lua
  > x = 10
  > print(x * 2)
  20
  ```

- **Function Testing:** Quickly test functions and snippets before integrating them into larger scripts.

  ```lua
  > function greet(name) return "Hello, " .. name end
  > print(greet("Alice"))
  Hello, Alice
  ```

- **Error Handling:** Errors are reported immediately, facilitating quick debugging.

  ```lua
  > print(unknownVariable)
  stdin:1: attempt to index a nil value (global 'unknownVariable')
  stack traceback:
      stdin:1: in main chunk
      [C]: in ?
  ```

### Exiting the REPL

To exit the interactive mode, simply use the `os.exit()` function or an EOF signal (Ctrl+D on Unix, Ctrl+Z on Windows).

```lua
> os.exit()
```

## Executing Scripts

Lua scripts are typically executed by passing the script file to the `lua` command. The script can accept arguments from the command line, which are accessible within the script through the `arg` table.

### Script Arguments

The `arg` table contains all command-line arguments passed to the script. The script name is stored at `arg[0]`, and subsequent arguments are stored in `arg[1]`, `arg[2]`, etc.

**Example Script:**

`script.lua`

```lua
print("Script name:", arg[0])
for i = 1, #arg do
  print("Argument " .. i .. ":", arg[i])
end
```

**Executing the Script:**

```sh
lua script.lua first second third
```

**Output:**

```
Script name: script.lua
Argument 1: first
Argument 2: second
Argument 3: third
```

### Handling Edge Cases

- **No Arguments**: If no arguments are passed, `arg` contains only the script name.
  
  ```sh
  lua script.lua
  ```

  Output:
  ```
  Script name: script.lua
  ```

- **Quoted Arguments**: Arguments containing spaces should be quoted.

  ```sh
  lua script.lua "hello world"
  ```

  Output:
  ```
  Script name: script.lua
  Argument 1: hello world
  ```

## Advanced Usage Examples

### Combining Options

Using multiple command-line options can customize the execution further. For example, loading a module and entering interactive mode:

```sh
lua -l json -i
```

This loads the `json` module and opens the REPL, ready to parse JSON data.

### Using `LUA_INIT` for Setup

Pre-configure the environment using `LUA_INIT` to load essential libraries every time:

```sh
export LUA_INIT='require("my_custom_lib")'
lua
```

This ensures `my_custom_lib` is available in every session.

### Embedding Lua in Shell Scripts

Combine Lua with shell scripting for powerful toolchains.

**Script:**

```sh
#!/bin/sh
lua -e "print('Running Lua from a shell script!')"
```

Make the script executable and run it:

```sh
chmod +x my_script.sh
./my_script.sh
```

Output:

```
Running Lua from a shell script!
```

## Best Practices for CLI Usage

- **Environment Isolation**: Use the `-E` option when scripts should run in a clean environment, unaffected by `LUA_INIT`.

- **Performance Considerations**: For performance-critical applications, ensure unnecessary libraries are not loaded, and use the `-E` option to skip environment initialization.

- **Debugging**: Enable warnings with `-W` during development to catch potential issues early.

- **Version Management**: Regularly check the Lua version with `-v` to ensure compatibility, especially when deploying across different environments.

- **Environment Variables**: Use `LUA_PATH` and `LUA_CPATH` to manage module and library paths effectively, facilitating easier deployment and modularization of code.

- **Interactive Exploration**: Use the REPL for rapid prototyping and testing of code snippets, benefiting from immediate feedback and the ability to iterate quickly.

By understanding and leveraging the full capabilities of the Lua CLI, developers can enhance their productivity, streamline their workflows, and ensure consistent, reliable execution of Lua scripts in various environments.