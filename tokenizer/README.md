# Tokenizer

## Purpose

This directory documents the tokenizer used by the job-title extraction model.

The tokenizer is an important part of the model deployment because the token IDs supplied to the model must correspond to the tokenizer behavior used during model training.

## Tokenizer format

The production tokenizer is distributed as:

```text
tokenizer.json
```

The tokenizer is based on the tokenizer associated with:

```text
microsoft/Multilingual-MiniLM-L12-H384
```

## Android integration

The Android application loads the tokenizer from:

```text
app/src/main/assets/job_title/tokenizer.json
```

The Android implementation uses the Hugging Face Rust `tokenizers` library through a small JNI bridge.

The native Android library is:

```text
libsunjob_tokenizer.so
```

for the ARM64 Android build.

## Why the tokenizer is part of the model release

A token-classification model does not operate directly on arbitrary text strings.

The text must first be converted into the token IDs expected by the model.

Conceptually:

```text
Text
  |
  v
Tokenizer
  |
  v
Input IDs
  |
  v
Model
  |
  v
BIO labels
```

Using an incompatible tokenizer can produce incorrect token IDs and therefore incorrect model output.

The tokenizer should therefore be treated as part of the deployable model package.

## Tokenization requirements

The Android tokenizer must preserve the behavior expected by the trained model, including:

* vocabulary
* token IDs
* special tokens
* tokenization rules
* token-to-text alignment
* offset information where required
* attention-mask behavior

The Android implementation includes tokenizer-equivalence testing against the reference tokenizer used during development.

## Sequence length

The model was trained with:

```text
maximum sequence length = 256
```

Longer candidate text is handled through application-level windowing.

The tokenizer configuration used by the Android application must not prematurely truncate the complete candidate before the application's windowing logic has an opportunity to create the required model-sized windows.

## Tokenizer and BIO labels

The BIO labels are associated with token positions.

For example:

```text
Tokens:

Senior
Android
Engineer

Labels:

B-JOB_TITLE
I-JOB_TITLE
I-JOB_TITLE
```

The decoder uses these token-level labels to reconstruct the job-title span.

Correct tokenization is therefore required for correct BIO decoding.

## Public distribution

The public tokenizer artifact may be distributed together with the model artifact.

The public distribution must not contain:

* private recruitment emails
* private benchmark cases
* private training examples
* private validation examples
* private BIO datasets

## Verification

The Android integration includes tests that compare reference tokenizer behavior with the Android tokenizer implementation.

The tokenizer/windowing integration was also tested as part of the Android instrumentation suite.

## License

The tokenizer artifact is distributed subject to the applicable license and attribution requirements of the upstream tokenizer/model components.

The project documentation should preserve the relevant upstream attribution information.
