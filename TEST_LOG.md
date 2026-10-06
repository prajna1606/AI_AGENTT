# Test log

Each run saves its audit, draft, reviewed draft and check results in `runs/`. System prompt versions are in `prompts/`.

## Run 0: Gemini, prompt v1 (first working run)

- **Setup:** `prompts/system_prompt_v1.md`, `gemini-3.8-flash` via Google's OpenAI-compatible endpoint, full pipeline (audit, draft, critic, check, repair).
- **Before it worked:** my first model, `gemini-2.5-flash`, returned a 404 (no longer available to new users), and the agent retried it 4 times for nothing. After switching to `gemini-3.8-flash` I got 503 "high demand" and 429 per-minute quota errors, which the retries recovered from.
- **What went wrong in the brief:**
  - The code checker found 1 problem, which the repair pass fixed. [Say what it was, from `04_check_1.txt`.]
  - The brief says D1 is wrong about revenue, but still cites D1 (with D3) for the copper cost figure. [Check whether D3 states the copper figure on its own.]
  - The brief says D1 "appears to have transposed digits". The documents don't say that, so it's an inference stated as fact.
  - D6 was not cited, and the brief doesn't say why. [Say what D6 is and whether it should have been used.]
  - The brief is longer than the 450-word target. [Add your word count.]
- **Evidence:** `runs/run0_gemini/` [folder name], `brief.md`, `01_audit.md`, `04_check_1.txt`.
- **What I changed:** wrote prompt v2, adding rules that a cause for a discrepancy must be supported by the documents or labelled "likely", that unreliable or contradicted documents are never cited as support, that the brief stays within 450 words, and that unused documents are explained in one line.

## Run 1: Groq, prompt v1 (baseline on the new model)

- **Why the model changed:** my rerun with v2 stopped in the audit stage with a 429 daily quota error (20 requests per day for `gemini-3.8-flash`, retry in about 12.5 hours). I switched to Groq so I could run v1 and v2 on the same model and change only the prompt.
- **Setup:** `prompts/system_prompt_v1.md`, `[Groq model name]`, full pipeline.
- **What went wrong:** [from the brief]
- **Evidence:** `runs/run1_.../`
- **What I changed:** [nothing, this is the baseline, or note any fix to agent.py]

## Run 2: Groq, prompt v2

- **Setup:** `prompts/system_prompt_v2.md`, `[Groq model name]`, full pipeline. Only the prompt changed from run 1.
- **What improved:** [compare with run 1: D1 citation, the transposition claim, D6, word count]
- **What still went wrong:** [from the brief]
- **Evidence:** `runs/run2_.../`
- **What I changed:** [for run 3]

## Run 3

- **Setup:** [prompt v3, or `--no-critic --no-repair` on v2, and the model]
- **What went wrong / what it showed:** [for the no-critic comparison: what the critic and repair passes caught, or that they changed nothing]
- **Evidence:** `runs/run3_.../`
- **What I changed:** [or "nothing, final configuration"]

## Final run

- **Setup:** [which run produced the pasted brief, prompt version, model]
- **Remaining problems:** [be honest, e.g. string-matching number check, model-judged source reliability]
- **Hand edits to the pasted brief (if any):** [none, or exactly what changed]