# Spider2-DBT Runnable62 Baseline Quickstart

This branch keeps the official Spider2-DBT runner/evaluator layout and adds only the runtime plumbing needed to run the baseline with OpenAI-compatible endpoints, Gemini, or GLM/Z.ai.

It also includes `spider2-dbt/examples/spider2-dbt-runnable62.jsonl`, the 62-case subset used for the shared Spider2-DBT table. The subset excludes six cases from the upstream DBT metadata: `airbnb002`, `biketheft001`, `chinook001`, `gitcoin001`, `google_ads001`, and `tpch002`.

## Setup

```bash
git clone -b zixuan/spider2-dbt-baseline https://github.com/Ternura143/Spider2.git
cd Spider2/methods/spider-agent-dbt
python -m pip install -r requirements.txt
```

Download and prepare the Spider2-DBT data:

```bash
cd ../../spider2-dbt
gdown 'https://drive.google.com/uc?id=1N3f7BSWC4foj-V-1C9n8M2XmgV7FOcqL'
gdown 'https://drive.google.com/uc?id=1s0USV_iQLo4oe05QqAMnhGGp5jeejCzp'
python setup.py
```

## Run GPT-5.4 Through An OpenAI-Compatible Endpoint

From `Spider2/spider2-dbt`, return to the agent directory:

```bash
cd ../methods/spider-agent-dbt
export OPENAI_API_KEY='<your key>'
export OPENAI_BASE_URL='<your endpoint, for example http://host:port/v1>'
export OPENAI_REQUEST_TIMEOUT=300

python run.py \
  --model gpt-5.4 \
  --suffix official-runnable62-ms30-gpt54 \
  --max_steps 30 \
  --test_path ../../spider2-dbt/examples/spider2-dbt-runnable62.jsonl \
  --example_index all \
  --overwriting
```

The experiment id is `gpt-5.4-official-runnable62-ms30-gpt54` because `run.py` prefixes the suffix with the model name.

## Run Gemini Direct

From the repository root:

```bash
cd methods/spider-agent-dbt
export GEMINI_API_KEY='<your key>'

python run.py \
  --model gemini-3.1-pro-preview \
  --suffix official-runnable62-ms30-gemini31pro \
  --max_steps 30 \
  --test_path ../../spider2-dbt/examples/spider2-dbt-runnable62.jsonl \
  --example_index all \
  --overwriting
```

## Run GLM/Z.ai Direct

From the repository root:

```bash
cd methods/spider-agent-dbt
export ZAI_API_KEY='<your key>'
export GLM_BASE_URL='https://api.z.ai/api/coding/paas/v4'

python run.py \
  --model glm-5.2 \
  --suffix official-runnable62-ms30-glm52 \
  --max_steps 30 \
  --test_path ../../spider2-dbt/examples/spider2-dbt-runnable62.jsonl \
  --example_index all \
  --overwriting
```

`GLM_API_KEY` is also accepted; if both are set, `GLM_API_KEY` takes precedence.

## Export And Evaluate

Replace `<experiment_id>` with the generated id, for example `gpt-5.4-official-runnable62-ms30-gpt54`.

```bash
cd methods/spider-agent-dbt
python get_spider2_submission_data.py \
  --experiment_suffix <experiment_id> \
  --results_folder_name ../../spider2-dbt/evaluation_suite/<experiment_id>

cd ../../spider2-dbt/evaluation_suite
python evaluate.py --result_dir <experiment_id>
```

## Notes

- This branch does not change the Spider2-DBT evaluator.
- The OpenAI path respects `OPENAI_BASE_URL` and `OPENAI_REQUEST_TIMEOUT`.
- The Gemini path uses `GEMINI_API_KEY` and Google `generateContent` directly.
- The GLM path uses OpenAI-compatible chat completions with `GLM_BASE_URL` plus `GLM_API_KEY` or `ZAI_API_KEY`.
- Keep run outputs under `methods/spider-agent-dbt/output/` and evaluation outputs under `spider2-dbt/evaluation_suite/<experiment_id>/`; do not commit generated outputs.
