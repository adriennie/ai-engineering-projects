# AI Engineering Projects

Practical projects for building reliable AI, data, and machine-learning
systems.

This repository is organized as a collection of focused implementations.
Each project has its own runtime, dependencies, documentation, and execution
workflow. The emphasis is on useful system behavior: ingestion, validation,
preprocessing, model integration, reproducibility, error handling, and
operational visibility.

## Projects

| Project | Description | Primary technologies | Status |
| --- | --- | --- | --- |
| [FeedDigest](FeedDigest/) | RSS/Atom news ingestion and LLM summarization CLI | Python, `feedparser`, `requests`, multiple LLM APIs | Runnable |
| [Curator](curator/) | Deterministic tabular data curation and MLOps pipeline | Python, pandas, Pandera, scikit-learn, Typer | Runnable |

## FeedDigest

[FeedDigest](FeedDigest/) fetches articles from RSS and Atom feeds, extracts
usable text, and summarizes the selected articles through a configured LLM
provider.

### Capabilities

- Process one or more feeds with repeatable `--feed` options.
- Limit the number of articles summarized per feed.
- Support OpenAI, Anthropic, Hugging Face, Groq, Ollama, and custom
  OpenAI-compatible endpoints.
- Load credentials and runtime configuration from environment variables.
- Export summaries to an append-only Markdown report.
- Continue after malformed feeds, network failures, invalid responses, or
  unusable articles.
- Test network and provider behavior deterministically with mocked requests.

### Run FeedDigest

```bash
cd FeedDigest
python3 -m venv .venv
source .venv/bin/activate
python -m pip install -r requirements.txt
```

Configure the provider settings described in
[FeedDigest/readme.md](FeedDigest/readme.md), then run:

```bash
python main.py \
  --feed https://example.com/rss.xml \
  --limit 5 \
  --output results.md
```

Run its tests:

```bash
python -m unittest discover -s tests
```

Read the complete [FeedDigest documentation](FeedDigest/readme.md) for
provider configuration, supported response formats, error behavior, and
roadmap items.

## Curator

[Curator](curator/) validates and prepares tabular data for machine-learning
workflows. It is intentionally narrow and deterministic so that its input
contract, transformations, outputs, and audit trail remain easy to inspect.

### Processing workflow

```text
CSV input
  -> strict Pandera schema validation
  -> group-aware median imputation
  -> IsolationForest outlier detection
  -> remove flagged outliers from model splits
  -> deterministic train/test split
  -> fit StandardScaler on training data only
  -> write datasets, logs, and a run report
```

### Input contract

The input CSV must contain exactly these columns:

| Column | Rules |
| --- | --- |
| `amount` | Nullable, numeric, and greater than or equal to `0` |
| `tenure` | Nullable, numeric, and greater than or equal to `0` |
| `score` | Nullable numeric value within the configured bounds |
| `group` | Required value: `alpha`, `beta`, or `gamma` |
| `target` | Nullable numeric target |

### Run Curator

```bash
cd curator
python3 -m venv .venv
source .venv/bin/activate
python -m pip install -r requirements.txt

python pipeline.py generate \
  --count 5000 \
  --output-dir mock_data

python pipeline.py run \
  --input-file mock_data/mock_data.csv \
  --output-dir output
```

The pipeline produces:

- `curated_full.csv` — all validated rows with imputed features and an
  `outlier` flag
- `X_train.csv` and `X_test.csv` — scaled features for inlier rows
- `y_train.csv` and `y_test.csv` — corresponding targets
- `pipeline.log.jsonl` — append-only structured JSON Lines events
- `reports/latest.md` — the latest execution summary

The checked-in sample run processed 5,000 rows, flagged 805 outliers, and
produced 3,356 training rows and 839 test rows.

Read the complete [Curator documentation](curator/readme.md), plus the
[product requirements](curator/docs/PRD.md) and
[engineering reference](curator/docs/ENGINEERING.md).

## Repository layout

```text
ai-engineering-projects/
├── FeedDigest/
│   ├── fetch.py
│   ├── main.py
│   ├── summarize.py
│   ├── tests/
│   ├── requirements.txt
│   └── readme.md
├── curator/
│   ├── pipeline.py
│   ├── mock_data/
│   ├── output/
│   ├── docs/
│   ├── requirements.txt
│   └── readme.md
├── .gitignore
└── readme.md
```

## Design principles

- **Small, focused systems:** each project solves a defined engineering
  problem without unnecessary platform complexity.
- **Explicit contracts:** configuration, schemas, command-line arguments, and
  generated artifacts are documented.
- **Deterministic workflows:** fixed seeds, isolated transformations, and
  reproducible local execution are preferred where appropriate.
- **Safe failure behavior:** invalid input and external-service failures are
  surfaced clearly without leaking secrets or raw provider responses.
- **Operational evidence:** outputs, structured logs, reports, and tests make
  behavior inspectable.

## Current scope

The repository currently contains two runnable Python projects:

- FeedDigest for feed ingestion and LLM-assisted summarization.
- Curator for validated, leakage-aware tabular data preparation.

The projects are independent. Install dependencies and run commands from the
individual project directory rather than from the repository root.

## Licensing

Each runnable project has its own license file:

- [FeedDigest license](FeedDigest/docs/LICENSE.txt)
- [Curator license](curator/docs/LICENSE.txt)
