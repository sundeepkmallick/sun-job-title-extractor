---

license: mit
language:

* en
* de
  pipeline_tag: token-classification
  tags:
* job-title-extraction
* token-classification
* recruitment
* android
* on-device
* onnx
* int8
  base_model:
* microsoft/Multilingual-MiniLM-L12-H384

---

# Job Title Extractor — English & German

## Model description

This model is a multilingual token-classification model for extracting job-title spans from recruitment-related text.

It is designed for privacy-preserving, offline deployment, with Android on-device inference as the primary deployment scenario.

The model predicts BIO labels:

```text
O
B-JOB_TITLE
I-JOB_TITLE
```

The resulting token spans can be converted into a job-title string by a BIO decoder.

## Base model

The model was fine-tuned from:

```text
microsoft/Multilingual-MiniLM-L12-H384
```

The intended languages are:

* English
* German
* mixed English/German recruitment text

## Intended use

The model is intended for extracting job titles from recruitment-related communications, including:

* application acknowledgements
* application confirmations
* interview-related emails
* recruitment messages
* hiring communications containing a position title

Example:

```text
Thank you for applying for the Senior Android Engineer position.
```

Expected extraction:

```text
Senior Android Engineer
```

Public examples are synthetic and do not contain private recruitment-email data.

## Out-of-scope use

The model is not intended to:

* make hiring decisions
* rank candidates
* predict hiring outcomes
* determine candidate suitability
* provide employment advice
* determine whether a candidate will be hired
* extract arbitrary entities from arbitrary documents
* replace human decision-making

It is an extraction component, not a hiring or decision-making model.

## Entities not targeted

The intended extraction target is the job title.

The model is not intended to return:

* company names
* recruiter names
* candidate names
* job identifiers
* email addresses
* URLs
* hiring outcomes
* surrounding email prose

## Training

The model was fine-tuned using a private recruitment-related dataset.

The original training and validation examples are not publicly distributed.

The dataset contains:

* English examples
* German examples
* mixed-language examples
* positive job-title examples
* negative/no-title examples
* difficult recruitment-email examples

No private email bodies or other private recruitment information are included in this public model release.

### Training configuration

```text
Base model:
microsoft/Multilingual-MiniLM-L12-H384

Epochs:
8

Learning rate:
2e-5

Batch size:
8

Gradient accumulation:
2

Weight decay:
0.01

Warmup:
0.10

Maximum sequence length:
256

Seed:
42
```

Weighted token classification was used during training.

## Model architecture

The underlying model is a multilingual MiniLM transformer with a token-classification head.

The classifier predicts:

```text
O
B-JOB_TITLE
I-JOB_TITLE
```

The application reconstructs job-title spans using BIO decoding.

## Deployment

The production Android deployment uses an ONNX export with per-channel dynamic INT8 quantization.

Approximate sizes:

```text
FP32 ONNX:
448 MB

INT8 ONNX:
113 MB
```

Approximate reduction:

```text
75%
```

The INT8 model was selected for mobile deployment because it provides a substantially smaller artifact while preserving the desired frozen-benchmark behavior.

## Android integration

The Android deployment uses:

* ONNX Runtime
* Hugging Face Tokenizers
* Rust/JNI tokenizer integration
* candidate preprocessing
* candidate windowing
* BIO decoding
* deterministic validation
* regex fallback

The extraction path is designed to run locally on the Android device.

The model does not require a remote inference API.

## Evaluation

### Token-classification validation

The final model achieved the following validation metrics during development:

```text
F1:
0.876543

Precision:
0.835294

Recall:
0.922078
```

These are token-classification metrics on the private validation split.

### Frozen application benchmark

A private frozen 100-case benchmark was used to evaluate complete extraction behavior.

The final INT8 model achieved:

```text
98 / 100
```

expected extraction outcomes.

Observed result:

```text
98 correct outcomes
0 false positives
0 missed titles
2 extraction-span differences
```

This benchmark is not a claim of 98% general accuracy.

It is a result on a specific private frozen benchmark used for engineering regression testing.

## Limitations

Performance may vary for:

* languages other than English and German
* unusual job-title wording
* unusual email formatting
* heavily corrupted text
* OCR output
* very long or repetitive messages
* job titles that are not explicitly present
* non-recruitment documents
* resumes
* job-board pages
* arbitrary web pages
* subject-only processing

The model should be evaluated against domain-specific data before being used in another environment.

## Privacy considerations

The model is intended to support on-device processing of private recruitment emails.

The public model artifact does not contain the original private recruitment emails as training examples.

The public release must not include:

* real email bodies
* real email subjects
* private recruiter information
* private employer information derived from private emails
* private application identifiers
* private URLs
* private training datasets
* private validation datasets
* private benchmark examples

## Bias and limitations

The training and evaluation data were created for a specific recruitment-email extraction task.

The dataset is not intended to represent every language, geography, industry, employer, or recruitment communication style.

Consequently, performance may vary across domains and populations.

No claim is made that the model is equally accurate for every job-title format.

## Ethical considerations

This model should be used as an information-extraction component.

It should not be used as an automated decision-maker about people.

In particular, extracted job titles should not be interpreted as evidence that a person was hired, rejected, qualified, or suitable for a position.

## License

The project-specific model release is intended to use the MIT License, subject to the applicable licenses and attribution requirements of the upstream model and third-party dependencies.

The model is based on:

```text
microsoft/Multilingual-MiniLM-L12-H384
```

Users should review the upstream model's licensing and terms before redistribution or commercial use.

## Citation

If you use this model, please reference the model repository and the upstream Microsoft Multilingual MiniLM model.

Upstream model:

```text
microsoft/Multilingual-MiniLM-L12-H384
```

Model repository:

```text
sundeepmallick/sun-job-title-extractor-en-de
```

## Disclaimer

This model is provided as an information-extraction tool.

It does not provide legal, employment, hiring, recruitment, or career advice.

Model predictions may be incorrect and should be validated when they matter.
