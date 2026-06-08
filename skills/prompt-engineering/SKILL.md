---
name: prompt-engineering
description: 'Craft, review, and improve prompts for LLMs. Use when writing, refining, evaluating, or debugging AI prompts. Trigger on "write a prompt", "improve this prompt", "prompt review", "prompt template", or any request to optimize instructions for an AI model.'
argument-hint: 'Describe the task or paste a draft prompt to improve'
---

# Prompt Engineering

Apply structured best practices to craft effective LLM prompts. Based on [Google Cloud prompt engineering guide](https://cloud.google.com/discover/what-is-prompt-engineering) and [Five best practices for prompt engineering](https://cloud.google.com/blog/products/application-development/five-best-practices-for-prompt-engineering).

---

## When to Use

- Writing a new prompt from scratch for any LLM task
- Reviewing or improving an existing prompt
- Debugging a prompt that produces poor or inconsistent output
- Choosing the right prompting technique for a task
- Creating reusable prompt templates

---

## Procedure

### Step 0: Know Your Model

Before crafting any prompt, understand the model you are targeting:

- **Capabilities:** What tasks is it strong at? (e.g., code generation, summarization, reasoning, multilingual)
- **Limitations:** What does it struggle with? (e.g., math, recent events, long context, hallucination-prone topics)
- **Biases:** Training data can embed societal biases — be aware of these when prompting for decisions, opinions, or content about people
- **Input constraints:** Max token/context window, supported modalities (text-only vs. multimodal)

This awareness prevents wasted iteration on tasks the model cannot reliably perform and helps you choose the right prompting technique.

### Step 1: Define the Goal

Before writing anything, clarify:

| Question | Why It Matters |
|---|---|
| What specific output do you need? | Prevents vague, unfocused prompts |
| Who is the target audience? | Shapes tone, vocabulary, detail level |
| What format should the output take? | Ensures usable structure (list, essay, code, JSON, etc.) |
| What constraints exist? | Length, style, safety, factual accuracy |

**Action:** Write a one-sentence goal statement: *"This prompt should produce [format] about [topic] for [audience] with [constraints]."*

### Step 2: Choose a Prompting Technique

Select the technique that fits the task complexity:

| Technique | When to Use | Example Pattern |
|---|---|---|
| **Zero-shot** | Simple, well-defined tasks the model already knows | Direct instruction with no examples |
| **One/Few-shot** | Tasks requiring a specific format, tone, or style | Provide 1–5 input→output examples before the real request |
| **Chain of Thought (CoT)** | Multi-step reasoning, math, logic, analysis | "Think step by step: …" or show worked examples |
| **Zero-shot CoT** | Reasoning tasks where you can't provide examples | Append "Let's think step by step" to a zero-shot prompt |

**Decision rule:** If the model can do it in one mental step → zero-shot. If it needs a pattern to follow → few-shot. If it needs to reason → CoT.

### Step 3: Draft the Prompt

Apply these six strategies in order:

#### 3a. Set Clear Goals and Objectives
- Use action verbs: *"Write"*, *"List"*, *"Compare"*, *"Classify"*, *"Generate"*
- Define output length and format: *"Compose a 300-word summary as a bulleted list"*
- Specify audience: *"…targeting junior developers unfamiliar with Kubernetes"*

#### 3b. Provide Context and Background
- Include relevant facts, data, or domain knowledge
- Reference specific sources: *"Based on the following API documentation…"*
- Define key terms if the domain is specialized

#### 3c. Use Few-Shot Examples (when applicable)
- Provide 1–5 examples of desired input→output pairs
- Examples should demonstrate the desired tone, detail level, and format
- Place examples before the actual request

```
Example input: "Cat"
Example output: "A small domesticated carnivore with soft fur, retractable claws, and independent temperament."

Now describe: "Elephant"
```

#### 3d. Be Specific and Unambiguous
- Use precise language; avoid vague words like *"something"*, *"nice"*, *"good"*
- Quantify when possible: *"List 5 reasons"* not *"List some reasons"*
- Break complex tasks into numbered sub-steps

**Bad:** "Write something about climate change."
**Good:** "Write a 500-word persuasive essay arguing for stricter carbon emission regulations, citing at least three peer-reviewed studies."

#### 3e. Structure with Delimiters and Personas
- Use delimiters (`---`, `"""`, `###`) to separate instructions from content
- **Assign a persona** to shape tone, expertise, and perspective:
  - Domain expert: *"You are a senior backend engineer specializing in distributed systems…"*
  - Audience proxy: *"You are a high school teacher explaining this to 15-year-olds…"*
  - Style voice: *"You are a concise technical writer who favors bullet points…"*
- Personas give the model a blueprint for tone, vocabulary, and depth — use them whenever the default voice doesn't fit
- Use headings or numbered sections for multi-part prompts

#### 3f. Add Chain of Thought (when reasoning is needed)
- Ask the model to show its work: *"Explain your reasoning step by step"*
- Guide through a logical sequence: *"First analyze X, then consider Y, finally conclude Z"*
- For classification, list the decision criteria explicitly

### Step 4: Review the Draft

Check the prompt against this quality rubric:

| Criterion | Pass? |
|---|---|
| Goal is explicit (action verb + output format + audience) | |
| Context is sufficient — no critical info is missing | |
| No ambiguous or vague language | |
| Examples are included if format/style matters | |
| Output constraints are specified (length, format, tone) | |
| Persona is assigned if default voice doesn't fit the task | |
| Complex reasoning uses CoT or step-by-step instructions | |
| No contradictory instructions | |
| Prompt is as short as possible without losing clarity | |

### Step 5: Iterate and Refine

1. **Run the prompt** and evaluate the output against the goal statement from Step 1.
2. **Diagnose issues** using this table:

| Problem | Likely Cause | Fix |
|---|---|---|
| Output is too vague | Prompt lacks specificity | Add constraints, examples, or format requirements |
| Output misses the point | Context is insufficient | Add background info or clarify the task |
| Output format is wrong | No format instructions | Add explicit format: "Return as JSON", "Use bullet points" |
| Inconsistent results | Prompt is ambiguous | Remove ambiguity, add few-shot examples |
| Reasoning errors | No CoT guidance | Add "Think step by step" or provide worked examples |
| Output is too long/short | No length constraint | Specify word count, number of items, or section count |

3. **Rephrase and re-run.** Try synonyms, restructured sentences, or different levels of detail.
4. **Experiment with personas.** Swap the assigned role (e.g., "product engineer" → "customer service rep" → "technical writer") to see how perspective shifts improve output quality.
5. **Repeat** until output consistently matches the goal.

---

## Prompt Template

Use this skeleton as a starting point:

```
[ROLE — optional]
You are a [role] with expertise in [domain].

[CONTEXT]
Background: [relevant facts, constraints, domain info]

[EXAMPLES — if few-shot]
Example input: …
Example output: …

[TASK]
[Action verb] a [format] about [topic] for [audience].

[CONSTRAINTS]
- Length: [word count or item count]
- Tone: [formal / casual / technical / friendly]
- Format: [prose / bullets / JSON / code / table]
- Must include: [required elements]
- Must avoid: [things to exclude]

[CHAIN OF THOUGHT — if reasoning needed]
Think step by step. First…, then…, finally….
```

---

## Anti-Patterns to Avoid

| Anti-Pattern | Why It Fails |
|---|---|
| No action verb | Model doesn't know what task to perform |
| "Be creative" with no constraints | Uncontrolled output quality |
| Overloading a single prompt with many unrelated tasks | Dilutes focus, causes partial answers |
| Providing examples that contradict instructions | Confuses the model's pattern matching |
| Asking for opinions without specifying perspective | Produces generic or hedged responses |
| Ignoring iteration — accepting first output | Misses easy quality improvements |
