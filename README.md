# AGSciX

**Agentic Metadata Extraction from Scientific Papers and Code using Ontology-based Skills**

AGSciX extracts scientific metadata from an ML paper and its associated software repository without task-specific model fine-tuning. Reusable extraction skills guide an agent to identify workflow entities, classify them using an ontology, preserve source evidence, and produce structured outputs for analysis.

This repository contains the ontology, original and refined paper/code skills, benchmark inputs and predictions, paper--repository extraction outputs, and preprocessing/evaluation utilities used in the study.

## Contents

- [Study overview](#study-overview)
- [Repository layout](#repository-layout)
- [Ontology and output formats](#ontology-and-output-formats)
- [Datasets](#datasets)
- [Reported results](#reported-results)
- [Quick start](#quick-start)
- [Using skills with an agent](#using-skills-with-an-agent)
- [Manus](#manus)
- [Claude Code](#claude-code)
- [Codex: GPT-5.5, DeepSeek, GLM, and Kimi](#codex-gpt-55-deepseek-glm-and-kimi)
- [Benchmark extraction](#benchmark-extraction)
- [Licensing and attribution](#licensing-and-attribution)

## Study overview

The study investigates:

1. **Demonstration selection:** random, hard-example, and agent-guided sampling with five or ten demonstrations on GSAP-NER.
2. **Backend transfer:** the same dataset-specific skill is used with Manus, Claude Code, and Codex-based configurations using GPT-5.5, DeepSeek V4 Pro, GLM 5.2, and Kimi K3.
3. **Skill refinement:** original and refined ontology-guided skills are compared on nine paper--code pairs, using entity correctness assessment.

## Repository layout

```text
AGSciX/
|-- README.md
|-- code/
|   |-- calculate_precision.py
|   |-- count_paper_tokens.py
|   |-- pdf_conversion_markdown.py
|   |-- HyperXtractor_Preprocessing.ipynb
|   |-- HyperXtractor_anchor_Extraction.ipynb
|   |-- HyperXtractor_(SciNLP).ipynb
|   `-- Samlping_Strategies_(GSAP).ipynb
|-- taxonomy/
|   |-- Methodology-IE-V0.1.ttl / .owx
|   |-- Methodology-IE-V0.2.ttl / .owx
|   |-- Mermaid UML.txt
|   `-- ontology diagrams
`-- data/
    |-- GSAP Eval/
    |   |-- TXT/ and JSONL/
    |   |-- All NER Entities GSAP Test Set.csv
    |   |-- All Relations Entities GSAP Test Set.csv
    |   |-- GSAP-SKill.zip
    |   |-- agent-specific prediction folders
    |   `-- Samlping Strategies/
    |       |-- Skills/
    |       `-- Predictions/
    |-- SciNLP/
    |   |-- Inputs/
    |   |-- SciNLP_Entities_Test_Set.csv
    |   |-- SciNLP SKill v5.zip
    |   `-- agent-specific prediction folders
    `-- academic papers/
        |-- Markdown Files/
        |-- Skills/Original/
        |-- Skills/Refined/
        |-- Original_Skill_Output/Paper/ and Code/
        |-- Refined_Skill_Output/Paper/ and Code/
        `-- HyperXtractor_Dataset - Papers with Repos.csv
```

### Utilities

| File | Purpose | Execution notes |
|---|---|---|
| `code/count_paper_tokens.py` | Counts document tokens, words, and characters; writes CSV and totals | Executable CLI; uses `tiktoken` with `o200k_base` |
| `code/calculate_precision.py` | Reports manual correctness counts and precision by document, type, and ontology dimension | Currently reads a hard-coded `/content/` CSV path; change it before running |
| `code/HyperXtractor_Preprocessing.ipynb` | Converts annotations to CSV and checks source spans | Notebook; update local paths |
| `code/HyperXtractor_(SciNLP).ipynb` | Exact token-span evaluation on SciNLP | Contains separate document evaluation blocks; update paths and aggregate explicitly |
| `code/HyperXtractor_anchor_Extraction.ipynb` | Finds source spans and surrounding context | Notebook helpers |
| `code/Samlping_Strategies_(GSAP).ipynb` | Selects random and heuristic hard-example demonstration subsets | Set input bank, output directory, `K`, and seed |
| `code/pdf_conversion_markdown.py` | PDF/text/image conversion experiments | Colab-exported code contains `!pip` commands and hard-coded paths; not a ready-to-run Python CLI |

There is no packaged installation, the agent executes the workflow described by the selected skill.

## Ontology and output formats

Our proposed ontology describes 22 ML-workflow entity types organized into four dimensions:

| Dimension | Scope and representative entities |
|---|---|
| Meta-level | Study, Person, Organization |
| Paper-specific | Experiment, Task, Dataset, Method, Model, Algorithm, Prompt, Eval, Measure |
| Code-specific | SoftwareRepository, Implementation, Environment, Dependency, HyperParameter |
| Provenance-related | DataSource, DataProcessingStep, TrainingRun, EvaluationRun, PredictionSet |

The skills also define grouped experiments and a hyperparameter hierarchy covering optimization, training, architecture, and data settings.

Use the class names and relations in the selected skill's `references/schema_mermaid.md` and `references/output_schema.md`. The OWL serializations under `taxonomy/` are versioned separately; do not assume that every archived diagram, skill schema, and output uses identical names or namespaces.

Paper and code extraction produce JSON-LD objects with `@context` and `@graph`. Source metadata includes offsets, context, and extracted/inferred markers; code evidence additionally records file provenance. Inspect the bundled templates for the exact representation. JSON-LD samples use different namespace/name conventions across versions, so normalization must be explicit when combining files.

### Source grounding

- Preserve the exact input text used for extraction and offset calculation.
- Distinguish source-extracted values from inferred/normalized information using `valueSource`.
- Use missing-value conventions from the selected schema rather than inventing content.
- Check spans against the relevant paper or repository file.

## Datasets

| Evaluation setting | Documents used in the study | Input | Labels/output |
|---|---:|---|---|
| GSAP-NER | 10 test papers | Original plain text | Method, Dataset, MLModel, Task, DataSource, ModelArchitecture |
| SciNLP | 3 test papers | JSON containing tokenized sentences | Task, Method, Dataset, Metric in this repository's skill |
| AGSciX paper--code collection | 9 paper--repository pairs | Paper Markdown and corresponding repository | Ontology-guided workflow JSON-LD |

GSAP-NER and SciNLP are existing benchmarks, not newly created by this study. Their full datasets (train and val) are larger than the evaluated subsets.

### Nine evaluated pairs

| Paper ID | Repository |
|---|---|
| `1911.02116v2` | [XLM](https://github.com/facebookresearch/xlm) |
| `2104.08727v2` | [GooAQ](https://github.com/allenai/gooaq) |
| `2107.07653v3` | [Table-Pretraining](https://github.com/microsoft/Table-Pretraining) |
| `1908.08962v2` | [BERT](https://github.com/google-research/bert) |
| `1406.2661v1` | [adversarial](https://github.com/goodfeli/adversarial) |
| `2208.04405v3` | [Structure Identification](https://github.com/ASanctvs/Structure-Identification) |
| `2204.02965v1` | [LilNetX](https://github.com/Sharath-girish/LilNetX) |
| `2005.06182v2` | [mlsolva](https://github.com/ht0620/mlsolva) |
| `2021.acl-long.353` | [PrefixTuning](https://github.com/XiangLi1999/PrefixTuning) |

The original/refined output directories each contain nine paper and nine code JSON-LD files. Use those nine evaluated pairs for the study comparison.

## Reported results

### Demonstration sampling on GSAP-NER

Manus 1.6 Max; macro-F1 percentages over the six shared labels:

| Strategy | Shots | EM F1 | PM F1 | Total time (min) | Total cost ($) |
|---|---:|---:|---:|---:|---:|
| Random | 5 | 51.4 | 58.6 | 67 | 33.68 |
| Random | 10 | 50.6 | 55.6 | 69 | 34.12 |
| Hard-example | 5 | 46.2 | 52.3 | 77 | 39.55 |
| Hard-example | 10 | 50.1 | 55.6 | 68 | 33.01 |
| Agent-guided | 5 | **54.7** | **61.1** | **64** | **30.40** |
| Agent-guided | 10 | 47.8 | 54.7 | 73 | 32.46 |

Five-shot agent-guided sampling provides the best reported performance, runtime, and cost in this sampling comparison. More demonstrations do not consistently improve performance.

### Original versus refined skills on the paper--code collection

Codex GPT-5.5; precision is computed across reviewed extracted entities:

| Source | Skill | Correct | Incorrect | Reviewed | Precision (%) |
|---|---|---:|---:|---:|---:|
| Paper | Original | 659 | 107 | 766 | 86.03 |
| Paper | Refined | 570 | 85 | 655 | **87.02** |
| Code | Original | 1,275 | 206 | 1,481 | 86.09 |
| Code | Refined | 1,022 | 95 | 1,117 | **91.50** |

Refinement improves precision by 0.99 points for papers and 5.41 points for code, while reducing the number of extracted entities. Code supplies substantially more hyperparameters and dependencies. Gains are not uniform across types, and all reviewed PredictionSet predictions remain incorrect under both versions.

| Skill | Paper time (min) | Code time (min) | Combined time (min) | Combined cost (USD) |
|---|---:|---:|---:|---:|
| Original | 14.32 | 18.87 | 33.19 | 34.21 |
| Refined | 13.93 | 20.89 | 34.82 | 29.27 |

The refined version reduces combined cost by $ 4.94 while increasing total runtime by 1.63 minutes.

In the cross-backend experiments, the manuscript reports Codex GPT-5.5 as the strongest GSAP-NER backend and DeepSeek V4 Pro as the strongest SciNLP backend (75.7 EM macro-F1). Backend effectiveness therefore varies across datasets and entity types. Comparisons with supervised baselines should preserve the matching and aggregation conventions used in the manuscript.

## Quick start

Clone the repository and create an optional Python environment:

```bash
git clone https://github.com/Dahouabdelhalim/AGSciX.git
cd AGSciX
python3 -m venv .venv
source .venv/bin/activate
python -m pip install tiktoken PyYAML jupyter
mkdir -p work/skills work/input work/repos work/output
```

This installs dependencies for token counting, selected archived skill helpers, and notebooks. It is not a complete frozen environment for every conversion experiment. Install the agent separately using its official instructions.

Extract the refined skills:

```bash
python -m zipfile -e \
  "data/academic papers/Skills/Refined/paper2jsonld-contrib-extractor-refined-21-09-2026.zip" \
  work/skills

python -m zipfile -e \
  "data/academic papers/Skills/Refined/repo2json-contrib-extractor-refined-21-09-2026.zip" \
  work/skills

cp "data/academic papers/Markdown Files/1 - 1911.02116v2.md" work/input/paper.md
git clone https://github.com/facebookresearch/xlm.git work/repos/xlm
```

Each skill is a folder containing `SKILL.md`, references, scripts, and sometimes templates. Read `SKILL.md` together with its references; copying only the manifest loses the supporting resources.

## Using skills with an agent

The three components have distinct roles:

| Component | Role |
|---|---|
| Paper Markdown or benchmark input | The source evidence being analyzed |
| Skill folder | Instructions, ontology/schema guidance, extraction policies, demonstrations, and helper scripts |
| Agent and underlying model | Read the evidence, apply the skill, use tools, calculate/check offsets, and write the output |

A Markdown paper is not a skill. `SKILL.md` is the skill's entry point.

### Paper extraction prompt

Use this with the refined paper skill installed or attached. Adapt the skill path for your agent:

```text
Use the refined extraction skill.
Read the complete SKILL.md and its referenced extraction policy,
output schema, and schema_mermaid.md before extracting.

PAPER_MD: work/input/paper.md
SCHEMA_TEXT: the schema and policy bundled with this selected skill;
read references/schema_mermaid.md and references/extraction_policy.md.

Extract the paper's contribution-related workflow entities and relations.
Write the JSON-LD object to work/output/paper.jsonld.
```

### Repository extraction prompt

```text
Use the refined repository extraction skill.

Read the complete SKILL.md and its referenced schema, extraction policy,
output schema, and file-pattern guidance before extracting.

REPO_URL: https://github.com/facebookresearch/xlm
LOCAL_REPO_PATH: work/repos/xlm
SCHEMA_TEXT: the schema and policy bundled with this selected skill;
read references/schema_mermaid.md and references/extraction_policy.md.

Extract the paper's contribution-related workflow entities and relations.
Write the JSON-LD object to work/output/paper.jsonld.
```

These are usage templates, not archived verbatim experimental prompts. For a paper--code comparison, run both prompts and retain both source-specific outputs.

## Manus

1. Open a Manus task and select the desired agent configuration; the study used Manus 1.6 Max.
2. Attach the chosen skill ZIP and paper Markdown. For repository extraction, also provide the repository URL or an accessible repository snapshot.
3. Ask Manus to unpack the ZIP, read the complete `SKILL.md` and its references, and use that skill for extraction.
4. Provide the prompt above and use Manus's sandbox rather than local paths on your computer.
5. Request downloadable JSON-LD files and retain execution/usage logs.

If the skill is registered in your Manus account, select/invoke it using the interface's available skill mechanism. Manus documents explicit `/SKILL_NAME` invocation. If it is only attached as a ZIP, explicitly asking Manus to unpack and read it is a filesystem workflow; attaching it alone does not establish that it was registered or invoked.

Official reference: [Manus Agent Skills](https://manus.im/blog/manus-skills).

## Claude Code

Install and authenticate [Claude Code](https://code.claude.com/docs/en/setup), then install project-local copies of the extracted skill folders:

```bash
mkdir -p .claude/skills/agsci-paper .claude/skills/agsci-code
cp -R work/skills/paper2jsonld-contrib-extractor-refined-21-09-2026/. \
  .claude/skills/agsci-paper/
cp -R work/skills/repo2json-contrib-extractor-refined-21-09-2026/. \
  .claude/skills/agsci-code/
claude
```

Launch Claude Code from the repository root. Select the intended model using the available model selector; the study reports Claude Opus 4.7. Use the exact model identifier your account exposes rather than guessing a version slug.

Paste the paper/repository prompt and explicitly identify `.claude/skills/agsci-paper/SKILL.md` or `.claude/skills/agsci-code/SKILL.md`. Claude Code also supports slash invocation for discovered skills. Check the available command before using it.

Official reference: [Claude Code skills](https://code.claude.com/docs/en/skills).

## Codex: GPT-5.5, DeepSeek, GLM, and Kimi

### Install the same skills for Codex

Install and authenticate [Codex CLI](https://developers.openai.com/codex/cli/). Current Codex documentation uses `.agents/skills/` for project-local skill discovery:

```bash
mkdir -p .agents/skills/agsci-paper .agents/skills/agsci-code
cp -R work/skills/paper2jsonld-contrib-extractor-refined-21-09-2026/. \
  .agents/skills/agsci-paper/
cp -R work/skills/repo2json-contrib-extractor-refined-21-09-2026/. \
  .agents/skills/agsci-code/
```

Launch from the repository root and use the same prompts, explicitly referencing `.agents/skills/agsci-paper/SKILL.md` or `.agents/skills/agsci-code/SKILL.md`. Older clients may use different skill paths; consult the documentation for the version you record.

### GPT-5.5 through OpenAI

After configuring OpenAI authentication and confirming access to the model:

```bash
codex -m gpt-5.5
```

Select only reasoning settings supported by the model and record them. Keep extraction instructions and the source document constant when changing models.

References: [GPT-5.5](https://developers.openai.com/api/docs/models/gpt-5.5), [Codex skills](https://developers.openai.com/codex/skills/).

### DeepSeek V4 Pro

DeepSeek provides an official Codex integration, including Responses API support and a model catalog. Follow [DeepSeek's Codex configuration guide](https://api-docs.deepseek.com/quick_start/agent_integrations/codex/) to configure the client and model metadata.

For a client already configured with a provider named `deepseek`, the essential provider settings follow this pattern:

```toml
model = "deepseek-v4-pro"
model_provider = "deepseek"

[model_providers.deepseek]
name = "DeepSeek"
base_url = "https://api.deepseek.com"
env_key = "DEEPSEEK_API_KEY"
wire_api = "responses"
```

Set `DEEPSEEK_API_KEY` in your shell environment through your preferred secret-management method, then launch Codex. This snippet documents the endpoint/authentication pattern; it does not replace DeepSeek's complete model-catalog instructions.

The same installed AGSciX skills and paper/code prompts apply. DeepSeek is the underlying model; Codex supplies the agent tools and orchestration.

### GLM 5.2 and Kimi K3 through OpenRouter

OpenRouter documents a Codex-compatible Responses API setup. Configure a separate provider rather than changing the built-in OpenAI provider. Merge the following provider block into your existing `~/.codex/config.toml`:

```toml
[model_providers.openrouter]
name = "OpenRouter"
base_url = "https://openrouter.ai/api/v1"
env_key = "OPENROUTER_API_KEY"
wire_api = "responses"
```

Set `OPENROUTER_API_KEY` in the environment of the shell launching Codex. Choose the model at launch:

```bash
# GLM
codex -m z-ai/glm-5.2 -c 'model_provider="openrouter"'

# Kimi
codex -m moonshotai/kimi-k3 -c 'model_provider="openrouter"'
```

These slugs identify the model routes, not the provider used in every historical experiment. Confirm that the selected route currently supports the client-required Responses API, streaming, and tool calls. OpenRouter routes can have different serving providers; record the actual serving provider when comparing cost, latency, or reproducibility.

The study reports **Max reasoning** for GLM and Kimi. This is a reasoning setting, not a suffix to the model's API identifier. Configure it only if the selected client/model route advertises and accepts it; otherwise document the supported setting rather than silently treating another setting as equivalent.

References: [OpenRouter Codex guide](https://openrouter.ai/blog/tutorials/codex-cli-openrouter/), [GLM 5.2](https://openrouter.ai/z-ai/glm-5.2), [Kimi K3](https://openrouter.ai/moonshotai/kimi-k3), [Codex configuration](https://developers.openai.com/codex/config-reference/).

## Benchmark extraction

Benchmark skills have different input/output contracts from ontology-wide paper/code extraction.

### GSAP-NER

Unpack `data/GSAP Eval/GSAP-SKill.zip` or a specific strategy archive under `data/GSAP Eval/Samlping Strategies/Skills/`. Provide one original file from `data/GSAP Eval/TXT/` and explicitly ask the agent to use the six-label skill.

The skill outputs `final_annotations.csv` with:

```text
doc_id,local_entity_id,text,start_char,end_char,ner_label
```

Character offsets are zero-based and end-exclusive:

```python
raw_text[start_char:end_char] == text
```

The agent must use the benchmark's original text rather than the separately converted Markdown paper, since conversion changes offsets. Save each document's output in a separate directory to prevent repeated `final_annotations.csv` filenames from overwriting previous runs.

### SciNLP

Unpack `data/SciNLP/SciNLP SKill v5.zip`. Supply one single-document JSON from `data/SciNLP/Inputs/` and retain its tokenized sentences exactly as provided. Do not supply gold `ner` labels as extraction input.

The skill outputs `final_annotations.csv` with:

```text
doc_key,local_entity_id,text,start_token,end_token,ner_label
```

Token offsets are zero-based, document-global, and inclusive at both ends. Flatten the sentence token lists without changing tokens:

```python
tokens = [token for sentence in document["sentences"] for token in sentence]
mention_text = " ".join(tokens[start_token:end_token + 1])
```

For **length analysis only**, join sentences into plain text before tokenization and exclude JSON structure and annotation fields. The counting script expects a single JSON object with `sentences`; the benchmark's `test.json` may contain multiple JSONL records, so do not pass it as though it were one document.

The script reports total/average tokens and writes document-level counts. It uses `o200k_base` vocabulary as a consistent text-length measure.
## Licensing

Our AGSciX dataset released under the Creative Commons Attribution-NonCommercial 4.0 International (CC BY-NC 4.0) license.
