# Go CLI Command Reference

Go, often referred to as Golang, is a statically typed, compiled programming language designed for simplicity and high performance. The Go command-line interface (CLI) offers a robust set of tools for managing Go code, building projects, testing, and more. This comprehensive reference provides detailed information on each command, flag, and argument available in the Go CLI.

## Table of Contents

1. [Overview](#overview)
2. [Basic Commands](#basic-commands)
    - [go help](#go-help)
    - [go version](#go-version)
    - [go env](#go-env)
3. [Building and Installing](#building-and-installing)
    - [go build](#go-build)
    - [go install](#go-install)
    - [go clean](#go-clean)
4. [Dependency Management](#dependency-management)
    - [go mod](#go-mod)
    - [go get](#go-get)
5. [Testing](#testing)
    - [go test](#go-test)
6. [Code Generation](#code-generation)
    - [go generate](#go-generate)
7. [Formatting and Linting](#formatting-and-linting)
    - [go fmt](#go-fmt)
    - [go vet](#go-vet)
8. [Advanced Commands](#advanced-commands)
    - [go tool](#go-tool)
    - [go fix](#go-fix)
9. [Examples](#examples)
10. [Conclusion](#conclusion)
11. [References](#references)

## Overview

The Go CLI provides a suite of commands for developers to interact with Go codebases. From building and running applications to managing dependencies and testing, the Go CLI facilitates efficient development processes. 

## Basic Commands

### go help

The `go help` command provides help for Go commands and their usage.

**Usage:**

```bash
go help [command]
```

**Examples:**

- Get general help:

  ```bash
  go help
  ```

- Get help for a specific command:

  ```bash
  go help build
  ```

### go version

The `go version` command prints the installed Go version.

**Usage:**

```bash
go version
```

**Example:**

```bash
go version
```

The output will be something like:

```text
go version go1.19 linux/amd64
```

### go env

The `go env` command prints Go environment information.

**Usage:**

```bash
go env [var ...]
```

**Examples:**

- Print all environment variables:

  ```bash
  go env
  ```

- Print a specific environment variable:

  ```bash
  go env GOPATH
  ```

## Building and Installing

### go build

The `go build` command compiles the packages named by the import paths.

**Usage:**

```bash
go build [build flags] [packages]
```

**Key Flags:**

- `-o`: Specify the output file name.

**Examples:**

- Build the current package:

  ```bash
  go build
  ```

- Build with a custom output name:

  ```bash
  go build -o myapp
  ```

### go install

The `go install` command compiles and installs the packages.

**Usage:**

```bash
go install [build flags] [packages]
```

**Examples:**

- Install the current package:

  ```bash
  go install
  ```

- Install a specific package:

  ```bash
  go install ./cmd/myapp
  ```

### go clean

The `go clean` command removes object files and cached files.

**Usage:**

```bash
go clean [clean flags] [packages]
```

**Key Flags:**

- `-i`: Remove the installed archive or binary.
- `-modcache`: Remove the entire module cache.

**Examples:**

- Clean the current package:

  ```bash
  go clean
  ```

- Remove installed binaries:

  ```bash
  go clean -i
  ```

## Dependency Management

### go mod

The `go mod` command provides access to operations on modules.

**Subcommands:**

- `init`: Initialize a new module in the current directory.
- `tidy`: Add missing and remove unused modules.
- `vendor`: Make vendored copy of dependencies.

**Examples:**

- Initialize a new module:

  ```bash
  go mod init example.com/myapp
  ```

- Tidy up the module dependencies:

  ```bash
  go mod tidy
  ```

### go get

The `go get` command adds dependencies to the current module and installs them.

**Usage:**

```bash
go get [packages]
```

**Examples:**

- Get the latest version of a package:

  ```bash
  go get github.com/gin-gonic/gin
  ```

- Get a specific version of a package:

  ```bash
  go get github.com/gin-gonic/gin@v1.7.4
  ```

## Testing

### go test

The `go test` command is used to test Go packages.

**Usage:**

```bash
go test [build/test flags] [packages] [flags for test binary]
```

**Key Flags:**

- `-v`: Verbose output.
- `-cover`: Enable coverage analysis.
- `-run`: Run only those tests and examples matching the regular expression.

**Examples:**

- Run tests for the current package:

  ```bash
  go test
  ```

- Run tests with verbose output:

  ```bash
  go test -v
  ```

- Run tests with coverage:

  ```bash
  go test -cover
  ```

## Code Generation

### go generate

The `go generate` command scans the source code for `//go:generate` directives and executes them.

**Usage:**

```bash
go generate [generate flags] [packages]
```

**Examples:**

- Execute generate directives in the current package:

  ```bash
  go generate
  ```

- Run generate for specific files:

  ```bash
  go generate ./...
  ```

## Formatting and Linting

### go fmt

The `go fmt` command formats Go source files.

**Usage:**

```bash
go fmt [packages]
```

**Examples:**

- Format the current package:

  ```bash
  go fmt
  ```

- Format all packages in the module:

  ```bash
  go fmt ./...
  ```

### go vet

The `go vet` command examines Go source code and reports suspicious constructs.

**Usage:**

```bash
go vet [vet flags] [packages]
```

**Examples:**

- Vet the current package:

  ```bash
  go vet
  ```

- Vet all packages in the module:

  ```bash
  go vet ./...
  ```

## Advanced Commands

### go tool

The `go tool` command runs a specified go tool.

**Usage:**

```bash
go tool [command] [args...]
```

**Examples:**

- Compile a Go package:

  ```bash
  go tool compile main.go
  ```

### go fix

The `go fix` command updates Go code to the latest language standards.

**Usage:**

```bash
go fix [packages]
```

**Examples:**

- Fix the current package:

  ```bash
  go fix
  ```

## Examples

### Building and Running a Simple Go Application

1. Create a simple Go application:

   ```go
   // main.go
   package main
   
   import "fmt"
   
   func main() {
       fmt.Println("Hello, Go!")
   }
   ```

2. Build the application:

   ```bash
   go build -o hello
   ```

3. Run the application:

   ```bash
   ./hello
   ```

### Managing Dependencies

1. Initialize a new module:

   ```bash
   go mod init example.com/myapp
   ```

2. Add a dependency:

   ```bash
   go get github.com/gin-gonic/gin
   ```

3. Tidy up the module:

   ```bash
   go mod tidy
   ```

### Testing with Coverage

1. Write a simple test:

   ```go
   // main_test.go
   package main
   
   import "testing"
   
   func TestMain(t *testing.T) {
       if true != true {
           t.Error("Expected true to be true")
       }
   }
   ```

2. Run the tests with coverage:

   ```bash
   go test -cover
   ```

## Conclusion

The Go CLI is a powerful interface that facilitates the management of Go codebases. With commands for building, testing, formatting, and more, the Go CLI is essential for any Go developer. Understanding these commands and their usage can significantly enhance productivity and streamline development workflows.

## References

- [Go Official Documentation](https://golang.org/doc/)
- [Go Command Reference](https://golang.org/cmd/)
- [Effective Go](https://golang.org/doc/effective_go.html)
- [Go Modules Reference](https://golang.org/ref/mod)