# Multi-Agent LLM Debate for Benchmark Evaluation

This repository contains a Python implementation of a multi-agent debate system designed to evaluate the reasoning and knowledge of Large Language Models (LLMs) on standard benchmarks.

Inspired by the paper **"Improving Factuality and Reasoning in Language Models through Multiagent Debate" (Du et al., 2023, arXiv:2305.14325)**, this project simulates a structured debate between two LLM agents who critique and refine their answers over multiple rounds.

## Evaluation Setup

### Tasks Evaluated:
1.  **GSM8K (Grade School Math)**
    * **Source:** `openai/gsm8k` (via Hugging Face `datasets`)
    * **Task:** Solving multi-step mathematical word problems (2-8 reasoning steps).
    * **Our Testing:** Evaluated on 100+ samples from the test split, checking for the correct final numerical answer (e.g., `\boxed{number}`).
2.  **MMLU (Massive Multitask Language Understanding)**
    * **Source:** `cais/mmlu` (via Hugging Face `datasets`, specific subject tested, e.g., `high_school_computer_science`)
    * **Task:** Answering multiple-choice questions across diverse academic subjects.
    * **Format:** Question + four options (A, B, C, D).
    * **Our Testing:** Evaluated on 100+ samples from the test split, checking for the correct final **letter choice** (A, B, C, or D, e.g., `\boxed{LETTER}`).

### Agent Setup:
* **Agents:** 2 Agents (State-of-the-Art models) accessed via API Inference:
    * Agent 1: Google Gemini 2.5 Pro (Preview)
    * Agent 2: OpenAI GPT-4-Turbo
* **API Access:** Python `subprocess` + `curl` used for direct API interaction. *(Note: Official SDKs recommended for production).*

## Methodology

1.  **Single-Agent Baseline:**
    * Established baseline accuracy for Gemini 2.5 Pro using zero-shot Chain-of-Thought prompting on both GSM8K and MMLU tasks.

2.  **Multi-Agent Debate:**
    * **Round 0:** Agents independently generated initial responses (Reasoning + `\boxed{ANSWER_FORMAT}`).
    * **Round 1+ (Debate):** Agents received prior responses (self + other). A debate prompt requested critique, refinement, and an updated final answer in the specified boxed format.
    * **Optimization:** Implemented early stopping – the debate terminated if agents converged on the same extracted answer (`\boxed{number}` or `\boxed{LETTER}`) in any round.
    * **Aggregation:**
        * Checked for convergence (identical extracted answers from both agents).
        * If converged, used the common answer.
        * If no convergence after the final round, **defaulted to Agent 1's (Gemini) extracted answer** (if available).
    * **Metrics:** Calculated **Accuracy** (comparing the final aggregated answer to ground truth) and **Convergence Rate** (% of debates ending with identical answers).

## Results Summary

The following results were observed over 100+ samples for each dataset using the specified models and methodology:

| Model Approach                       | GSM8K Accuracy | MMLU Accuracy (Computer Science) |
| :----------------------------------- | :------------- | :------------------------------- |
| Single Agent (Gemini 2.5 Pro)        | 98%            | 97%                              |
| Multi-Agent Debate (Gemini + GPT-4T) | 93%            | 98%                              |

| Metric (Multi-Agent)         | GSM8K          | MMLU (Computer Science)          |
| :--------------------------- | :------------- | :------------------------------- |
| Convergence Rate             | 94%            | 96%                              |

*(Note: These results reflect performance with the specific prompts, models, and configurations used in this implementation.)*

## Setup and Usage

1.  **Clone Repository:** `git clone <your-repo-url>`
2.  **Install Dependencies:** `pip install datasets tqdm` (Ensure `curl` is available).
3.  **Configure API Keys:**
    * **CRITICAL:** Obtain API keys for Google AI Studio (Gemini) and OpenAI. **Use NEW, SECURE keys.**
    * Store them securely. If using Colab: Use Secrets Manager (Key icon 🔑) with names `GEMINI_API_KEY` and `OPENAI_API_KEY`, ensuring "Notebook access" is ON. If running locally: Set environment variables. **DO NOT HARDCODE KEYS IN THE SCRIPT.**
4.  **Customize Script:**
    * Choose the desired `MMLU_SUBJECT` inside the script.
    * Set the `SAMPLE_SIZE`.
    * Verify model names in `model_config`.
5.  **Run:** Execute the Python script or notebook cells.

## Status

Functional prototype demonstrating multi-agent debate evaluation on GSM8K and MMLU using direct API calls.
