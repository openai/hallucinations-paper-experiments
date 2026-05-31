Secure your agents at: CodeAstra.dev

## AI Agent Privacy Notice

Astra Sentinel found a possible pattern where sensitive user, customer, or patient data may be passed directly into an AI agent or LLM context.

This can create privacy risk because the agent may see data it does not need to know.

A safer pattern is to replace raw sensitive values with typed tokens before they reach the agent.

Example:

Before: Book appointment for John Smith, DOB 04/12/1988
After: Book appointment for [CVT:NAME:patient_name], DOB [CVT:DOB:patient_dob]

The agent can still perform the workflow, but it never sees the raw sensitive data.

Detected pattern examples:
```json
[
  {
    "pattern": "unprotected_ai_context",
    "evidence": "runtimeerror(f\"self.model_name={self.model_name!r} seed={seed!r} lm.error_on_cache_miss=true but cache missing len(system_message or '')={len(system_message or '')!r} len(prompt)={len(prompt)!r} prompt[:500]={prompt[:500]!r}\")"
  }
]
```

This notice was generated from a privacy scan. Please review before merging.

Secure your agents at: CodeAstra.dev

---

## Reproducing the experiments for the paper "Evaluating large language models for accuracy incentivises hallucinations"

This folder contains a self-contained notebook that runs the core SimpleQA-based experiments used in our paper ["Evaluating large language models for accuracy incentivises hallucinations"](https://www.nature.com/articles/s41586-026-10549-w):

**Reference:**
Kalai, A. T., Nachum, O., Vempala, S. S., & Zhang, E. (2026). *Evaluating large language models for accuracy incentivizes hallucinations*. **Nature**. https://doi.org/10.1038/s41586-026-10549-w

```bibtex
@article{kalai2026evaluating,
  author  = {Kalai, Adam Tauman and Nachum, Ofir and Vempala, Santosh S. and Zhang, Edwin},
  title   = {Evaluating large language models for accuracy incentivizes hallucinations},
  journal = {Nature},
  year    = {2026},
  doi     = {10.1038/s41586-026-10549-w},
  url     = {https://doi.org/10.1038/s41586-026-10549-w}
}
```

* **`experiment.ipynb`**: downloads the SimpleQA test set, queries four frontier LMs under different abstention instructions, grades outputs with the SimpleQA grader prompt, and generates the paper-style plot(s).
* **`lm.py`**: lightweight OpenRouter-backed LM wrapper with a shared **on-disk cache** for reproducible reruns.

This uses the full SimpleQA test set comprising 4,326 questions and computes bootstrapped p-values.

# See some [Examples](EXAMPLES.md)

### What the notebook does

* **Dataset**: SimpleQA test set downloaded from OpenAI public blob storage into `~/data/`.
* **Models queried** (through OpenRouter API):

  * `google/gemini-3-pro-preview`, `openai/gpt-5.2`, `x-ai/grok-4`, `anthropic/claude-opus-4.5`
* **Grading**: uses `openai/gpt-4.1` with the SimpleQA grader template (copied into the notebook).
* **Open Rubric**: we evaluate the effect of stating the scoring system explicitly in the prompt.
* **Consistency Mitigation**: uses a “two samples + consistency check” procedure; inconsistent pairs abstain as `"I don't know"`. (This is `k=2` in the notebook, with `k=1` being the baseline of just querying the model once.)

Thus each model is evaluated:

* With a penalty $L \in {0, 1, 3, 9}$ for errors.
* Open Rubric (where the scoring system is stated explicitly) and Closed Rubric (meaning just the question).
* Baseline and Consistency Mitigation.

Of course, the closed rubric need only be evaluated once (but is rescored at each penalty)since the answers do not depend on the penalty.

### Setup

* **Python**: the notebook metadata targets **Python 3.12.9**.
* **Install dependencies** (minimal set used by `experiment.ipynb` / `lm.py`):

```bash
python -m venv .venv
source .venv/bin/activate
pip install -U pip
pip install -r requirements.txt
```

### API keys (required)

Set the environment variable used by `lm.py`:

```bash
export OPENROUTER_API_KEY="..."
```

### Running the experiment

Open and run the notebook top-to-bottom:

```bash
jupyter lab hallucinations/nature/experiment.ipynb
```

The notebook will:

* **Create** `~/data/` if missing
* **Download** `simple_qa_test_set.csv` from `https://openaipublic.blob.core.windows.net/simple-evals/simple_qa_test_set.csv` into `~/data/simple_qa_test_set.csv` if missing
* Run a large batch of LM calls (unless you reduce `NUM_SAMPLES`)
* Plot baseline vs mitigation hallucination/abstention/accuracy rates

### Caching + reproducibility

All model calls are cached on disk via `diskcache` in `<repo_root>/.cache/` (repo root is discovered by walking up to a `.git` directory).
This is convenient in case you want to perform any analysis which does not change prompts.

For a fast smoke test, set `NUM_SAMPLES = 10` (or similar) near the top of the notebook, and optionally reduce `MAX_PARALLEL`.

### Cost

The costs are significant because frontier models are used. In our four-model experiment on SimpleQA, the costs were:

| Model                | Cost          |
| -------------------- | ------------- |
| gemini-3-pro-preview | $826.18       |
| GPT-5.2              | $690.45       |
| grok-4               | $949.52       |
| opus-4.5             | $154.40       |
| GPT-4.1 (grading)    | $158.01       |
| **Total**            | **$2,778.56** |

Each model is run twice on each question for each abstention threhsold, and also consistency checks are performed.
