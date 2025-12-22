# Rogue Evaluation Harness - Strategy Support Classification

This document classifies which evaluation strategies from the comprehensive evaluation framework taxonomy are supported by Rogue.

**Classification Legend:**
- **Supported (Native):** Available immediately after harness installation with only imports and minimal config (≤2 lines). No external dependencies or glue code.
- **Supported (Integrated):** Requires external package(s) and glue code but has documented integration patterns or official examples.
- **Not Supported:** Neither native nor integrated support available.

---

## Stage 0: Provisioning (The Runtime)

### Step A: Harness Installation

**Strategy 1: Git Clone**
- **Classification:** Supported (Native)
- **Evidence:** README.md shows `git clone https://github.com/qualifire-dev/rogue.git` followed by `uv sync` or `pip install -e .`
- **Details:** Standard installation method documented in README

**Strategy 2: PyPI Packages**
- **Classification:** Supported (Native)
- **Evidence:** `uvx rogue-ai` installs directly from PyPI without needing to clone the repository
- **Details:** Package published as `rogue-ai` on PyPI, can be installed with pip/uvx

**Strategy 3: Node Package**
- **Classification:** Not Supported
- **Evidence:** Rogue is a Python-based harness. While there is a TypeScript SDK in `packages/sdk/`, the core harness is not installable via npm
- **Details:** N/A

**Strategy 4: Binary Packages**
- **Classification:** Supported (Integrated)
- **Evidence:** The TUI component is a standalone Go binary that can be downloaded and run independently (see `packages/tui/`)
- **Details:** The TUI installer (`rogue/common/tui_installer.py`) downloads and installs the Go-based TUI binary

**Strategy 5: Container Images**
- **Classification:** Not Supported
- **Evidence:** No Docker/OCI container images are provided in the repository or documentation
- **Details:** N/A

### Step B: Credential Configuration

**Strategy 1: Model API Authentication**
- **Classification:** Supported (Native)
- **Evidence:** 
  - `.env.example` shows support for `OPENAI_API_KEY`, `ANTHROPIC_API_KEY`, `GOOGLE_API_KEY`
  - Uses LiteLLM which supports multiple model providers
  - CLI accepts `--judge-llm-api-key` parameter
- **Details:** Configured via environment variables or CLI parameters for remote inference to commercial providers

**Strategy 2: Repository Authentication**
- **Classification:** Supported (Integrated)
- **Evidence:** 
  - `prompt_injection_evaluator/run_prompt_injection_evaluator.py` uses `datasets` library from HuggingFace
  - Can load datasets via `load_dataset()` which supports HuggingFace Hub authentication
- **Details:** Uses HuggingFace `datasets` library which handles repository authentication for gated datasets

**Strategy 3: Evaluation Platform Authentication**
- **Classification:** Supported (Integrated)
- **Evidence:** 
  - Code references `QUALIFIRE_API_KEY` in `run_cli.py`
  - `rogue/server/services/qualifire_service.py` exists for platform integration
- **Details:** Supports authentication with Qualifire evaluation platform via API key

---

## Stage I: Specification (The Contract)

### Step A: SUT Preparation

**Strategy 1: Model-as-a-Service (Remote Inference)**
- **Classification:** Supported (Native)
- **Evidence:**
  - Primary use case: evaluates remote agents via HTTP endpoints
  - `AgentConfig.evaluated_agent_url` accepts remote URLs
  - Supports A2A protocol over HTTP (see `evaluator_agent/a2a/`)
- **Details:** Designed to test remote agents exposed via A2A or MCP protocols

**Strategy 2: Model-in-Process (Local Inference)**
- **Classification:** Not Supported
- **Evidence:** Rogue evaluates external agents via network protocols, not local model weights
- **Details:** The evaluated agent must be running as a separate service; Rogue does not load model weights

**Strategy 3: Non-Parametric Algorithms (Deterministic Computation)**
- **Classification:** Not Supported
- **Evidence:** Rogue is designed for conversational AI agents, not deterministic algorithms
- **Details:** N/A

**Strategy 4: Interactive Agents (Sequential Decision-Making)**
- **Classification:** Supported (Native)
- **Evidence:**
  - `base_evaluator_agent.py` supports multi-turn conversations via `context_id`
  - Deep test mode enables up to 5-message conversations per scenario
  - Maintains conversation state via `_context_id_to_chat_history`
- **Details:** Evaluator agent can conduct multi-turn conversations with stateful agents

### Step B: Benchmark Preparation (Inputs)

**Strategy 1: Benchmark Data Preparation (Offline)**
- **Classification:** Supported (Native)
- **Evidence:**
  - `Scenarios` model accepts predefined test scenarios
  - Can load scenarios from JSON files (`--input-scenarios-file`)
  - Prompt injection evaluator loads HuggingFace datasets (`load_dataset()`)
- **Details:** Supports both manually defined scenarios and pre-existing benchmark datasets

**Strategy 2: Synthetic Data Generation (Generative)**
- **Classification:** Supported (Native)
- **Evidence:**
  - `llm_service.py` contains `SCENARIO_GENERATION_SYSTEM_PROMPT` for generating test scenarios
  - LLM service creates 10-15 scenarios from business context automatically
  - Interview mode generates scenarios interactively
- **Details:** Uses LLM to generate test scenarios from high-level business context descriptions

**Strategy 3: Simulation Environment Setup (Simulated)**
- **Classification:** Not Supported
- **Evidence:** Rogue tests conversational agents, not simulated environments or robotics
- **Details:** N/A

**Strategy 4: Production Traffic Sampling (Online)**
- **Classification:** Not Supported
- **Evidence:** No evidence of real-time production traffic sampling in codebase
- **Details:** N/A

### Step C: Benchmark Preparation (References)

**Strategy 1: Ground Truth Preparation**
- **Classification:** Supported (Integrated)
- **Evidence:**
  - `Scenario.expected_outcome` field stores expected outcomes
  - Policy evaluation uses expected outcomes as ground truth
  - HuggingFace datasets include labels (e.g., `x["label"] == "jailbreak"`)
- **Details:** Supports manual expected outcomes and dataset-provided labels

**Strategy 2: Judge Preparation**
- **Classification:** Supported (Native)
- **Evidence:**
  - Uses LLM-as-a-judge pattern throughout (see `policy_evaluation.py`)
  - `judge_llm` configuration specifies the evaluation model
  - No fine-tuning of specialized judges, uses pre-trained models directly
- **Details:** Uses pre-trained LLMs as judges without fine-tuning

---

## Stage II: Execution (The Run)

### Step A: SUT Invocation

**Strategy 1: Batch Inference**
- **Classification:** Supported (Native)
- **Evidence:**
  - Evaluates multiple scenarios sequentially via `scenario_evaluation_service.py`
  - `scenario_runner.py` has `split_into_batches()` function
  - Supports parallel execution with `parallel_runs` configuration
- **Details:** Runs scenarios in batches, can parallelize with multiple workers

**Strategy 2: Arena Battle**
- **Classification:** Not Supported
- **Evidence:** No code for simultaneous evaluation of multiple SUTs on the same input
- **Details:** Evaluates one agent at a time, not side-by-side comparisons

**Strategy 3: Interactive Loop**
- **Classification:** Supported (Native)
- **Evidence:**
  - Multi-turn conversations maintained via `context_id`
  - `_send_message_to_evaluated_agent()` supports iterative message exchange
  - Deep test mode enables complex conversation flows (up to 5 messages)
- **Details:** Evaluator agent conducts stateful multi-turn conversations

**Strategy 4: Production Streaming**
- **Classification:** Not Supported
- **Evidence:** No real-time production traffic processing capabilities
- **Details:** N/A

---

## Stage III: Assessment (The Score)

### Step A: Individual Scoring

**Strategy 1: Deterministic Measurement**
- **Classification:** Not Supported
- **Evidence:** Rogue relies on LLM-as-a-judge, not deterministic metrics like BLEU/ROUGE
- **Details:** No algorithmic metrics like edit distance or token-based text metrics

**Strategy 2: Embedding Measurement**
- **Classification:** Not Supported
- **Evidence:** No semantic similarity or embedding-based scoring in codebase
- **Details:** N/A

**Strategy 3: Subjective Measurement**
- **Classification:** Supported (Native)
- **Evidence:**
  - `policy_evaluation.py` uses LLM to judge policy compliance
  - `_judge_injection_attempt()` evaluates prompt injection resistance
  - Uses `POLICY_EVALUATION_PROMPT` to assess conversation quality
- **Details:** Primary scoring method is LLM-as-a-judge for subjective assessment

**Strategy 4: Performance Measurement**
- **Classification:** Not Supported
- **Evidence:** No latency, throughput, or resource consumption tracking
- **Details:** Focuses on behavioral correctness, not performance metrics

### Step B: Aggregate Scoring

**Strategy 1: Distributional Statistics**
- **Classification:** Supported (Integrated)
- **Evidence:**
  - `EvaluationResults` aggregates per-scenario results
  - UI and reports show pass/fail counts and percentages
  - Report generator computes summary statistics
- **Details:** Calculates basic statistics like pass rates across scenarios

**Strategy 2: Uncertainty Quantification**
- **Classification:** Not Supported
- **Evidence:** No bootstrap resampling or confidence intervals in results
- **Details:** Reports point estimates only, no statistical confidence bounds

---

## Stage IV: Reporting (The Output)

### Step A: Insight Presentation

**Strategy 1: Execution Tracing**
- **Classification:** Supported (Native)
- **Evidence:**
  - Chat history captured in `ChatHistory` and `ConversationEvaluation`
  - Live chat updates via `_chat_update_callback`
  - TUI and Web UI display conversation flows in real-time
  - Full conversation logs stored in evaluation results
- **Details:** Captures and displays detailed message-by-message execution traces

**Strategy 2: Subgroup Analysis**
- **Classification:** Supported (Integrated)
- **Evidence:**
  - Scenarios are typed (`ScenarioType.POLICY`, `ScenarioType.PROMPT_INJECTION`)
  - Can filter results by scenario type via `get_scenarios_by_type()`
  - Results can be stratified by scenario type
- **Details:** Supports basic stratification by scenario type

**Strategy 3: Regression Alerting**
- **Classification:** Not Supported
- **Evidence:** No historical baseline comparison or automated regression detection
- **Details:** N/A

**Strategy 4: Chart Generation**
- **Classification:** Not Supported
- **Evidence:** No chart/plot generation in codebase (only text-based reports)
- **Details:** Reports are markdown text, not visual charts

**Strategy 5: Dashboard Creation**
- **Classification:** Supported (Native)
- **Evidence:**
  - TUI provides interactive dashboard (`packages/tui/internal/screens/dashboard/`)
  - Web UI (Gradio) shows configuration, scenarios, evaluation progress, and results
  - Live updates during evaluation runs
- **Details:** Multiple interactive interfaces (TUI and Web UI) for monitoring and results

**Strategy 6: Leaderboard Publication**
- **Classification:** Supported (Integrated)
- **Evidence:**
  - Integration with Qualifire platform (`qualifire_service.py`)
  - `QUALIFIRE_API_KEY` configuration for platform submission
- **Details:** Can submit results to Qualifire evaluation platform/leaderboard

---

## Summary Statistics

**Total Strategies Analyzed:** 34

**Native Support:** 13
- Installation: Git Clone, PyPI Packages
- Credentials: Model API Authentication
- SUT: Model-as-a-Service, Interactive Agents
- Inputs: Benchmark Data, Synthetic Generation
- References: Judge Preparation (pre-trained)
- Execution: Batch Inference, Interactive Loop
- Scoring: Subjective Measurement
- Reporting: Execution Tracing, Dashboard Creation

**Integrated Support:** 7
- Installation: Binary Packages
- Credentials: Repository Authentication, Platform Authentication
- References: Ground Truth Preparation
- Aggregation: Distributional Statistics
- Reporting: Subgroup Analysis, Leaderboard Publication

**Not Supported:** 14
- Installation: Node Package, Container Images
- SUT: Local Inference, Non-Parametric Algorithms
- Inputs: Simulation Environment, Production Traffic
- Execution: Arena Battle, Production Streaming
- Scoring: Deterministic Measurement, Embedding Measurement, Performance Measurement
- Aggregation: Uncertainty Quantification
- Reporting: Regression Alerting, Chart Generation

---

## Key Findings

### Rogue's Core Strengths:
1. **Conversational Agent Testing:** Designed specifically for evaluating LLM-based conversational agents
2. **LLM-as-a-Judge:** Primary evaluation methodology using subjective LLM assessment
3. **Multi-Protocol Support:** Native A2A and MCP protocol support for agent communication
4. **Scenario Generation:** Automated test scenario generation from business context
5. **Interactive Testing:** Multi-turn conversational evaluation capabilities
6. **Real-time Monitoring:** Live dashboards and execution tracing

### Rogue's Limitations:
1. **No Local Model Support:** Cannot evaluate models loaded locally; requires remote agent endpoints
2. **Limited Metric Diversity:** No deterministic metrics (BLEU, ROUGE) or embedding-based scoring
3. **No Performance Metrics:** Does not measure latency, throughput, or resource usage
4. **Single-Agent Focus:** No comparative evaluation (arena battles) or A/B testing
5. **No Statistical Rigor:** No confidence intervals or uncertainty quantification
6. **Limited Visualization:** Text-based reports without charts or plots

### Best Use Cases for Rogue:
- Policy compliance testing for conversational AI agents
- Prompt injection resistance evaluation
- Multi-turn conversation capability assessment
- Edge case discovery through synthetic scenario generation
- Real-time evaluation monitoring and debugging

### Not Suitable For:
- Benchmarking model performance (speed, efficiency)
- Comparing multiple models side-by-side
- Evaluating non-conversational AI systems
- Statistical significance testing
- Production monitoring and regression detection
