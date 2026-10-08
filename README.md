# Sun Job Title Extractor

A multilingual job-title extraction model designed for privacy-preserving, offline inference in Android applications.

The project focuses on extracting job titles from recruitment-related text using token classification rather than generating job titles.

## What it does

Given recruitment-related text such as:

```text
Thank you for applying for the position of Senior Android Engineer.
```

the system attempts to extract:

```text
Senior Android Engineer
```

The model supports English and German recruitment-related text.

## Architecture

```text
Recruitment email
       |
       v
Candidate preprocessing
       |
       v
Candidate windowing
       |
       v
Multilingual MiniLM token classifier
       |
       v
BIO decoding
       |
       v
Deterministic validation
       |
       +----------------------+
       |                      |
     valid              invalid/unavailable
       |                      |
       v                      v
ON_DEVICE_MODEL          Regex fallback
       |                      |
       +----------+-----------+
                  |
                  v
             JobTitleResult
```

## Model

The model is fine-tuned from:

```text
microsoft/Multilingual-MiniLM-L12-H384
```

The production deployment uses an INT8 ONNX model.

Approximate model sizes:

```text
FP32 ONNX: 448 MB
INT8 ONNX: 113 MB
```

The INT8 model is approximately 75% smaller than the FP32 export.

## Labels

The token classifier uses:

```text
O
B-JOB_TITLE
I-JOB_TITLE
```

## Android

The model is designed for local Android inference using:

* ONNX Runtime
* INT8 ONNX inference
* Hugging Face Tokenizers
* Rust/JNI tokenizer integration
* BIO decoding
* deterministic validation
* regex fallback

The extraction path does not require sending email text to a remote AI service.

## Evaluation

On a private frozen 100-case application benchmark, the final INT8 model achieved:

```text
98 / 100
```

expected extraction outcomes.

The result contained:

```text
98 correct outcomes
0 false positives
0 missed titles
2 extraction-span differences
```

This benchmark is an internal engineering benchmark and is not presented as general-world accuracy.

The token-classification validation metrics were:

```text
F1:        0.876543
Precision: 0.835294
Recall:    0.922078
```

The original training and evaluation data are private and are not included in this repository.

## Privacy

Private recruitment-email content used during model development is not included in the public repository.

The public project must not contain:

* real email bodies
* real email subjects
* private recruiter information
* private employer information derived from recruitment emails
* application identifiers
* private URLs
* private training datasets
* private validation datasets
* private benchmark cases

Examples in public documentation are synthetic.

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
└── RELEASE_POLICY.md
```

The deployable model artifacts are distributed separately.

## Model distribution

The intended public model distribution is hosted on Hugging Face:

```text
sundeepmallick/sun-job-title-extractor-en-de
```

The model distribution is intended to contain the deployable ONNX model, matching tokenizer, label configuration, evaluation summary, and model documentation.

The original private training data is not included.

## Intended use

The model is intended for:

* job-title extraction
* recruitment-email processing
* private/offline Android applications
* research and experimentation with mobile NLP

## Not intended for

The model is not intended for:

* hiring decisions
* candidate ranking
* employment outcome prediction
* suitability scoring
* legal or employment advice
* determining whether a candidate was hired
* determining whether a company will hire someone
* automated decisions about people

## License

The project-specific source code and documentation are released under the MIT License.

The model is based on third-party components and remains subject to their applicable licenses and attribution requirements.

See the model documentation and project notices for additional information.

## Status

This project is an engineering-focused mobile NLP system demonstrating:

* multilingual NLP
* token classification
* BIO labelling
* model fine-tuning
* ONNX export
* INT8 quantization
* tokenizer integration
* Rust/JNI Android integration
* on-device inference
* deterministic validation
* fallback extraction
* Android instrumentation testing
* Android 16 KB native-library compatibility
* privacy-oriented architecture
