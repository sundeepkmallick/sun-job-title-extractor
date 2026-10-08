# Sun Job Title Extractor

A multilingual, privacy-preserving job-title extraction system for recruitment emails, designed for **offline Android inference**.

The system uses a fine-tuned multilingual token-classification model, exported to **ONNX INT8**, combined with candidate preprocessing, BIO decoding, deterministic validation, and a regex fallback.

> **Private email content is not included in this public repository.**

## Project

Sun Job Title Extractor extracts job titles from recruitment-related email text.

For example:

```text
Thank you for applying for the position of Senior Android Engineer.
```

The system attempts to extract:

```text
Senior Android Engineer
```

The model supports recruitment-related text in **English and German**.

---

## Why this project?

Job-title extraction from recruitment emails looks simple, but real emails contain:

* company names
* recruiter names
* application identifiers
* URLs
* signatures
* quoted replies
* boilerplate text
* multiple mentions of positions
* English and German language
* titles embedded inside longer sentences

A large rule-based regex extractor can become difficult to maintain and can produce incorrect spans.

This project explores a lightweight alternative:

> **Use a small multilingual token-classification model to identify the exact tokens belonging to a job title, then validate the result deterministically on-device.**

---

## Architecture

```text
                    Recruitment Email
                           |
                           v
              JobTitleCandidatePreprocessor
                           |
                           v
                  Candidate Windowing
                           |
                           v
            Multilingual MiniLM Token Classifier
                           |
                           v
                      BIO Decoder
                           |
                           v
                Deterministic Validation
                           |
                +----------+----------+
                |                     |
              valid          invalid/unavailable
                |                     |
                v                     v
        ON_DEVICE_MODEL        Regex Fallback
                |                     |
                +----------+----------+
                           |
                           v
                     JobTitleResult
```

The production Android architecture keeps the complete extraction pipeline on the device.

No recruitment email content needs to be sent to a remote AI service.

---

## Model

The model is fine-tuned from:

```text
microsoft/Multilingual-MiniLM-L12-H384
```

The final production model is exported to:

```text
ONNX INT8
```

using **per-channel dynamic quantization**.

### Model size

| Format    | Approximate size |
| --------- | ---------------: |
| FP32 ONNX |           448 MB |
| INT8 ONNX |           113 MB |

The INT8 model is approximately **75% smaller** than the FP32 ONNX export.

The public deployable model is available on Hugging Face:

**https://huggingface.co/sundeepmallick/sun-job-title-extractor-en-de**

---

## Token classification

The model uses three BIO labels:

```text
O
B-JOB_TITLE
I-JOB_TITLE
```

For example:

```text
Senior Android Engineer
```

is represented conceptually as:

```text
Senior       B-JOB_TITLE
Android      I-JOB_TITLE
Engineer     I-JOB_TITLE
```

The Android implementation then reconstructs the job-title span and applies deterministic validation.

---

## Android implementation

The production Android integration uses:

* Kotlin
* ONNX Runtime Android
* INT8 ONNX inference
* Hugging Face Tokenizers
* Rust tokenizer implementation
* JNI bridge
* BIO decoding
* deterministic validation
* regex fallback

The tokenizer is integrated through a native Rust/JNI component to provide the tokenizer behavior required by the model.

The model is loaded locally by the Android application.

### Privacy-oriented execution

```text
Email body
    |
    v
Android device
    |
    +--> preprocessing
    |
    +--> tokenizer
    |
    +--> INT8 ONNX model
    |
    +--> BIO decoding
    |
    +--> validation
    |
    v
Job title
```

The email text does not need to leave the device.

---

## Evaluation

The final INT8 model was evaluated against a **private frozen 100-case application benchmark**.

Results:

```text
98 / 100 expected extraction outcomes
```

Detailed outcome:

```text
Correct extraction outcomes: 98
False positives:              0
Missed titles:                0
Span differences:             2
```

The two span differences were:

```text
Case 007
Expected:
Tech Lead - Mobile Application Developer

Predicted:
Mobile Application Developer
```

and:

```text
Case 012
Expected:
Engineering Manager (all genders)

Predicted:
Position Engineering Manager
```

These are extraction-span differences rather than missed-title or false-positive cases.

### Token-classification metrics

The validation metrics for the token-classification model were:

```text
F1:        0.876543
Precision: 0.835294
Recall:    0.922078
```

These results come from the project's private development/evaluation data.

They should not be interpreted as a guarantee of general-world accuracy.

---

## Public model files

The deployable model is distributed separately through Hugging Face:

**https://huggingface.co/sundeepmallick/sun-job-title-extractor-en-de**

The public model repository contains:

```text
model/
    job_title_model_int8.onnx

tokenizer/
    tokenizer.json

config/
    labels.json

evaluation/
    evaluation_summary.json

README.md
```

The GitHub repository contains the engineering documentation and integration information.

---

## Reproducibility

The public Android model artifacts have been verified against the production artifacts used by the Android application.

### INT8 ONNX model

SHA-256:

```text
6f9c5380a5aca4e1a0993195c6005090fe868c42bef9ae3569771c4e343bbd7f
```

### Tokenizer

SHA-256:

```text
e37bb14acbcb00d7bdea8f88d50926edcc5df78cc3c5bc84ff84120f82f9d741
```

The public tokenizer does not contain the earlier truncation configuration that forced a fixed 256-token sequence. Candidate windowing is handled by the application pipeline.

---

## Repository structure

```text
.
├── docs/
│   ├── android-integration.md
│   └── architecture.md
├── evaluation/
│   └── README.md
├── model/
│   ├── README.md
│   └── MODEL_CARD_DRAFT.md
├── tokenizer/
│   └── README.md
├── PRIVATE_DATA_POLICY.md
├── RELEASE_POLICY.md
├── LICENSE
└── NOTICE
```

The actual deployable model weights are distributed through Hugging Face rather than stored in this Git repository.

---

## Privacy

The original model-development dataset consists of private recruitment email content.

It is **not included** in this public repository or the public Hugging Face model repository.

The public release does not redistribute:

* real email bodies
* real email subjects
* private recruiter information
* private employer information derived from recruitment emails
* application identifiers
* private URLs
* private training data
* private validation data
* private benchmark cases

Examples in the public documentation are synthetic or generic.

See:

```text
PRIVATE_DATA_POLICY.md
```

for the project's public-data policy.

---

## Model provenance

The model is a fine-tuned and quantized derivative of:

```text
Microsoft Multilingual-MiniLM-L12-H384
```

The public release also uses:

* Hugging Face Tokenizers
* ONNX Runtime
* Rust `jni`

Third-party licenses and attribution are documented in:

```text
NOTICE
```

The model is not a newly trained foundation model. It is a task-specific fine-tuned model derived from the Microsoft multilingual MiniLM base model.

---

## Intended use

This model is intended for:

* job-title extraction
* recruitment-email processing
* private/offline applications
* Android applications
* multilingual NLP experimentation
* research and engineering projects

---

## Not intended for

This model is **not** intended for:

* hiring decisions
* candidate ranking
* candidate suitability scoring
* employment outcome prediction
* deciding whether someone should be hired
* automated employment decisions
* legal advice
* employment advice

The model extracts text from recruitment-related messages. It does not determine whether a candidate is suitable for a job.

---

## Engineering highlights

This project demonstrates an end-to-end mobile NLP workflow:

* multilingual NLP
* token classification
* BIO labeling
* model fine-tuning
* weighted token loss
* ONNX export
* INT8 quantization
* tokenizer integration
* Rust/JNI Android integration
* on-device inference
* deterministic candidate preprocessing
* deterministic validation
* regex fallback
* Android instrumentation testing
* Android 16 KB native-library compatibility
* privacy-oriented architecture
* public model distribution

---

## Related projects

### Deployable model

**Hugging Face**

https://huggingface.co/sundeepmallick/sun-job-title-extractor-en-de

### Android application

The model is integrated into the **SunJob** Android application for local job-email processing.

---

## License

The project-specific source code and documentation are released under the **MIT License**.

The model is derived from third-party components and remains subject to their applicable licenses and attribution requirements.

See `NOTICE` for third-party attribution.

---

## Status

**Public model release available.**

The current public release contains the validated INT8 ONNX model, matching tokenizer, label configuration, evaluation summary, and model documentation.

The original private recruitment-email dataset remains private and is not part of the public release.
