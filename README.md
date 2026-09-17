# Short-Form Video Automation Factory

A provider-agnostic workflow for turning a content brief into an approved, measurable short-form video.

## What this repository contains

- `WORKFLOW.md` — the complete production workflow.
- `templates/profile.example.yaml` — user-specific configuration template.

## Core idea

```text
brief → script → visual plan → audio/media → render → QA → human review → publish → analytics
```

The workflow is intentionally separated from the user's brand, product, audience, providers, and credentials. Each user fills a profile instead of editing the workflow core.

## Current status

This is a workflow specification, not a finished rendering application or publishing integration.

The recommended implementation order is:

1. dry-run structured content package;
2. Remotion renderer;
3. audio adapter;
4. optional avatar/media adapters;
5. QA and review queue;
6. manual publishing package;
7. platform publisher;
8. analytics adapter;
9. thin workflow runner.

## Security

Copy `templates/profile.example.yaml` to `profile.yaml`, but keep secret values in environment variables or a secret manager. Do not commit API keys, access tokens, generated media, private briefs, or review packages.

## Provider flexibility

The workflow does not require one LLM, TTS provider, avatar model, storage provider, publishing platform, or analytics service. Adapters should implement the required interfaces so providers can be replaced without rewriting the workflow.
