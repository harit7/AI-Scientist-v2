# llm_runtimes integration notes

Additive backends for AI-Scientist-v2: `claudecli-{sonnet,opus,haiku}` (Claude
via the local `claude -p` CLI, subscription auth, no API key) and `local-qwen`
(in-process vLLM). All existing API-based model paths are untouched.

## Files changed

- `llm_runtimes/` — vendored runtime package (embedded OpenAI-compatible server).
- `ai_scientist/llm.py` — `create_client` branch for the new prefixes; the
  `ollama/` dispatch conditions extended to also match `claudecli-`/`local-`;
  the `"claude" in model` Anthropic branch now excludes `claudecli-`; new model
  names appended to `AVAILABLE_LLMS`.
- `ai_scientist/vlm.py` — same routing for VLM feedback stages. The claude
  CLI backend has real vision support: image parts are written to temp files
  and claude -p reads them with a Read-only tool allowance.
- `ai_scientist/treesearch/backend/backend_openai.py` — `get_ai_client` routes
  the new prefixes to the runtime server (tree-search experiment coding path).
- `ai_scientist/tools/semantic_scholar.py` — bounded backoff (max 90s) and a
  graceful degradation message instead of an infinite retry loop when Semantic
  Scholar rate-limits (it does, without an `S2_API_KEY`).
- `bfts_config_runtimes.yaml` — compute-constrained run config (2 workers,
  reduced per-stage iteration budgets, claudecli models everywhere).
- `ai_scientist/ideas/tiny_model_calibration.md` — research topic used for the
  overnight test run (simulated human input; see below).

## Usage

```bash
# ideation
python ai_scientist/perform_ideation_temp_free.py \
  --workshop-file ai_scientist/ideas/tiny_model_calibration.md \
  --model claudecli-sonnet --max-num-generations 2 --num-reflections 2

# experiments + paper (copy bfts_config_runtimes.yaml over bfts_config.yaml first)
python launch_scientist_bfts.py \
  --load_ideas ai_scientist/ideas/tiny_model_calibration.json --idea_idx 0 \
  --model_writeup claudecli-sonnet --model_citation claudecli-haiku \
  --model_review claudecli-haiku --model_agg_plots claudecli-sonnet --num_cite_rounds 4
```

Local model instead: use `local-qwen` for any of the model settings and start
the engine host with a free GPU (`python -m llm_runtimes.server --preload-local`
from the vLLM venv), or let the embedded engine start in-process if vllm is
importable. Usual API models work exactly as upstream documents.

## Simulated human inputs (for review)

- The research topic file `ai_scientist/ideas/tiny_model_calibration.md`
  (calibration of tiny neural networks under compute constraints) was written
  by the assistant, sized so generated experiments run on CPU in minutes.
- Run configuration choices (2 workers, small stage budgets, claudecli-sonnet
  for coding/writeup, claudecli-haiku for feedback/citation/review).

## Limitations

- `claude -p` ignores `temperature`; token usage is reported as zero.
- Semantic Scholar without a key degrades to "proceed without literature".
