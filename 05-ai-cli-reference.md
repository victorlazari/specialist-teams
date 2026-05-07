# AI CLI Command Reference

## 1. Introduction
The `ai` Command Line Interface (CLI) is a powerful, unified tool designed to interact with various artificial intelligence models, manage AI workloads, orchestrate training pipelines, and deploy machine learning models directly from your terminal. This comprehensive reference guide details every command, flag, argument, and configuration option available in the `ai` CLI, providing extensive examples to facilitate seamless integration into your development and operational workflows.

## 2. Global Flags
Global flags can be applied to any `ai` command to modify its behavior, output format, or execution context.

- `--config, -c <path>`: Specify a custom path to the configuration file. Default is `~/.ai/config.yaml`.
- `--profile, -p <name>`: Use a specific configuration profile defined in your config file. Default is `default`.
- `--output, -o <format>`: Set the output format. Supported formats: `text`, `json`, `yaml`, `table`. Default is `text`.
- `--verbose, -v`: Enable verbose logging for detailed execution information.
- `--debug, -d`: Enable debug-level logging, including raw API requests and responses.
- `--quiet, -q`: Suppress all non-error output.
- `--region, -r <region>`: Specify the target region for cloud-based AI services (e.g., `us-east-1`, `eu-west-1`).
- `--help, -h`: Display help information for the current command or subcommand.
- `--version`: Display the current version of the `ai` CLI.

## 3. Core Commands

### 3.1 `ai init`
Initializes a new AI project in the current directory, creating necessary configuration files and directory structures.

**Usage:**
```bash
ai init [project-name] [flags]
```

**Arguments:**
- `project-name` (Optional): The name of the project. If omitted, the current directory name is used.

**Flags:**
- `--template, -t <template>`: Specify a project template (e.g., `nlp`, `vision`, `tabular`, `llm-finetune`). Default is `basic`.
- `--git`: Initialize a git repository in the project directory.
- `--force, -f`: Overwrite existing configuration files if they exist.

**Examples:**
```bash
# Initialize a basic project
ai init my-ai-project

# Initialize a vision project with git
ai init vision-app --template vision --git
```

### 3.2 `ai auth`
Manages authentication credentials for various AI service providers (e.g., OpenAI, Anthropic, Hugging Face, local clusters).

**Usage:**
```bash
ai auth <subcommand> [flags]
```

**Subcommands:**
- `login`: Authenticate with a provider.
- `logout`: Remove authentication credentials.
- `status`: Check current authentication status.

**Flags (for `login`):**
- `--provider <name>`: The AI service provider (e.g., `openai`, `huggingface`, `aws`).
- `--token <token>`: Provide the API token directly (not recommended for interactive use).
- `--sso`: Use Single Sign-On for enterprise providers.

**Examples:**
```bash
# Interactive login to OpenAI
ai auth login --provider openai

# Check authentication status
ai auth status
```

## 4. Model Management Commands

### 4.1 `ai model list`
Lists available models across configured providers or local storage.

**Usage:**
```bash
ai model list [flags]
```

**Flags:**
- `--provider <name>`: Filter models by provider.
- `--type <type>`: Filter by model type (e.g., `llm`, `embedding`, `image-generation`).
- `--local`: Only list models downloaded locally.
- `--remote`: Only list models available remotely.

**Examples:**
```bash
# List all local LLMs
ai model list --type llm --local

# List models from Hugging Face in JSON format
ai model list --provider huggingface -o json
```

### 4.2 `ai model pull`
Downloads a model from a remote registry to local storage for offline use or local inference.

**Usage:**
```bash
ai model pull <model-id> [flags]
```

**Arguments:**
- `model-id`: The identifier of the model (e.g., `meta-llama/Llama-2-7b-chat-hf`).

**Flags:**
- `--quantization, -q <level>`: Specify quantization level (e.g., `4bit`, `8bit`, `fp16`).
- `--dir <path>`: Custom directory to save the model. Default is `~/.ai/models/`.
- `--resume`: Resume an interrupted download.

**Examples:**
```bash
# Pull a model with 4-bit quantization
ai model pull mistralai/Mistral-7B-v0.1 --quantization 4bit
```

### 4.3 `ai model push`
Uploads a locally trained or fine-tuned model to a remote registry.

**Usage:**
```bash
ai model push <local-path> <remote-repo> [flags]
```

**Arguments:**
- `local-path`: Path to the local model directory.
- `remote-repo`: Destination repository (e.g., `username/my-custom-model`).

**Flags:**
- `--private`: Mark the repository as private.
- `--license <type>`: Specify the license (e.g., `mit`, `apache-2.0`).
- `--tags <tags>`: Comma-separated list of tags.

**Examples:**
```bash
ai model push ./output/checkpoint-500 myorg/custom-classifier --private --tags "vision,classification"
```

## 5. Inference Commands

### 5.1 `ai prompt`
Sends a prompt to an LLM and returns the response. Ideal for quick testing and scripting.

**Usage:**
```bash
ai prompt "your prompt text" [flags]
```

**Flags:**
- `--model, -m <model-id>`: The model to use.
- `--system <text>`: System prompt to set the behavior of the model.
- `--temperature, -t <float>`: Sampling temperature (0.0 to 2.0). Default is 0.7.
- `--max-tokens <int>`: Maximum number of tokens to generate.
- `--stream`: Stream the response to stdout as it's generated.
- `--file, -f <path>`: Read the prompt from a file instead of inline text.

**Examples:**
```bash
# Simple prompt
ai prompt "Explain quantum computing in one sentence." --model gpt-4

# Streaming response with system prompt
ai prompt "Write a Python script to parse JSON." --model claude-3-opus --system "You are an expert Python developer." --stream
```

### 5.2 `ai chat`
Starts an interactive chat session with an LLM in the terminal.

**Usage:**
```bash
ai chat [flags]
```

**Flags:**
- `--model, -m <model-id>`: The model to use.
- `--system <text>`: System prompt.
- `--history <path>`: Load and save chat history to a specific file.
- `--multiline`: Enable multiline input mode (press Ctrl+D to submit).

**Examples:**
```bash
ai chat --model local/llama-3-8b-instruct --multiline
```

### 5.3 `ai embed`
Generates embeddings for given text input.

**Usage:**
```bash
ai embed "text to embed" [flags]
```

**Flags:**
- `--model, -m <model-id>`: The embedding model to use.
- `--file, -f <path>`: Read input text from a file.
- `--batch`: Treat each line in the file as a separate input for batch processing.

**Examples:**
```bash
ai embed "Hello world" --model text-embedding-3-small -o json
```

## 6. Training and Fine-Tuning Commands

### 6.1 `ai train start`
Initiates a model training or fine-tuning job.

**Usage:**
```bash
ai train start [flags]
```

**Flags:**
- `--config <path>`: Path to the training configuration YAML file (required).
- `--dataset <path>`: Path to the training dataset.
- `--base-model <model-id>`: The base model to fine-tune.
- `--epochs <int>`: Number of training epochs.
- `--batch-size <int>`: Training batch size.
- `--learning-rate <float>`: Learning rate.
- `--gpu <count>`: Number of GPUs to allocate.
- `--cluster <name>`: Target compute cluster (for remote training).

**Examples:**
```bash
ai train start --config ./train_config.yaml --gpu 4
```

### 6.2 `ai train status`
Checks the status of an ongoing training job.

**Usage:**
```bash
ai train status <job-id> [flags]
```

**Flags:**
- `--watch, -w`: Continuously monitor the status (updates every 5 seconds).
- `--metrics`: Display detailed training metrics (loss, accuracy, etc.).

**Examples:**
```bash
ai train status job-12345abc --watch --metrics
```

### 6.3 `ai train logs`
Retrieves the logs for a specific training job.

**Usage:**
```bash
ai train logs <job-id> [flags]
```

**Flags:**
- `--tail, -t <lines>`: Number of lines to show from the end of the logs.
- `--follow, -f`: Follow log output in real-time.

**Examples:**
```bash
ai train logs job-12345abc --follow
```

### 6.4 `ai train stop`
Terminates an ongoing training job.

**Usage:**
```bash
ai train stop <job-id> [flags]
```

**Flags:**
- `--force`: Force termination without saving checkpoints.

**Examples:**
```bash
ai train stop job-12345abc
```

## 7. Data Management Commands

### 7.1 `ai data prepare`
Preprocesses and formats raw data into a suitable format for training or evaluation.

**Usage:**
```bash
ai data prepare <input-path> <output-path> [flags]
```

**Flags:**
- `--format <type>`: Target format (e.g., `jsonl`, `csv`, `parquet`, `huggingface`).
- `--task <type>`: The ML task (e.g., `text-classification`, `token-classification`, `seq2seq`).
- `--split <ratio>`: Train/validation/test split ratio (e.g., `80:10:10`).
- `--tokenize`: Apply tokenization during preparation.
- `--tokenizer <model-id>`: Specify the tokenizer to use.

**Examples:**
```bash
ai data prepare ./raw_data.csv ./processed_data --format jsonl --task seq2seq --split 90:10
```

### 7.2 `ai data validate`
Validates a dataset against a specific schema or model requirement.

**Usage:**
```bash
ai data validate <dataset-path> [flags]
```

**Flags:**
- `--schema <path>`: Path to JSON schema file.
- `--model <model-id>`: Validate against the requirements of a specific model.

**Examples:**
```bash
ai data validate ./processed_data/train.jsonl --model gpt-3.5-turbo-finetune
```

## 8. Deployment Commands

### 8.1 `ai deploy create`
Deploys a model as an API endpoint.

**Usage:**
```bash
ai deploy create <model-id> [flags]
```

**Flags:**
- `--name <string>`: Name of the deployment.
- `--env <environment>`: Target environment (e.g., `dev`, `staging`, `prod`).
- `--replicas <int>`: Number of instances to run.
- `--instance-type <type>`: Compute instance type (e.g., `g4dn.xlarge`).
- `--port <int>`: Port to expose the API. Default is 8080.
- `--auth-token <token>`: Require a specific token for API access.

**Examples:**
```bash
ai deploy create myorg/custom-classifier --name prod-classifier --env prod --replicas 3 --instance-type p4d.24xlarge
```

### 8.2 `ai deploy list`
Lists all active deployments.

**Usage:**
```bash
ai deploy list [flags]
```

**Flags:**
- `--env <environment>`: Filter by environment.

**Examples:**
```bash
ai deploy list --env prod
```

### 8.3 `ai deploy update`
Updates an existing deployment (e.g., scaling, changing model version).

**Usage:**
```bash
ai deploy update <deployment-name> [flags]
```

**Flags:**
- `--model <model-id>`: Update the deployed model version.
- `--replicas <int>`: Scale the number of replicas.
- `--traffic-split <percentages>`: Perform canary or A/B testing (e.g., `v1=90,v2=10`).

**Examples:**
```bash
ai deploy update prod-classifier --replicas 5
ai deploy update prod-classifier --model myorg/custom-classifier:v2 --traffic-split v1=80,v2=20
```

### 8.4 `ai deploy delete`
Tears down a deployment.

**Usage:**
```bash
ai deploy delete <deployment-name> [flags]
```

**Flags:**
- `--force, -f`: Skip confirmation prompt.

**Examples:**
```bash
ai deploy delete dev-classifier --force
```

## 9. Pipeline and Workflow Commands

### 9.1 `ai pipeline run`
Executes a multi-step AI workflow defined in a YAML file.

**Usage:**
```bash
ai pipeline run <pipeline-file> [flags]
```

**Flags:**
- `--param <key=value>`: Override pipeline parameters. Can be specified multiple times.
- `--step <step-name>`: Run only a specific step.
- `--dry-run`: Validate the pipeline without executing it.

**Examples:**
```bash
ai pipeline run ./rag_pipeline.yaml --param chunk_size=512 --param top_k=5
```

## 10. Agent Commands

### 10.1 `ai agent start`
Starts an autonomous AI agent with specific tools and goals.

**Usage:**
```bash
ai agent start [flags]
```

**Flags:**
- `--name <string>`: Name of the agent.
- `--goal <text>`: The primary objective for the agent.
- `--tools <list>`: Comma-separated list of tools the agent can use (e.g., `web_search,file_system,python_repl`).
- `--model <model-id>`: The underlying LLM powering the agent.
- `--max-iterations <int>`: Maximum number of steps the agent can take.

**Examples:**
```bash
ai agent start --name "ResearchBot" --goal "Summarize recent papers on Q-learning" --tools "web_search,arxiv_api,file_write" --model gpt-4-turbo
```

## 11. Configuration and Environment

### 11.1 `ai config set`
Sets a configuration value.

**Usage:**
```bash
ai config set <key> <value> [flags]
```

**Examples:**
```bash
ai config set default_model claude-3-sonnet
ai config set output_format json
```

### 11.2 `ai config get`
Retrieves a configuration value.

**Usage:**
```bash
ai config get <key>
```

**Examples:**
```bash
ai config get default_model
```

## 12. Advanced Usage and Scripting

The `ai` CLI is designed to be highly composable in shell scripts. By utilizing the `-o json` flag and tools like `jq`, you can build complex automation.

**Example: Automated Model Evaluation Script**
```bash
#!/bin/bash

MODELS=("model-a" "model-b" "model-c")
DATASET="./eval_data.jsonl"

for MODEL in "${MODELS[@]}"; do
  echo "Evaluating $MODEL..."
  
  # Run evaluation job and capture job ID
  JOB_ID=$(ai train start --config eval.yaml --base-model $MODEL --dataset $DATASET -o json | jq -r '.job_id')
  
  # Wait for completion
  while true; do
    STATUS=$(ai train status $JOB_ID -o json | jq -r '.status')
    if [ "$STATUS" == "COMPLETED" ]; then
      break
    elif [ "$STATUS" == "FAILED" ]; then
      echo "Evaluation failed for $MODEL"
      exit 1
    fi
    sleep 10
  done
  
  # Fetch and save metrics
  ai train status $JOB_ID --metrics -o json > "metrics_${MODEL}.json"
  echo "Metrics saved for $MODEL"
done
```

## 13. Troubleshooting

- **Authentication Errors:** Ensure your API keys are valid and not expired. Run `ai auth status` to verify.
- **Out of Memory (OOM):** When pulling or running local models, ensure your system has sufficient RAM/VRAM. Use the `--quantization` flag to reduce memory footprint.
- **Network Timeouts:** For large model downloads, use the `--resume` flag if the connection drops.
- **Dependency Issues:** If using local execution environments, ensure Python and required CUDA drivers are correctly installed and mapped in your `~/.ai/config.yaml`.

## 14. Conclusion
The `ai` CLI provides a robust, unified interface for the entire AI lifecycle, from data preparation and model training to deployment and inference. By mastering these commands, developers and MLOps engineers can significantly accelerate their AI workflows and maintain tight control over their infrastructure.