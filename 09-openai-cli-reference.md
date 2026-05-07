# OpenAI CLI Command Reference

## Table of Contents
1. [Introduction](#introduction)
2. [Installation](#installation)
3. [Authentication](#authentication)
4. [Global Flags](#global-flags)
5. [Commands](#commands)
    - [API](#api)
    - [Models](#models)
    - [Chat](#chat)
    - [Completions](#completions)
    - [Images](#images)
    - [Audio](#audio)
    - [Files](#files)
    - [Fine-tuning](#fine-tuning)
6. [Troubleshooting](#troubleshooting)
7. [Conclusion](#conclusion)

## Introduction

The OpenAI CLI (Command Line Interface) is a powerful tool that allows developers to interact with OpenAI's APIs directly from the terminal. It provides a comprehensive set of commands for managing models, generating text completions, and handling various AI-powered tasks. This document serves as a detailed reference for the OpenAI CLI, covering installation, authentication, usage of commands, flags, arguments, and troubleshooting common issues.

## Installation

To begin using the OpenAI CLI, you must first install it on your system. The OpenAI CLI is available as an npm package, which you can install using Node.js. Ensure that you have Node.js (version 12 or later) and npm installed on your system.

### Steps for Installation

1. **Install Node.js and npm:**
   - Download and install Node.js from the official website: [Node.js Downloads](https://nodejs.org/).
   - Verify the installation:
     ```bash
     node -v
     npm -v
     ```

2. **Install the OpenAI CLI:**
   - Use npm to install the OpenAI CLI globally:
     ```bash
     npm install -g openai-cli
     ```
   - Verify the installation:
     ```bash
     openai --version
     ```

3. **Update the OpenAI CLI:**
   - Keep the CLI updated to the latest version for new features and bug fixes:
     ```bash
     npm update -g openai-cli
     ```

## Authentication

Before you can use the OpenAI CLI, you need to authenticate with your OpenAI API key. The API key is required to access OpenAI's services.

### Setting Up Authentication

1. **Obtain your API Key:**
   - Log in to your OpenAI account and navigate to the API keys section.
   - Create a new API key if you don’t have one.

2. **Configure the CLI with your API Key:**
   - Set your API key as an environment variable:
     ```bash
     export OPENAI_API_KEY="your-api-key-here"
     ```
   - Alternatively, configure the CLI directly:
     ```bash
     openai config set api_key your-api-key-here
     ```

3. **Verify Authentication:**
   - Test the configuration by running a simple command:
     ```bash
     openai models list
     ```

## Global Flags

Global flags are options that apply to all commands within the OpenAI CLI. These flags can be used to modify the behavior of the CLI or provide additional functionality.

| Flag       | Description                       | Example Usage                          |
|------------|-----------------------------------|----------------------------------------|
| `--help`   | Display help information          | `openai --help`                        |
| `--version`| Show the version of the CLI       | `openai --version`                     |
| `--api-key`| Specify a custom API key          | `openai --api-key your-api-key api`    |
| `--quiet`  | Suppress output                   | `openai --quiet models list`           |
| `--output` | Specify output format (json/text) | `openai --output json models list`     |

## Commands

The OpenAI CLI supports a variety of commands, each tailored to specific functionalities. This section provides a comprehensive overview of each command, including flags, arguments, and examples.

### API

The `api` command is used to interact with OpenAI's API directly, allowing users to make custom requests.

#### Usage

```bash
openai api [options]
```

#### Flags and Arguments

| Flag          | Description                         | Example                                |
|---------------|-------------------------------------|----------------------------------------|
| `--endpoint`  | Specify the API endpoint            | `openai api --endpoint /v1/engines`    |
| `--method`    | HTTP method (GET, POST, etc.)       | `openai api --method POST`             |
| `--data`      | Data to send with the request       | `openai api --data '{"key":"value"}'`  |

#### Examples

1. **List Engines:**
   ```bash
   openai api --endpoint /v1/engines --method GET
   ```

2. **Create a Completion:**
   ```bash
   openai api --endpoint /v1/completions --method POST --data '{"model": "text-davinci-002", "prompt": "Translate the following English text to French: \"Hello, world!\"", "max_tokens": 60}'
   ```

### Models

The `models` command allows users to list and manage models available in OpenAI.

#### Usage

```bash
openai models [command] [options]
```

#### Subcommands

- **list**: List all available models.
- **get**: Retrieve details about a specific model.

#### Flags and Arguments

| Subcommand | Flag        | Description                   | Example                                |
|------------|-------------|-------------------------------|----------------------------------------|
| list       | `--all`     | List all models               | `openai models list --all`             |
| get        | `--model-id`| Specify the model ID          | `openai models get --model-id text-ada-001`|

#### Examples

1. **List Models:**
   ```bash
   openai models list
   ```

2. **Get Model Details:**
   ```bash
   openai models get --model-id text-davinci-002
   ```

### Chat

The `chat` command is used to create chat-based interactions with OpenAI's models.

#### Usage

```bash
openai chat [options]
```

#### Flags and Arguments

| Flag       | Description                     | Example                                |
|------------|---------------------------------|----------------------------------------|
| `--model`  | Specify the model to use        | `openai chat --model gpt-3.5-turbo`    |
| `--stream` | Stream responses in real-time   | `openai chat --stream`                 |

#### Examples

1. **Basic Chat:**
   ```bash
   openai chat --model gpt-3.5-turbo --prompt "Hello, how are you?"
   ```

2. **Streamed Chat:**
   ```bash
   openai chat --model gpt-3.5-turbo --prompt "Tell me about the history of AI." --stream
   ```

### Completions

The `completions` command generates text completions using OpenAI's models.

#### Usage

```bash
openai completions [options]
```

#### Flags and Arguments

| Flag         | Description                           | Example                                        |
|--------------|---------------------------------------|------------------------------------------------|
| `--model`    | Specify the model to use              | `openai completions --model text-davinci-002`  |
| `--prompt`   | Input prompt for the completion       | `openai completions --prompt "Once upon a time"`|
| `--max-tokens` | Maximum number of tokens in response | `openai completions --max-tokens 100`          |

#### Examples

1. **Generate Completion:**
   ```bash
   openai completions --model text-davinci-002 --prompt "Write a poem about the sea."
   ```

2. **Specify Maximum Tokens:**
   ```bash
   openai completions --model text-davinci-002 --prompt "Explain quantum physics." --max-tokens 150
   ```

### Images

The `images` command is used to generate or process images using OpenAI's capabilities.

#### Usage

```bash
openai images [options]
```

#### Flags and Arguments

| Flag         | Description                           | Example                                      |
|--------------|---------------------------------------|----------------------------------------------|
| `--prompt`   | Text prompt to generate an image      | `openai images --prompt "A futuristic cityscape"` |
| `--size`     | Specify image size (256x256, 512x512) | `openai images --size 512x512`               |

#### Examples

1. **Generate Image:**
   ```bash
   openai images --prompt "A dragon flying over mountains" --size 256x256
   ```

2. **Different Image Size:**
   ```bash
   openai images --prompt "A serene beach at sunset" --size 512x512
   ```

### Audio

The `audio` command processes audio inputs and provides capabilities like transcription.

#### Usage

```bash
openai audio [options]
```

#### Flags and Arguments

| Flag         | Description                           | Example                                      |
|--------------|---------------------------------------|----------------------------------------------|
| `--file`     | Path to the audio file                | `openai audio --file path/to/audio.mp3`      |
| `--language` | Language of the audio content         | `openai audio --language en`                 |

#### Examples

1. **Transcribe Audio:**
   ```bash
   openai audio --file path/to/audio.mp3 --language en
   ```

2. **Process Audio in Different Language:**
   ```bash
   openai audio --file path/to/audio.mp3 --language es
   ```

### Files

The `files` command is used to manage file uploads and downloads for OpenAI's APIs.

#### Usage

```bash
openai files [command] [options]
```

#### Subcommands

- **upload**: Upload a file to OpenAI.
- **list**: List all uploaded files.
- **delete**: Delete a specific file.

#### Flags and Arguments

| Subcommand | Flag        | Description                   | Example                                            |
|------------|-------------|-------------------------------|----------------------------------------------------|
| upload     | `--file`    | Path to the file to upload    | `openai files upload --file path/to/data.jsonl`    |
| delete     | `--file-id` | ID of the file to delete      | `openai files delete --file-id file-abc123`        |

#### Examples

1. **Upload a File:**
   ```bash
   openai files upload --file path/to/dataset.jsonl
   ```

2. **List Uploaded Files:**
   ```bash
   openai files list
   ```

3. **Delete a File:**
   ```bash
   openai files delete --file-id file-abc123
   ```

### Fine-tuning

The `fine-tuning` command is used to fine-tune existing models with custom datasets.

#### Usage

```bash
openai fine-tuning [options]
```

#### Flags and Arguments

| Flag           | Description                           | Example                                      |
|----------------|---------------------------------------|----------------------------------------------|
| `--model`      | Base model to fine-tune               | `openai fine-tuning --model text-davinci-002`|
| `--file`       | Path to the training dataset          | `openai fine-tuning --file path/to/data.jsonl`|

#### Examples

1. **Fine-tune a Model:**
   ```bash
   openai fine-tuning --model text-davinci-002 --file path/to/dataset.jsonl
   ```

2. **Fine-tune with Additional Parameters:**
   ```bash
   openai fine-tuning --model text-davinci-002 --file path/to/dataset.jsonl --epochs 5
   ```

## Troubleshooting

When using the OpenAI CLI, you may encounter errors or issues. This section provides guidance on troubleshooting common problems.

### Common Errors

1. **Authentication Errors:**
   - Ensure your API key is correctly set as an environment variable or configured directly in the CLI.
   - Double-check the API key for any typos or incorrect characters.

2. **Network Issues:**
   - Verify your internet connection.
   - Check if there are any restrictions or proxies blocking access to the OpenAI API endpoints.

3. **Invalid Command or Flags:**
   - Use the `--help` flag to get detailed information about available commands and flags.
   - Ensure the syntax and spelling of commands and flags are correct.

### Debugging Tips

- **Verbose Output:**
  - Use the `--verbose` flag to get detailed logs of the CLI operations, which can help in diagnosing issues.

- **Review Documentation:**
  - Refer to the official OpenAI CLI documentation for any updates or known issues.

## Conclusion

The OpenAI CLI is a versatile tool that empowers developers to interact with OpenAI's advanced AI models. This comprehensive reference guide provides detailed information on installation, authentication, commands, and troubleshooting. By understanding the capabilities and options available within the CLI, users can effectively integrate AI functionalities into their applications and workflows.