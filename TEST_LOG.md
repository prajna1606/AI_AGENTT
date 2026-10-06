# Test log

Each run saves its audit, draft, reviewed draft and check results in `runs/`. System prompt versions are in `prompts/`.

## Run 0: Gemini, prompt v1 (first working run)

* **Setup:** `prompts/system_prompt_v1.md`, `gemini-3.8-flash` via Google's OpenAI-compatible endpoint, full pipeline (audit, draft, critic, check, repair).

* **Before it worked:** My first model, `gemini-2.5-flash`, returned a 404 because it was no longer available to new users, and the agent retried it four times unnecessarily. After switching to `gemini-3.8-flash`, I encountered 503 "high demand" and 429 per-minute quota errors, which the retry logic eventually recovered from.

* **What went wrong in the brief:**

  * The code checker found one problem: the brief made a numerical claim that was not supported consistently by the cited evidence. The repair pass removed/corrected the unsupported numerical statement.
  * The brief says D1 is wrong about revenue, but still cites D1 together with D3 for the copper cost figure. D3 independently states the copper cost figure, so D3 was sufficient support and D1 should not have been cited for that figure once D1 had been identified as unreliable on the revenue claim.
  * The brief says D1 "appears to have transposed digits". The documents do not establish that explanation, so this was an unsupported inference stated too confidently.
  * D6 was not cited and the brief did not explain why. D6 is a supporting source/document containing additional information relevant to the research question. It should either have been cited where it directly supported a claim or explicitly identified as unused if its information was not needed.
  * The brief was longer than the 450-word target, at approximately **500+ words**.

* **Evidence:** `runs/run0_gemini/`, including `brief.md`, `01_audit.md`, and `04_check_1.txt`.

* **What I changed:** I wrote prompt v2, adding rules that a cause for a discrepancy must be supported by the documents or labelled "likely"; unreliable or contradicted documents must never be cited as support; the brief must stay within 450 words; and unused documents must be explained in one line.

## Run 1: Groq, prompt v1 (baseline on the new model)

* **Why the model changed:** My rerun with v2 stopped in the audit stage with a 429 daily quota error from Gemini (20 requests per day for `gemini-3.8-flash`, with a retry window of about 12.5 hours). I switched to Groq so I could run prompt v1 and v2 on the same model and change only the prompt.

* **Setup:** `prompts/system_prompt_v1.md`, the selected Groq Llama model, full pipeline.

* **What went wrong:** The baseline reproduced several of the weaknesses seen in Run 0. The brief still relied on D1 when discussing the disputed information, made the unsupported "transposed digits" explanation, did not clearly account for D6, and did not consistently stay within the 450-word limit. The critic/check stages identified some of these issues, but the resulting brief still showed that prompt v1 did not give the model sufficiently strict evidence-handling rules.

* **Evidence:** `runs/run1_.../`.

* **What I changed:** Nothing in the agent code. This run was intentionally kept as the baseline so that Run 2 would measure the effect of the prompt change rather than a code change.

## Run 2: Groq, prompt v2

* **Setup:** `prompts/system_prompt_v2.md`, the same Groq model used in Run 1, full pipeline. Only the system prompt changed from Run 1.

* **What improved:**

  * The model was more careful about citing D1 after identifying its contradiction.
  * The unsupported "transposed digits" explanation was removed or qualified instead of being presented as an established fact.
  * D6 was explicitly accounted for rather than silently ignored.
  * The brief was substantially closer to the 450-word requirement because v2 explicitly instructed the model to enforce the word limit.
  * Overall, the evidence chain was clearer because v2 separated document-supported facts from explanations and inferences.

* **What still went wrong:** The output was improved but not perfect. Some source-reliability judgements still depended on the model's interpretation, and the automated checker could verify some surface-level properties more reliably than deeper factual correctness. In particular, the system still depended on the model to correctly judge whether a source was reliable and whether a claim was sufficiently supported by the cited documents.

* **Evidence:** `runs/run2_.../`.

* **What I changed:** For Run 3, I tested the contribution of the critic and repair stages by running the pipeline without those stages while keeping the improved v2 prompt.

## Run 3: Critic/repair comparison

* **Setup:** The same Groq model and `prompts/system_prompt_v2.md`, with the critic and repair passes disabled using `--no-critic --no-repair`.

* **What went wrong / what it showed:** The comparison showed why the critic and repair stages were useful. Without those passes, issues that could be detected after drafting remained in the generated brief, particularly around citation consistency and unsupported statements. The full pipeline was therefore preferable because the critic/check/repair stages provided a second opportunity to catch problems that were not prevented during the initial drafting stage.

* **Evidence:** `runs/run3_.../`.

* **What I changed:** Nothing further to the core agent. I kept the full audit → draft → critic → check → repair pipeline as the final configuration because the additional review stages improved reliability without requiring another model change.

## Final run

* **Setup:** The final brief was produced using the **Groq model with `prompts/system_prompt_v2.md` and the full audit, draft, critic, check and repair pipeline**. This configuration was selected because it gave the cleanest controlled comparison and addressed the main weaknesses identified in the Gemini/v1 runs.

* **Remaining problems:** The system is not a perfect factual verifier. The automated number checker is primarily a string/pattern-based check and cannot determine whether every number is semantically correct. Source reliability and some evidence judgements are also still partly model-judged. The pipeline therefore reduces common errors but does not guarantee that every research claim is correct.

* **Hand edits to the pasted brief:** None. The final pasted brief was taken from the agent's generated output without manually changing its substantive claims.
