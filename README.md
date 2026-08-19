# NLBSE'27 Tool Competition: Multitask Code Classification

This repository contains the dataset and baseline results for the
NLBSE'27 Tool Competition on multitask code classification.

Competition webpage: [https://nlbse2027.github.io/tool/](https://nlbse2027.github.io/tool/)

Participants must develop **one multi-label model** that jointly predicts:

1. whether a source-code function contains a weakness or vulnerability; and
2. whether the function is positive for the MAT annotation, which represents
   the competition's SATD-related target.

Models are compared by predictive performance, execution time, and
computational cost in a common evaluation environment.

- [Competition task](#competition-task)
- [Dataset](#dataset)
- [Loading the dataset](#loading-the-dataset)
- [Dataset preparation](#dataset-preparation)
- [Evaluation](#evaluation)
- [CodeBERT baseline](#codebert-baseline)
- [Participation](#participation)
- [Citing related work](#citing-related-work)

## Competition task

Each instance contains a formatted source-code function and one two-element
binary label. The label order is `[weakness, MAT]`:

| Label | Meaning |
| --- | --- |
| `[0, 0]` | Neither weakness-positive nor MAT-positive |
| `[1, 0]` | Weakness-positive and not MAT-positive |
| `[0, 1]` | Not weakness-positive and MAT-positive |
| `[1, 1]` | Both weakness-positive and MAT-positive |

Both outputs must be predicted by a single model. Separate classifiers for
individual projects, programming languages, or labels are not required.

**Dataset input:** The released dataset contains the field `Function` but no
separate developer-comment field. MAT is supplied as a target annotation; it
is not an additional text input in this release.

## Dataset

The competition dataset is derived from
[MADE-WIC](https://doi.org/10.1145/3691620.3695348) and combines instances from
the OSPR, Big-Vul, and Devign collections.

The [Hugging Face repository](https://huggingface.co/datasets/NLBSE/nlbse27-code-classification) provides two participant-facing splits:

| Hub split | Instances | Labels | Intended use |
| --- | ---: | --- | --- |
| `train` | **214,586** | Available | Model training |
| `test` | **30,655** | Available | Local evaluation and result reporting |

An additional **61,311-instance hidden test split** is retained by the
organisers for the final ranking. It is not distributed through this
repository and must not be used for training, tuning, or model selection.

Each published row contains four columns:

| Column | Description |
| --- | --- |
| `Projectname` | Name of the source project |
| `Filepath` | Original source-file path |
| `Function` | Source-code function after formatting |
| `Label` | Binary vector in `[weakness, MAT]` order |

## Loading the dataset

Install the Datasets library and load both public splits:

```bash
python -m pip install datasets
```

```python
from datasets import load_dataset

dataset = load_dataset("NLBSE/nlbse27-code-classification")
train_dataset = dataset["train"]
test_dataset = dataset["test"]

print(train_dataset)
print(test_dataset)
print(train_dataset[0])
print(train_dataset[0].keys())
```

## Dataset preparation

The dataset was prepared as follows:

1. Combine the OSPR, Big-Vul, and Devign MADE-WIC datasets.
2. Format every function with `clang-format` using the LLVM style.
3. Combine the original `Devign`, `Big-Vul`, and `W` annotations with a Boolean
   OR to obtain the weakness label.
4. Remove functions containing more than 500 `cl100k_base` tokens.
5. Downsample overrepresented joint `[weakness, MAT]` classes to restore the
   distribution that existed before token filtering.
6. Create deterministic, jointly stratified training, public test, and hidden
   test partitions using a 70/10/20 ratio.

## Evaluation

Precision, recall, accuracy, and F1-score are calculated independently for the
weakness and MAT outputs. The primary predictive metric is the unweighted mean
of the two task-specific F1-scores:

\[
F1_{mean} = \frac{F1_{weakness} + F1_{MAT}}{2}
\]

For the final competition ranking, mean F1 contributes 60% of the score.
Average execution time and average GFLOPs each contribute 20%. The organisers
will run every submission on the hidden test split in the same evaluation
environment. The complete scoring procedure is provided in the
competition notebook.

Competition notebook: [https://colab.research.google.com/drive/1b_EiXx5woyCrDntwbljfi-BwS9Dwvhoa?usp=sharing](https://colab.research.google.com/drive/1b_EiXx5woyCrDntwbljfi-BwS9Dwvhoa?usp=sharing)

## CodeBERT baseline

The baseline is based on
[`microsoft/codebert-base`](https://huggingface.co/microsoft/codebert-base). It
uses a shared CodeBERT encoder and a two-output multi-label classification head.

Validation data is not consulted during fine-tuning. The final trained model is
evaluated once on the complete public test split.

| Output | Accuracy | Precision | Recall | F1 |
| --- | ---: | ---: | ---: | ---: |
| Weakness/vulnerability | 0.960 | 0.943 | 0.935 | 0.939 |
| MAT | 0.996 | 0.978 | 0.802 | 0.881 |
| Unweighted average | 0.978 | 0.961 | 0.869 | 0.910 |

The trained baseline is available from the
[NLBSE'27 CodeBERT baseline repository](https://huggingface.co/NLBSE/nlbse27-code-classification-baseline).

## Participation

Participants must train and evaluate one model that predicts both labels. A
submission must include:

- a description of the model architecture and preprocessing;
- the training and tuning procedure;
- results on the published test split; and
- open-source code with instructions for reproducing the results.

Detailed rules and executable instructions will be published separately.

- Colab instructions: [https://colab.research.google.com/drive/1b_EiXx5woyCrDntwbljfi-BwS9Dwvhoa?usp=sharing](https://colab.research.google.com/drive/1b_EiXx5woyCrDntwbljfi-BwS9Dwvhoa?usp=sharing)
- CoteBERT baseline notebook: [https://colab.research.google.com/drive/1q1fGPpU7uxaK1mUVHa3b2xJOtEwhWwLG?usp=sharing](https://colab.research.google.com/drive/1q1fGPpU7uxaK1mUVHa3b2xJOtEwhWwLG?usp=sharing)

## Citing related work

Please also cite the work underlying the dataset:

```bibtex
@inproceedings{MockEtAl2024MadeWIC,
  author    = {Mock, Moritz and Melegati, Jorge and Kretschmann, Max and Diaz Ferreyra, Nicolas E. and Russo, Barbara},
  title     = {MADE-WIC: Multiple Annotated Datasets for Exploring Weaknesses In Code},
  booktitle = {Proceedings of the 39th IEEE/ACM International Conference on Automated Software Engineering},
  pages     = {2346--2349},
  year      = {2024},
  doi       = {10.1145/3691620.3695348}
}
```

```bibtex
@inproceedings{RussoEtAl2025VulSATD,
  author    = {Russo, Barbara and Melegati, Jorge and Mock, Moritz},
  title     = {Leveraging Multi-Task Learning to Improve the Detection of SATD and Vulnerability},
  booktitle = {2025 IEEE/ACM 33rd International Conference on Program Comprehension (ICPC)},
  pages     = {1--12},
  year      = {2025},
  doi       = {10.1109/ICPC66645.2025.00017}
}
```
