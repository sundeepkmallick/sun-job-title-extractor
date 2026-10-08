# Public Release Policy

## Privacy

This repository must never contain private recruitment email data.

The following must not be committed:

- Real email subjects or bodies
- Personal names extracted from private emails
- Email addresses
- Recruiter information
- Employer information derived from private emails
- Job application identifiers
- Private URLs
- Private datasets
- Private validation or test datasets
- Internal BIO datasets derived from private emails

## Public examples

Examples included in this repository must be synthetic or otherwise
safe for public distribution.

## Model artifacts

The trained model may be distributed publicly when the release audit
has confirmed that the model artifact, tokenizer and accompanying
documentation contain no intentionally embedded private email content.

## Evaluation

Private evaluation data remains private.

Public documentation may describe aggregate evaluation results, but
must not disclose the underlying private email dataset.

## Android application

The production SunJob application remains a separate private project.

This public repository contains engineering and documentation material
for the job-title extraction model and its integration approach, not
the user's private email corpus.
