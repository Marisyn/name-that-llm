## Description

18 large language models from six AI labs now generate everything from code explanations to creative fiction. As model-generated text fills the web and training datasets, knowing which model wrote a given passage is critical for auditing synthetic data and detecting benchmark contamination. Yet telling frontier models apart from a single short paragraph remains surprisingly difficult.

Standard AI detectors only try to separate machine-written text from human writing. Pinpointing the exact source model (a task known as authorship attribution) is much harder when prompts are unseen and passage lengths vary randomly between 40 and 250 words. You can't rely on simple character-count shortcuts or topic keywords. However, each lab's training data and post-training choices leave behind distinct stylistic fingerprints in vocabulary, phrasing, and punctuation.

In this competition, hosted by Kaggle, you'll build models to classify ~4,000 unlabeled text passages across 18 frontier language models from OpenAI, Google, Anthropic, Qwen, ZAI, and Moonshot AI. Every passage in the dataset is an authentic, self-contained response to prompts covering philosophy, physics, mathematics, biology, computer science, and creative prose.

Your solutions will show how reliably today's frontier models can be fingerprinted from short text alone. Better attribution methods will help researchers audit open benchmarks and understand what makes each model family distinct.

## abstract

Ask 18 frontier language models the same question, and no two answers read quite the same. Even in a single short paragraph, every model leaves subtle clues in its word choice, tone, and sentence structure.

In this competition, you'll identify which among 18 leading models wrote each unlabeled passage. Your work will sharpen methods for AI provenance and benchmark auditing.

## Timeline

* **October 14, 2026** - Start Date.

* **${competition.ProhibitNewEntrantsExplicitDeadline}** - Entry Deadline. You must accept the competition rules before this date in order to compete.

* **${competition.TeamMergerExplicitDeadline}** - Team Merger Deadline. This is the last day participants may join or merge teams.

* **January 13, 2027** - Final Submission Deadline.

All deadlines are at 11:59 PM UTC on the corresponding day unless otherwise noted. *The competition organizers reserve the right to update the contest timeline if they deem it necessary.*

## Evaluation

Submissions are evaluated on **balanced accuracy**: the average, over all candidate models, of the fraction of that model's test passages you labelled correctly (per-class recall).

$$\text{Balanced Accuracy} = \frac{1}{K} \sum_{k=1}^{K} \frac{\left|\{\, i : y_i = k \;\wedge\; \hat{y}_i = k \,\}\right|}{\left|\{\, i : y_i = k \,\}\right|}$$

where \\( K = 18 \\) is the number of candidate models, \\( y_i \\) is the true authoring model and \\( \hat{y}_i \\) is your prediction for passage \\( i \\).

The number of test passages per model is **not** equal, and the mix of models differs between the public and private leaderboard. Balanced accuracy gives every model the same weight.


## Valid Labels

Every prediction must be exactly one of the following 18 model names:

| Provider | Model Name |
|:---|:---|
| **OpenAI** | `gpt-5.5` |
| **OpenAI** | `gpt-5.6-luna` |
| **OpenAI** | `gpt-5.6-sol` |
| **OpenAI** | `gpt-5.6-terra` |
| **OpenAI** | `gpt-6-astra` |
| **Google** | `gemini-3.1-pro-preview` |
| **Google** | `gemini-3.6-flash` |
| **Google** | `gemini-3.7-flash` |
| **Google** | `gemini-3.8-flash` |
| **Anthropic** | `claude-fable-5` |
| **Anthropic** | `claude-opus-4-6` |
| **Anthropic** | `claude-opus-4-8` |
| **Anthropic** | `claude-opus-5` |
| **Anthropic** | `claude-sonnet-5` |
| **Qwen** | `qwen3.8-max` |
| **ZAI** | `glm-5.3` |
| **ZAI** | `glm-5.3-flash` |
| **Moonshot AI** | `kimi-k3` |

## Submission File

This is a Code Competition. Your notebook reads the test set from `/kaggle/input/competitions/name-that-llm/test.csv` and must write a file named `submission.parquet` to the notebook's working directory.

During an interactive session `test.csv` contains a single example row so your notebook can run end to end. When your notebook is submitted for scoring, it is re-run against the full hidden test set (~4000 passages).

For each `id` in the test set, predict the authoring `model`. Every `id` must appear exactly once. The file should contain a header and have the following format:

    id,model
    a1b2c3d4e5f6,gpt-6-astra
    b2c3d4e5f6a7,claude-opus-5
    c3d4e5f6a7b8,gemini-3.8-flash
    ...

A minimal example:

```python
import pandas as pd

test = pd.read_csv('/kaggle/input/competitions/name-that-llm/test.csv')
submission = pd.DataFrame({'id': test['id'], 'model': predictions})
submission.to_parquet('submission.parquet', index=False)
```

A submission is rejected if it is missing the `id` or `model` column, contains null values, repeats an `id`, or predicts a label outside the 18 listed above.

## Prizes

- 1st Place - $5,000
- 2nd Place - $5,000
- 3rd Place - $5,000
- 4th Place - $5,000
- 5th Place - $5,000
- 6th Place - $5,000
- 7th Place - $5,000
- 8th Place - $5,000
- 9th Place - $5,000
- 10th Place - $5,000

## Code Requirements

<img src="https://storage.googleapis.com/kaggle-media/competitions/general/Kerneler-white-desc2_transparent.png" style="float: right; width: 70px; padding: 10px">

Submissions to this competition must be made through Notebooks. In order for the "Submit" button to be active after a commit, the following conditions must be met: 
- CPU Notebook <= 9 hours run-time 
- GPU Notebook <= 9 hours run-time 
- Internet access disabled 
- Freely & publicly available external data is allowed, including pre-trained models 
- Submission file must be named `submission.parquet` 

Please see the [Code Competition FAQ](https://www.kaggle.com/docs/competitions#notebooks-only-FAQ) for more information on how to submit. And review the [code debugging doc](https://www.kaggle.com/code-competition-debugging) if you are encountering submission errors.