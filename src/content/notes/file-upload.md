---
title: Designing a Resilient Multi-file Upload System
---

## Context
Why we needed multi-file upload with progress.

## First naive attempt
What broke / felt wrong.

## Architecture choice
Direct-to-S3 with signed URLs.

## Frontend concerns
- per-file progress
- cancellation
- partial failures

## Backend responsibilities
- signing
- validation
- download access

## What I’d improve next
Things postponed or still rough.
