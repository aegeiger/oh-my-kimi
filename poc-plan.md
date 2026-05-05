# PoC Plan: oh-my-kimi

## Project Classification
- **Type:** infrastructure (CLI tool / developer tooling)
- **Key Technologies:** TypeScript, Node.js 20+, Commander, Zod, node-pty, npm
- **ODH Relevance:** Low direct relevance. Multi-agent CLI orchestration tool for Kimi Code. Demonstrates containerizing a Node.js CLI tool on OpenShift.

## PoC Objectives
1. Build the TypeScript CLI tool in a UBI-based container
2. Validate that the compiled CLI starts and displays help/version
3. Run the built-in test suite (253 tests) inside the container
4. Validate `omk doctor --soft` runs without hard failure

## Infrastructure Requirements
- **Deployment Model:** job
- **Listens on Port:** no
- **Resource Profile:** small (256Mi RAM, 250m CPU)
- **Inference Server:** none
- **Vector Database:** none
- **GPU Required:** no
- **Persistent Storage:** none
- **Sidecar Containers:** none

## Test Scenarios

### Scenario 1: Version Check
- **Description:** Verify the CLI binary is built and reports its version
- **Type:** cli (Job)
- **Input:** `omk --version`
- **Expected:** Exit 0, output contains version number (1.1.1)
- **Timeout:** 30

### Scenario 2: Help Output
- **Description:** Verify the CLI displays help text with available commands
- **Type:** cli (Job)
- **Input:** `omk --help`
- **Expected:** Exit 0, output contains "omk" and command descriptions
- **Timeout:** 30

### Scenario 3: Doctor Soft Mode
- **Description:** Run omk doctor in soft mode (won't fail on missing Kimi CLI)
- **Type:** cli (Job)
- **Input:** `omk doctor --soft`
- **Expected:** Exit 0, output shows check results
- **Timeout:** 60

### Scenario 4: Test Suite
- **Description:** Run the project's 253 built-in tests
- **Type:** cli (Job)
- **Input:** `npm test`
- **Expected:** Exit 0, all tests pass
- **Timeout:** 120

## Dockerfile Considerations
- Base image: `registry.access.redhat.com/ubi9/nodejs-22` (Node.js 20+ required)
- Install system deps: python3, make, gcc, gcc-c++ (for node-pty native compilation)
- Install git (required by the tool for worktree operations)
- npm install + npm run build to compile TypeScript
- No EXPOSE (CLI tool, no port)
- ENTRYPOINT: ["node", "dist/cli.js"], CMD: ["--help"]
- USER 1001 for OpenShift

## Deployment Considerations
- Deploy as Jobs only (CLI tool - NOT a Deployment, would CrashLoopBackOff)
- No Service needed (no port)
- One Job per test scenario
- Small resource profile sufficient
- imagePullPolicy: Always (Quay)
