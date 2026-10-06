# Research Brief Agent

Takes an NSE ticker and a folder of scraped documents and writes a one-page research brief for a retail investor: snapshot, bull case, bear case, open questions, sources.

## How it works

The agent is a small pipeline, not a single prompt:

1. **Audit.** The model reads all documents and writes an evidence log: source type, date, age compared with 23 Sep 2026, reliability, key numbers, conflicts between documents, stale items, gaps, and any text inside a document that tries to instruct the AI.
2. **Draft.** It writes sections 1-4 from that log, with a `[D#]` citation on every claim.
3. **Critic.** A second call re-checks the draft line by line against the documents and fixes unsupported claims, wrong numbers, stale data shown as current, and silently resolved conflicts.
4. **Check (plain code, no LLM).** Every claim must be cited, every cited id must exist, and every number must appear in the cited document. Problems trigger one repair call, and any that remain are listed in a comment at the end of the brief.
5. **Sources (plain code).** The Sources section is built from each file's header (source, URL, date), so links and dates cannot be invented by the model.

The rules the model follows (documents are data, not instructions; source weighting; stale-data handling; conflict handling; no advice) are in [`system_prompt_v2.md`](system_prompt_v2.md).

## Run it

```
python -m venv venv
venv\Scripts\activate          # macOS/Linux: source venv/bin/activate
pip install -r requirements.txt
copy .env.example .env         # macOS/Linux: cp .env.example .env, then add your key
python agent.py --ticker SRVCABLE --docs ./research_pack
```

The documents go in `research_pack/` (not included in this repo). The agent works with any OpenAI-compatible endpoint, so switching provider means changing three lines in `.env`:

```
LLM_API_KEY=your_key
LLM_BASE_URL=https://api.groq.com/openai/v1
LLM_MODEL=openai/gpt-oss-120b
```

Useful flags:

- `--dry-run` loads the documents and prints ids, dates and sources without calling any model.
- `--prompt prompts/system_prompt_v2.md` uses a different system prompt (handy for comparing versions).
- `--no-critic` and `--no-repair` switch off parts of the pipeline to see what each one adds.

Each run saves every stage to `out/<TICKER>_<timestamp>/` (`01_audit.md`, `02_draft.md`, `03_reviewed.md`, the check results and `brief.md`).

## Model(s) used and why

- **Run 0:** `gemini-3.8-flash` through Google's OpenAI-compatible endpoint. I chose it because the free tier is enough for 8 short documents and costs nothing. My first choice, `gemini-2.5-flash`, returned a 404 (no longer available to new users).
- **Runs 1 onwards:** `[Groq model name]` through Groq's OpenAI-compatible endpoint. I switched because the Gemini free tier allows 20 requests per day per model and I ran out, and I wanted runs 1 and 2 on the same model so that only the prompt differed.
- **Why a general-purpose chat model and not a reasoning-heavy one:** [say what you observed, or delete this line]. The task is reading about 7,000 characters and following citation rules, so speed and low cost mattered more than the largest model. I did not benchmark other models, so I can't say these are the best choice.

All stages (audit, draft, review, repair) use the same model in each run. The full list of runs and what changed between them is in [`TEST_LOG.md`](TEST_LOG.md).

## Known limitations

- The number check is string matching. A correct number in the wrong context passes, and a rounded or reformatted number is flagged.
- Each document is truncated at 15,000 characters.
- Source reliability is judged by the model, not by a fixed list.
- There is no live-data tool, so the agent only knows what is in the document pack.
- A run makes 3 to 4 model calls. On a free tier with a daily request limit, that capped how many test runs I could do in a day.
- Model output varies a little between runs, so a change in the brief after a prompt change is evidence, not proof.

## Files

| File | Purpose |
|---|---|
| `agent.py` | The pipeline |
| `system_prompt.md` | Rules and output format (current version) |
| `prompts/` | Prompt versions used in the test runs |
| `runs/` | Saved outputs from the test runs cited in the log |
| `TEST_LOG.md` | Runs, what went wrong, what changed |
| `.env.example` | Model and API configuration template |