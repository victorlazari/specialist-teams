# Bash Troubleshooting & Diagnostics Guide

This comprehensive guide provides detailed instructions on troubleshooting and diagnosing issues in Bash scripts. It covers common error codes, recovery strategies, health checks, and frequent issues encountered when using Bash.

## Table of Contents
1. [Common Error Codes in Bash](#common-error-codes-in-bash)
2. [Recovery Strategies](#recovery-strategies)
3. [Health Checks](#health-checks)
4. [Common Issues and Solutions](#common-issues-and-solutions)
5. [Advanced Debugging Techniques](#advanced-debugging-techniques)
6. [Best Practices for Writing Robust Bash Scripts](#best-practices-for-writing-robust-bash-scripts)

## Common Error Codes in Bash

Understanding Bash exit codes is crucial for diagnosing issues. By convention, a command returns an exit status of zero if it succeeds, and a non-zero status if it fails.

### Standard Exit Codes
- **0**: Success
- **1**: General error
- **2**: Misuse of shell builtins
- **126**: Command invoked cannot execute
- **127**: Command not found
- **128**: Invalid argument to exit
- **130**: Script terminated by Control-C
- **255**: Exit status out of range

### Custom Exit Codes
Users can define custom exit codes in their scripts. Use numbers between 3 and 125 to avoid conflicts with standard exit codes.

#### Example:
```bash
#!/bin/bash

# Custom exit code for file not found
FILE_NOT_FOUND=3

if [[ ! -f "/path/to/file" ]]; then
  echo "File not found!"
  exit $FILE_NOT_FOUND
fi
```

## Recovery Strategies

When a Bash script fails, the following strategies can be employed to recover and continue script execution.

### Error Handling with `trap`

The `trap` command in Bash allows you to execute commands when the script receives specific signals or exits.

#### Example:
```bash
#!/bin/bash

trap 'echo "An error occurred. Exiting..."; exit 1;' ERR

# Commands that may fail
cp /nonexistent/file /tmp/
```

### Using `set` for Error Handling

The `set` built-in can be used to modify shell behavior for error recovery.

#### Common Options:
- `set -e`: Exit immediately if a command exits with a non-zero status.
- `set -u`: Treat unset variables as an error and exit immediately.
- `set -o pipefail`: Make pipelines fail if any command fails.

#### Example:
```bash
#!/bin/bash

set -euo pipefail

echo "Starting script..."
cp /nonexistent/file /tmp/
echo "This will not be printed if cp fails."
```

### Implementing Rollback Mechanisms

For scripts that modify system state, implement rollback mechanisms to revert changes in case of failure.

#### Example:
```bash
#!/bin/bash

function rollback {
  echo "Rolling back changes..."
  # Commands to undo changes
}

trap rollback ERR

# Commands that modify system state
touch /tmp/samplefile
cp /nonexistent/file /tmp/
```

## Health Checks

Regular health checks can help ensure that Bash scripts function correctly and minimize potential downtime.

### Check Syntax with `bash -n`

Before running a script, check its syntax using `bash -n`.

#### Example:
```bash
bash -n my_script.sh
```

### Use `shellcheck` for Static Analysis

`shellcheck` is a tool for static analysis of shell scripts, highlighting potential issues and offering suggestions.

#### Installation:
```bash
sudo apt-get install shellcheck  # Debian-based systems
brew install shellcheck          # macOS
```

#### Usage:
```bash
shellcheck my_script.sh
```

### Monitor Resource Usage with `time`

Use the `time` command to measure the resources used by your script.

#### Example:
```bash
time ./my_script.sh
```

### Validate Environment Variables

Before executing commands, validate that required environment variables are set.

#### Example:
```bash
: "${REQUIRED_VAR:?REQUIRED_VAR is not set}"
```

## Common Issues and Solutions

### Issue: File or Directory Not Found
- **Cause**: Incorrect path or missing file/directory.
- **Solution**: Validate paths before accessing files or directories.

#### Example:
```bash
if [[ ! -d "/expected/directory" ]]; then
  echo "Directory not found!"
  exit 1
fi
```

### Issue: Permission Denied
- **Cause**: Insufficient permissions to access a file or execute a command.
- **Solution**: Check file permissions and adjust as necessary.

#### Example:
```bash
chmod +x my_script.sh
./my_script.sh
```

### Issue: Command Not Found
- **Cause**: Incorrect command or missing executable in `PATH`.
- **Solution**: Ensure the command is installed and the path is correct.

#### Example:
```bash
if ! command -v my_command &> /dev/null; then
  echo "my_command could not be found"
  exit 1
fi
```

### Issue: Argument List Too Long
- **Cause**: Exceeding the system's limit for command-line arguments.
- **Solution**: Use xargs or split arguments into batches.

#### Example:
```bash
find . -type f -print0 | xargs -0 -n 1000 rm -f
```

## Advanced Debugging Techniques

### Enable Shell Debugging

Use `set -x` to print each command before execution for debugging purposes.

#### Example:
```bash
#!/bin/bash

set -x  # Enable debugging

echo "Debugging script..."
cp /nonexistent/file /tmp/
```

### Use `PS4` for Enhanced Debugging

Customize the `PS4` variable to include additional information in debug output.

#### Example:
```bash
#!/bin/bash

export PS4='+ ${BASH_SOURCE}:${LINENO}:${FUNCNAME[0]}: '
set -x

echo "Enhanced debugging..."
cp /nonexistent/file /tmp/
```

### Use a Debugger: `bashdb`

`bashdb` is a debugger for Bash scripts, allowing breakpoints and step-by-step execution.

#### Installation:
```bash
sudo apt-get install bashdb  # Debian-based systems
```

#### Usage:
```bash
bashdb my_script.sh
```

### Logging with `logger`

Use `logger` to send script output to the system log, facilitating centralized logging.

#### Example:
```bash
#!/bin/bash

logger "Starting my script..."
cp /nonexistent/file /tmp/ 2>&1 | logger
logger "Finished running my script."
```

## Best Practices for Writing Robust Bash Scripts

### Use `#!/bin/bash` Shebang

Always specify the shell interpreter using a shebang (`#!/bin/bash`) at the top of your scripts.

### Use `declare` for Variables

Use `declare` to specify variable types and ensure proper handling.

#### Example:
```bash
declare -i my_integer=10  # Integer
declare -r my_constant="constant_value"  # Read-only
```

### Quote Variables

Always quote variables to prevent word splitting and globbing.

#### Example:
```bash
echo "The value is: $my_variable"
```

### Prefer `$(...)` Over Backticks

Use `$(...)` for command substitution as it is more readable and nests better than backticks.

#### Example:
```bash
current_date=$(date +%Y-%m-%d)
```

### Use Functions for Reusable Code

Encapsulate code in functions to improve readability and reusability.

#### Example:
```bash
function greet {
  echo "Hello, $1!"
}

greet "World"
```

### Handle Signals Gracefully

Implement signal handling to clean up resources properly.

#### Example:
```bash
#!/bin/bash

function cleanup {
  echo "Cleaning up..."
  # Cleanup code here
}

trap cleanup EXIT

# Main script code
```

By following this comprehensive guide, you can effectively troubleshoot, diagnose, and improve the robustness of your Bash scripts.