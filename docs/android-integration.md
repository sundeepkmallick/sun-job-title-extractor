# Android Integration

This document describes how the SunJob job-title extraction model is integrated into an Android application.

## Overview

The model is designed for private, offline job-title extraction from recruitment-related email text.

The Android integration uses:

* Multilingual MiniLM-based token classification
* ONNX Runtime for Android
* INT8 ONNX inference
* Hugging Face Tokenizers through a small Rust/JNI bridge
* BIO decoding
* Candidate preprocessing and windowing
* Deterministic validation
* Regex fallback

The email content used for inference remains on the device.

## Runtime pipeline

```text
Email body
    |
    v
Candidate preprocessing
    |
    v
Candidate text windows
    |
    v
Multilingual MiniLM INT8 ONNX model
    |
    v
BIO token decoding
    |
    v
Deterministic validation
    |
    +--------------------+
    |                    |
    | valid              | invalid/unavailable
    v                    v
ON_DEVICE_MODEL      Regex fallback
    |                    |
    +---------+----------+
              |
              v
        JobTitleResult
```

## Model

The production Android model is an INT8 ONNX token-classification model.

The model uses three labels:

```text
O
B-JOB_TITLE
I-JOB_TITLE
```

The model was fine-tuned from:

```text
microsoft/Multilingual-MiniLM-L12-H384
```

The Android application uses the tokenizer corresponding to the fine-tuned model.

## ONNX Runtime

The Android application uses ONNX Runtime for inference.

The production dependency is:

```text
com.microsoft.onnxruntime:onnxruntime-android:1.30.0
```

Inference is performed locally on the Android device.

No network request is required for job-title extraction.

## Tokenizer integration

The tokenizer is distributed as:

```text
app/src/main/assets/job_title/tokenizer.json
```

The Android application uses a native Rust tokenizer bridge exposed to Kotlin through JNI.

The native library is:

```text
app/src/main/jniLibs/arm64-v8a/libsunjob_tokenizer.so
```

The Rust bridge uses the Hugging Face `tokenizers` crate.

The native tokenizer is used because the production tokenizer must reproduce the tokenizer behavior used during model training.

## Candidate preprocessing

The complete email body is not blindly passed to the model as one large sequence.

The Android preprocessing layer:

1. Converts supported HTML email content into text.
2. Preserves useful text boundaries.
3. Removes quoted reply content where identifiable.
4. Removes signatures where identifiable.
5. Removes only narrowly defined greeting-only content.
6. Identifies recruitment-related context.
7. Deduplicates candidate text.
8. Limits candidate size.
9. Produces candidate windows suitable for model inference.

The purpose is to reduce irrelevant text and improve the probability that a job title appears in the model's input window.

## Token windows

The model was trained with a maximum token length of 256.

Long candidate text is therefore divided into windows before inference.

The tokenizer configuration used by the Android application does not force the entire candidate to be truncated to 256 tokens before the application's windowing logic runs.

This distinction is important:

```text
candidate text
      |
      v
application-level windowing
      |
      +--> window 1
      +--> window 2
      +--> window 3
      |
      v
tokenizer/model inference
```

## BIO decoding

The model produces token-level predictions.

The Android BIO decoder converts these predictions into a job-title span.

Conceptually:

```text
O O B-JOB_TITLE I-JOB_TITLE I-JOB_TITLE O
        |-------------|
        job title
```

The decoded span is then normalized and validated before being returned to the application.

## Deterministic validation

The model is not treated as the final authority on whether an extracted span is usable.

After BIO decoding, deterministic validation checks the extracted candidate.

Invalid or unsuitable candidates are rejected rather than being returned as job titles.

This provides an additional safety layer between model output and application behavior.

## Fallback behavior

The production extractor supports a deterministic regex-based fallback.

The result identifies its source using:

```kotlin
enum class ExtractionSource {
    ON_DEVICE_MODEL,
    REGEX_FALLBACK,
    NONE
}
```

The intended behavior is:

```text
Model available
      |
      v
Model extraction
      |
      +--> valid title --> ON_DEVICE_MODEL
      |
      +--> invalid --> regex fallback

Model unavailable
      |
      v
Regex fallback
```

If neither approach produces a valid title, the result is:

```text
jobTitle = null
source = NONE
```

## Result contract

The application-level result is represented by:

```kotlin
data class JobTitleResult(
    val jobTitle: String?,
    val confidence: Float?,
    val source: ExtractionSource
)
```

The current production implementation does not expose a calibrated probability as application confidence.

Therefore:

```text
confidence = null
```

is expected for the current production path.

## Privacy

The extraction model is intended to operate entirely on-device.

The production architecture does not require sending email content to a remote AI service.

The model therefore does not require:

* an API key
* a cloud inference service
* a remote LLM
* uploading email bodies for inference

Private recruitment emails remain local to the Android application.

## Android 16 KB page-size compatibility

The production APK was checked for Android 16 KB page-size compatibility.

The relevant native ARM64 libraries, including:

```text
libonnxruntime.so
libonnxruntime4j_jni.so
libsunjob_tokenizer.so
```

were verified with 16 KB-compatible ELF load-segment alignment.

The release APK was also verified with:

```bash
zipalign -c -P 16 -v 4 app/build/outputs/apk/release/app-release-unsigned.apk
```

and the verification completed successfully.

## Testing

The Android integration has been tested with instrumentation tests covering:

* BIO decoding
* tokenizer equivalence
* tokenizer windowing
* model inference
* model/tokenizer integration
* complete Android instrumentation behavior

The complete instrumentation suite was successfully executed on both:

* a physical Android device
* an Android emulator

The model's Python/ONNX reference behavior was also compared with Android inference during the integration work.

## Important limitation

The model is a job-title extraction component.

It is not a general-purpose employment classifier and does not determine:

* whether a person was hired
* whether a person will be hired
* whether a job is suitable
* whether an employer is good or bad
* whether an application will succeed
* whether an email represents a legally valid employment decision

The component should be treated as an extraction utility inside a larger application.
