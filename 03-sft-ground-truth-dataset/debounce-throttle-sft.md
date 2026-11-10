# Debounce and Throttle SFT Reference

03-sft-ground-truth-dataset/debounce-throttle-sft.md
Markdown

## Supervised Fine-Tuning (SFT): Gold-Standard Reference Generation

## 1. Task Metadata

- **Domain:** Frontend Web Performance / JavaScript Runtime
- **Task Type:** Golden Response Authoring & Negative Lexical Enforcement
- **Target Metric:** 100% Instruction Steerability, Negative Keyword Avoidance, Zero Conversational Noise

---

## 2. Input Prompt & Multi-Layered Constraints

> "Write a concise technical explanation comparing **debounce** vs **throttle** in frontend JavaScript for an event handler.
>
> Strict requirements:
>
> 1. Provide output under two Markdown level-2 headers (`## Debounce` and `## Throttle`).
> 2. Under each header, explain the core trigger mechanism in exactly one clear sentence.
> 3. Under each header, include a one-line bullet point (`-`) giving one ideal real-world browser event use case.
> 4. Do NOT use the words 'fast', 'quick', 'delay', or 'wait' anywhere in the response.
> 5. Total word count across the entire response must be strictly between 45 and 70 words (excluding headers)."

---

## 3. The 10/10 Gold-Standard Ground Truth Output

```markdown
## Debounce

Debounce postpones execution until a specified period of inactivity concludes following the final trigger event.

- Ideal for handling live search input fields that query backend APIs.

## Throttle

Throttle ensures the target callback executes at a steady, regulated frequency while continuous events fire.

- Ideal for managing window resize listeners tracking dimensions.

4. Verification & Constraint Compliance Audit
   Structural Integrity (Pass): Uses exact level-2 Markdown headers (## Debounce and ## Throttle).

Sentence Boundaries (Pass): Exactly one sentence under each header explaining the operational mechanism.

Bullet Point Syntax (Pass): Exactly one bullet point (-) under each header designating an applicable browser event.

Negative Lexical Enforcement (Pass): Zero instances of the banned words fast, quick, delay, or wait. Substituted with operational alternatives (postpones, concludes, regulated frequency).

Word Count Verification (Pass):

Debounce text: 26 words

Throttle text: 23 words

Total count: 49 words (excluding headers), falling strictly inside the required 45–70 word target window.

Conversational Cleanliness (Pass): Zero conversational preambles, apologies, or trailing closing remarks. Directly consumable by downstream training pipelines.

<ElicitationsGroup message="Select the next step to publish and polish your GitHub portfolio:">
  <Elicitation label="Generate complete root README.md file" query="Provide the complete Markdown text for the root README.md file to link all four portfolio audits together."/>
  <Elicitation label="Review Git commands to commit and push changes" query="Walk me through the terminal commands to commit all four files and push them to my remote GitHub repository."/>
  <Elicitation label="Format my resume with this new repository link" query="Show me how to update the technical projects section of my resume with this specific repository link."/>
</ElicitationsGroup>
```
