# VoIP-OnCall CLI Command Reference

Welcome to the comprehensive CLI Command Reference for VoIP-OnCall, a powerful command-line interface designed to manage and operate VoIP-based communications systems. This document serves as an exhaustive guide to using VoIP-OnCall's CLI, detailing every command, flag, argument, and providing examples for each. Whether you are a seasoned systems administrator or a new user, this reference will help you make the most of the VoIP-OnCall CLI.

## Table of Contents

1. [Introduction](#introduction)
2. [Getting Started](#getting-started)
3. [Command Syntax](#command-syntax)
4. [Global Options](#global-options)
5. [Commands Overview](#commands-overview)
6. [Detailed Command Reference](#detailed-command-reference)
   - [voip-oncall start](#voip-oncall-start)
   - [voip-oncall stop](#voip-oncall-stop)
   - [voip-oncall restart](#voip-oncall-restart)
   - [voip-oncall status](#voip-oncall-status)
   - [voip-oncall call](#voip-oncall-call)
   - [voip-oncall hangup](#voip-oncall-hangup)
   - [voip-oncall list](#voip-oncall-list)
   - [voip-oncall config](#voip-oncall-config)
   - [voip-oncall logs](#voip-oncall-logs)
7. [Examples](#examples)
8. [Troubleshooting](#troubleshooting)
9. [Best Practices](#best-practices)
10. [Conclusion](#conclusion)

## Introduction

VoIP-OnCall is a command-line tool designed for managing VoIP communication systems. This tool allows users to start, stop, and monitor VoIP services, initiate and manage calls, and configure system settings. The CLI provides a powerful interface for quick and automated management of VoIP services.

## Getting Started

Before using the VoIP-OnCall CLI, ensure that it is installed on your system. You can install it using the package manager of your choice, or by downloading it from the official website.

```bash
# Install via package manager
sudo apt-get install voip-oncall
```

Verify the installation by running the following command:

```bash
voip-oncall --version
```

## Command Syntax

The general syntax for using the VoIP-OnCall CLI is as follows:

```bash
voip-oncall [command] [options]
```

- **command**: The specific action you want to perform.
- **options**: Additional flags and arguments that modify the behavior of the command.

## Global Options

Global options can be applied to any command to modify its behavior:

- `-h, --help`: Display help information about a command.
- `-v, --version`: Display the version of the VoIP-OnCall CLI.
- `-c, --config <path>`: Specify a custom configuration file.
- `--verbose`: Enable verbose output for debugging purposes.

## Commands Overview

Here is a quick overview of the primary commands available in VoIP-OnCall:

- `start`: Start the VoIP service.
- `stop`: Stop the VoIP service.
- `restart`: Restart the VoIP service.
- `status`: Check the status of the VoIP service.
- `call`: Initiate a VoIP call.
- `hangup`: Hang up an ongoing call.
- `list`: List all active calls.
- `config`: Configure VoIP service settings.
- `logs`: View logs for the VoIP service.

## Detailed Command Reference

### voip-oncall start

Start the VoIP service.

#### Syntax

```bash
voip-oncall start [options]
```

#### Options

- `--foreground`: Run the service in the foreground.
- `--debug`: Start the service in debug mode.

#### Examples

Start the VoIP service in the background:

```bash
voip-oncall start
```

Start the VoIP service in the foreground with debug mode enabled:

```bash
voip-oncall start --foreground --debug
```

### voip-oncall stop

Stop the VoIP service.

#### Syntax

```bash
voip-oncall stop [options]
```

#### Options

- `--force`: Force stop the service.

#### Examples

Stop the VoIP service gracefully:

```bash
voip-oncall stop
```

Force stop the VoIP service:

```bash
voip-oncall stop --force
```

### voip-oncall restart

Restart the VoIP service.

#### Syntax

```bash
voip-oncall restart [options]
```

#### Options

- `--fast`: Perform a fast restart.

#### Examples

Restart the VoIP service:

```bash
voip-oncall restart
```

Perform a fast restart:

```bash
voip-oncall restart --fast
```

### voip-oncall status

Check the status of the VoIP service.

#### Syntax

```bash
voip-oncall status [options]
```

#### Options

- `--output <format>`: Specify the output format (`text`, `json`).

#### Examples

Check the service status in text format:

```bash
voip-oncall status
```

Check the service status in JSON format:

```bash
voip-oncall status --output json
```

### voip-oncall call

Initiate a VoIP call.

#### Syntax

```bash
voip-oncall call <target> [options]
```

#### Arguments

- `target`: The destination number or SIP address.

#### Options

- `--audio-only`: Initiate an audio-only call.
- `--video`: Enable video for the call.
- `--codec <codec>`: Specify the codec to use.

#### Examples

Initiate an audio-only call:

```bash
voip-oncall call 123456789 --audio-only
```

Initiate a video call using a specific codec:

```bash
voip-oncall call sip:example@domain.com --video --codec opus
```

### voip-oncall hangup

Hang up an ongoing call.

#### Syntax

```bash
voip-oncall hangup <call-id> [options]
```

#### Arguments

- `call-id`: The identifier of the call to hang up.

#### Options

- `--all`: Hang up all ongoing calls.

#### Examples

Hang up a specific call:

```bash
voip-oncall hangup 987654321
```

Hang up all calls:

```bash
voip-oncall hangup --all
```

### voip-oncall list

List all active calls.

#### Syntax

```bash
voip-oncall list [options]
```

#### Options

- `--output <format>`: Specify the output format (`text`, `json`).

#### Examples

List all active calls in text format:

```bash
voip-oncall list
```

List all active calls in JSON format:

```bash
voip-oncall list --output json
```

### voip-oncall config

Configure VoIP service settings.

#### Syntax

```bash
voip-oncall config <setting> <value> [options]
```

#### Arguments

- `setting`: The configuration parameter to set.
- `value`: The value to assign to the parameter.

#### Options

- `--apply`: Apply changes immediately.

#### Examples

Set the audio codec configuration:

```bash
voip-oncall config audio.codec opus
```

Apply changes immediately:

```bash
voip-oncall config audio.codec opus --apply
```

### voip-oncall logs

View logs for the VoIP service.

#### Syntax

```bash
voip-oncall logs [options]
```

#### Options

- `--tail <n>`: Show the last `n` lines of logs.
- `--follow`: Follow the log output in real-time.
- `--level <level>`: Set the log level (`info`, `debug`, `error`).

#### Examples

View the last 50 lines of logs:

```bash
voip-oncall logs --tail 50
```

Follow the log output in real-time:

```bash
voip-oncall logs --follow
```

## Examples

Here are some practical examples of how to use the VoIP-OnCall CLI.

### Example 1: Starting and Stopping the Service

Start the VoIP service in the background and then stop it:

```bash
voip-oncall start
voip-oncall stop
```

### Example 2: Initiating a Call

Initiate a video call to a SIP address:

```bash
voip-oncall call sip:contact@domain.com --video
```

### Example 3: Configuring Settings

Set and apply a new audio codec configuration:

```bash
voip-oncall config audio.codec g711 --apply
```

## Troubleshooting

If you encounter issues with VoIP-OnCall, consider the following troubleshooting steps:

- Ensure the service is running using `voip-oncall status`.
- Check logs for errors using `voip-oncall logs --level error`.
- Verify network connectivity to the SIP server.
- Use the `--verbose` option for detailed command output.

## Best Practices

- Regularly check the status of your VoIP service.
- Keep your configuration files backed up.
- Monitor logs for any anomalies or errors.
- Use secure SIP protocols and encryption where possible.

## Conclusion

This comprehensive CLI Command Reference for VoIP-OnCall should serve as a valuable resource for managing and operating your VoIP systems. By leveraging the power of the command line, you can efficiently control VoIP services, ensuring reliable and flexible communication solutions.