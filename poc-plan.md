# PoC Plan: oh-my-kimi

## Project Classification
- **Type:** llm-app
- **Key Technologies:** TypeScript, Node.js, Kimi Code CLI, Multi-agent orchestration
- **ODH Relevance:** Provides LLM-powered code generation workflow orchestration, aligns with ODH's workbenches and model-serving capabilities

## PoC Objectives
What we want to prove:
1. The CLI tool can be containerized and run in Kubernetes
2. Core commands (init, doctor, --help) execute successfully
3. The multi-agent orchestration harness initializes and validates system requirements

## Infrastructure Requirements
- **Inference Server:** none
- **Vector Database:** none
- **Embedding Model:** none
- **GPU Required:** no
- **Persistent Storage:** none
- **Resource Profile:** small
- **Sidecar Containers:** none

## Test Scenarios
### Scenario 1: init-command
- **Description:** Verify the init command initializes a new project structure
- **Type:** cli
- **Input:** `omk init`
- **Expected:** Job exits 0, creates `.omk` directory with config files
- **Timeout:** 30

### Scenario 2: doctor-command
- **Description:** Verify the doctor command checks system requirements
- **Type:** cli
- **Input:** `omk doctor`
- **Expected:** Job exits 0, outputs system check summary
- **Timeout:** 15

### Scenario 3: help-output
- **Description:** Verify the CLI shows help with available commands
- **Type:** cli
- **Input:** `omk --help`
- **Expected:** Job exits 0, outputs usage info listing available subcommands
- **Timeout:** 10

## Dockerfile Considerations
This is a CLI tool. ENTRYPOINT should be `node dist/cli.js`. CMD should default to `--help`. Do NOT add EXPOSE — there is no port to expose. The container needs Node.js runtime and installed dependencies from package.json. The Dockerfile should:
1. Use a Node.js base image
2. Copy package.json and install dependencies
3. Copy the compiled TypeScript files from dist/
4. Set ENTRYPOINT to run the CLI binary

## Deployment Considerations
Do NOT deploy as a Deployment — the process exits immediately. Do NOT create a Service — there is no port. Test via kubectl run --rm. Each test scenario should be executed as a separate Kubernetes Job with the appropriate CLI command. Success is determined by exit code 0 and expected log output.