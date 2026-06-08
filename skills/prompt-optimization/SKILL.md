---
name: prompt-optimization
description: 'Automatically generate, evaluate, and rank system prompts for task-specific LLM agents. Use when building a prompt optimization pipeline, benchmarking prompt variants, scoring prompt quality against test cases, or selecting the best prompt across models. Trigger on "optimize prompt", "benchmark prompts", "rank prompts", "prompt evaluation", "prompt scoring", "generate prompt candidates".'
argument-hint: 'Provide a task description and test cases to optimize against'
---

# Prompt Optimization

Automated pipeline for generating diverse prompt candidates, evaluating them against test cases across models, and returning the highest-performing prompt. Designed for agent-driven workflows — no human-in-the-loop required.

Uses drafting strategies from the [prompt-engineering](../prompt-engineering/SKILL.md) skill as the generation grammar.

---

## When to Use

- Building an automated prompt optimization agent
- Benchmarking multiple prompt variants against test cases
- Selecting the best system prompt for a task-specific LLM agent
- Comparing prompt performance across different models
- Generating meaningfully diverse prompt candidates

---

## Inputs

The pipeline requires three inputs:

| Input | Required | Description |
|---|---|---|
| **Task description** | Yes | What the target agent should do — its purpose, domain, expected behavior |
| **Test cases** | Yes | Input→expected output pairs (or input→evaluation criteria) to score against |
| **Model list** | No | Models to test against (defaults to the primary model if omitted) |

### Task Description Schema

Extract or require these fields from the task description:

```
goal:        What the agent should accomplish (single sentence)
domain:      Subject area (e.g., "customer support", "code review", "data analysis")
audience:    Who will interact with the agent
input_type:  What the agent receives (e.g., "user question", "code snippet", "document")
output_type: What the agent produces (e.g., "answer", "review comments", "JSON report")
constraints: Hard requirements (tone, length, safety, format, forbidden topics)
```

### Test Case Schema

Each test case must include:

```
id:          Unique identifier
input:       The user message or content the agent will receive
expected:    One of:
               - exact:    Exact expected output (for deterministic tasks)
               - contains: Required keywords or phrases in the output
               - criteria: Natural language description of what a good output looks like
               - rubric:   Structured scoring dimensions with weights
tags:        Optional labels for categorization (e.g., "edge-case", "happy-path")
```

---

## Procedure

### Step 1: Parse and Validate Inputs

1. Extract structured fields from the task description (goal, domain, audience, input_type, output_type, constraints).
2. Validate that test cases have inputs and at least one evaluation method (exact, contains, criteria, or rubric).
3. Flag any ambiguities or missing fields — resolve them before proceeding to generation.

### Step 2: Generate N Diverse Candidates

Produce N candidate prompts (recommended: 5–10) that are **meaningfully different** from each other. Diversity is critical — near-duplicate candidates waste evaluation budget.

#### Variation Axes

Generate candidates by varying across these dimensions:

| Axis | Variations | Example |
|---|---|---|
| **Technique** | Zero-shot, few-shot (1/3/5 examples), CoT, zero-shot CoT | One candidate with examples, another without |
| **Persona** | Domain expert, generalist, audience peer, strict validator | "You are a senior security engineer" vs. "You are a helpful assistant" |
| **Structure** | Prose instructions, numbered steps, bullet constraints, template with placeholders | Free-form vs. rigid format |
| **Specificity** | Minimal (high model autonomy) vs. detailed (heavily constrained) | 3-line prompt vs. 30-line prompt with edge case handling |
| **Constraint ordering** | Rules first → task, task first → rules, interleaved | Affects how models weight instructions |
| **Output framing** | Direct answer, structured format (JSON/XML), think-then-answer | "Return a JSON object" vs. "Explain your reasoning, then provide the answer" |

#### Generation Rules

1. Every candidate MUST address the goal, audience, and output_type from the task description.
2. At least one candidate should be **minimal** (shortest viable prompt).
3. At least one candidate should be **maximal** (most detailed, heavily constrained).
4. At least one candidate should use **few-shot examples** drawn from the test cases (hold out those test cases from evaluation).
5. No two candidates should share the same combination of technique + persona + structure.
6. Apply the drafting strategies from the [prompt-engineering](../prompt-engineering/SKILL.md) skill (Steps 3a–3f) as the construction grammar for each candidate.

#### Candidate Schema

```
candidate_id:    Sequential identifier (1..N)
prompt_text:     The full system prompt
technique:       Which prompting technique it uses
persona:         Assigned persona (or "none")
structure:       Structure style (prose / numbered / bullets / template)
specificity:     minimal / moderate / detailed
notes:           Brief description of what makes this candidate distinct
```

### Step 3: Execute Test Matrix

Run every candidate against every test case on every model.

```
Total runs = N_candidates × N_test_cases × N_models
```

For each run, capture:

| Field | Description |
|---|---|
| `candidate_id` | Which prompt was used |
| `test_case_id` | Which test case was run |
| `model` | Which model executed it |
| `output` | Raw model output |
| `latency_ms` | Response time |
| `token_count` | Output token usage |

#### Execution Guidelines

- Use identical generation parameters (temperature, max tokens) across all runs for fair comparison.
- For non-deterministic evaluation, run each (candidate, test_case, model) tuple **3 times** and average the scores.
- If a candidate uses few-shot examples from test cases, **exclude those test cases** from its evaluation set.

### Step 4: Score Results

Score each run on these dimensions:

| Dimension | Weight | Scoring Method |
|---|---|---|
| **Correctness** | 0.40 | Does the output match the expected result? (exact match, keyword containment, or criteria satisfaction) |
| **Format compliance** | 0.20 | Does the output follow the required format? (structure, length, schema) |
| **Consistency** | 0.15 | Across repeated runs with the same input, how stable are the outputs? (semantic similarity across 3 runs) |
| **Completeness** | 0.15 | Does the output address all parts of the input? (no partial answers, no skipped requirements) |
| **Token efficiency** | 0.10 | Output achieves the goal with minimal token usage (penalize verbose outputs that add no value) |

Weights are defaults — adjust based on task priorities (e.g., for safety-critical tasks, increase correctness to 0.60).

#### Scoring Scale

Each dimension scores 0.0–1.0:

| Score | Meaning |
|---|---|
| 1.0 | Perfect — fully meets the criterion |
| 0.7–0.9 | Good — minor issues that don't materially affect quality |
| 0.4–0.6 | Partial — meets some aspects but has notable gaps |
| 0.1–0.3 | Poor — mostly fails the criterion |
| 0.0 | Fail — completely misses |

#### Per-Run Score

```
run_score = Σ (dimension_weight × dimension_score)
```

### Step 5: Aggregate and Rank

#### Per-Candidate Score

For each candidate, aggregate across all test cases and models:

```
candidate_score = mean(run_scores across all test_cases and models)
```

#### Rank Candidates

1. Sort by `candidate_score` descending.
2. For tied scores, prefer the candidate with:
   - Higher minimum test case score (fewer failures)
   - Better consistency score
   - Lower token usage

#### Cross-Model Analysis

If testing across multiple models, also compute:

```
model_variance = variance(per_model_average_scores)
```

Flag candidates with high model variance — they are model-dependent and may not generalize.

### Step 6: Return Results

Return a structured report:

```
winner:
  candidate_id:    [id]
  prompt_text:     [full prompt]
  overall_score:   [0.0–1.0]
  per_dimension:   {correctness: x, format: x, consistency: x, completeness: x, efficiency: x}

runner_up:
  candidate_id:    [id]
  prompt_text:     [full prompt]
  overall_score:   [0.0–1.0]

rankings:          [ordered list of all candidates with scores]

analysis:
  best_technique:  [which prompting technique scored highest]
  best_persona:    [which persona scored highest]
  model_variance:  [high/low — does the winner generalize across models?]
  failure_cases:   [test case IDs where even the winner scored < 0.5]
  recommendations: [suggested improvements based on failure patterns]
```

---

## Failure Pattern → Fix Mapping

When the winning prompt still has weak test cases, diagnose with:

| Failure Pattern | Diagnosis | Candidate Adjustment |
|---|---|---|
| Fails on edge cases but passes happy path | Prompt doesn't handle exceptions | Add explicit edge case instructions or constraints |
| Format compliance is low across all candidates | Task requires strict output schema | Add output schema/template in the prompt with an example |
| High model variance | Prompt relies on model-specific behavior | Simplify prompt, use more explicit instructions, add few-shot examples |
| Inconsistent outputs (low consistency) | Prompt is ambiguous | Remove ambiguous wording, add constraints, increase specificity |
| Correctness is high but efficiency is low | Prompt encourages verbosity | Add length constraints, use "be concise" directive |
| All candidates score poorly | Task description may be flawed, or the task exceeds model capability | Revisit task description, break into sub-tasks, or reconsider model choice |

---

## Anti-Patterns

| Anti-Pattern | Why It Fails |
|---|---|
| Generating N copies with minor word changes | Wastes evaluation budget — no meaningful diversity |
| Testing with < 5 test cases | Insufficient coverage — winner may not generalize |
| Using test case examples as few-shot AND evaluation | Inflated scores — the model is being tested on examples it was given |
| Same temperature/settings for generation and evaluation | Fair comparison requires controlled parameters |
| Optimizing for a single model then deploying on another | Prompt performance doesn't always transfer across models |
| Skipping the minimal candidate | Simple prompts often outperform complex ones — always include a baseline |
