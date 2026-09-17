# Short-Form Video Automation Factory — Generic Workflow v0.1

**Purpose:** a reusable, product-agnostic workflow for turning a content idea into an approved, measurable short-form video.

**Status:** design-only. This document contains no brand, product, audience, or account-specific information.

## 1. Design principles

1. **Separate the workflow from the profile.** Brand facts, product facts, audience details, providers, and credentials belong in a user-owned profile/config file, never in the workflow core.
2. **One video, one promise.** A short video may contain several scenes, but it should have one primary idea and one primary action.
3. **Structured artifacts over giant prompts.** Each stage reads and writes typed JSON plus media files.
4. **Human approval before public publishing.** Automation prepares and queues content; it does not silently publish by default.
5. **Cache expensive stages.** A small copy edit should not regenerate audio, avatar video, or every frame.
6. **Use adapters around providers.** TTS, avatar, storage, publishing, and analytics providers must be replaceable.
7. **Measure outcomes, not activity.** Views are useful, but retention, saves, shares, clicks, and conversions matter more.
8. **Build the narrowest vertical slice first.** One reliable video path beats a large dashboard full of decorative machinery.

## 2. Scope

This workflow supports short-form vertical video for platforms such as:

- Instagram Reels;
- TikTok;
- YouTube Shorts;
- other platforms with a compatible publishing adapter.

The default production shape is a 9:16 video assembled from any combination of:

- programmatic motion graphics;
- product or app demonstrations;
- screen recordings;
- text and diagrams;
- synthetic avatar clips;
- user-provided footage;
- stock or generated footage when the profile permits it.

No one visual style is mandatory. The profile decides which visual modes are allowed.

## 3. Profile/config boundary

The reusable workflow must not know who the user is. A profile supplies the details.

Example profile:

```yaml
# profile.example.yaml
brand:
  name: "YOUR_BRAND"
  website: "https://example.com"
  tone: [clear, specific, human]
  colors: []
  fonts: []
  logo_path: "assets/logo.svg"

product:
  description: "What the product does in one sentence."
  approved_claims: []
  prohibited_claims: []
  proof_assets: []

audience:
  primary: "Describe the main viewer."
  pains: []
  awareness_stage: "problem_aware"

content:
  default_duration_seconds: 30
  default_language: "en"
  default_cta: "YOUR_CALL_TO_ACTION"
  allowed_visual_modes: [motion_graphics, product_demo, avatar]
  forbidden_visual_modes: []

providers:
  llm: "structured-output-provider"
  tts: "tts-provider"
  avatar: "optional-avatar-provider"
  publisher: "manual-review"
  analytics: "manual-snapshots"

secrets:
  # Names only. Values belong in environment variables or a secret manager.
  llm_key_env: "LLM_API_KEY"
  tts_key_env: "TTS_API_KEY"
  avatar_key_env: "AVATAR_API_KEY"
  publisher_key_env: "PUBLISHER_API_KEY"
```

The profile may be committed if it contains placeholders and public brand information only. Secrets never belong in it.

## 4. End-to-end pipeline

```text
content idea / source material
          ↓
   1. Brief and claim check
          ↓
   2. Hook and script generation
          ↓  [human script approval]
   3. Visual script planning
          ↓
   4. Audio generation and timing
          ↓
   5. Optional avatar/media generation
          ↓
   6. Programmatic composition
          ↓
   7. Technical and creative QA
          ↓  [human video approval]
   8. Review queue
          ↓
   9. Platform publishing adapter
          ↓
  10. Metrics collection
          ↓
  11. Learning and next-test suggestion
```

Each stage receives a known input and produces a known output. A failure should identify the stage that failed rather than forcing a complete restart.

## 5. Inputs and outputs

### Per-video input

A content brief contains:

- target audience;
- viewer problem or desire;
- one product, service, idea, or lesson to feature;
- one approved proof point;
- content angle;
- primary CTA;
- target platform;
- target duration;
- language and voice direction;
- allowed visual modes;
- optional source links or files.

### Per-video output package

```text
jobs/<job-id>/
├── brief.json
├── claims.json
├── script.json
├── visual-script.json
├── audio/
│   ├── segments/
│   ├── full-voice.mp3
│   └── timings.json
├── media/
│   ├── avatar/
│   ├── footage/
│   ├── screenshots/
│   └── generated/
├── render/
│   ├── video.mp4
│   ├── cover.png
│   └── poster-frame.png
├── publish/
│   ├── caption.txt
│   ├── hashtags.txt
│   └── metadata.json
├── metrics/
├── qa.json
├── approvals.json
└── manifest.json
```

The folder is a suitable V1 job store. A database or dashboard can be added after the workflow has been used successfully.

## 6. Stage 0 — Create the content brief

The user or an upstream content source supplies an idea. The brief builder reduces it to one clear video opportunity.

Required questions:

1. Who is this for?
2. What problem, desire, question, or moment is relevant?
3. What is the single promise of this video?
4. What evidence supports the promise?
5. What should the viewer do next?

Reject or flag:

- multiple competing promises;
- claims with no evidence;
- features or facts not present in the profile;
- vague superlatives such as “best” or “instant” without proof;
- a CTA that does not match the actual user journey.

**Output:** `brief.json` and `claims.json`.

**Done when:** one audience, one promise, one proof path, and one CTA are explicit.

## 7. Stage 1 — Generate hooks and script

The scriptwriter selects a structure appropriate to the audience and angle.

Supported structures:

- **PAS:** problem → agitation → solution;
- **BAB:** before → after → bridge;
- **AIDA:** attention → interest → desire → action;
- **Hook–Story–Offer:** hook → one focused narrative → offer;
- **Tutorial:** promise → steps → result → CTA;
- **Proof-led:** claim → demonstration → explanation → CTA.

The script is a sequence of beats, not a paragraph.

```json
{
  "schema_version": "0.1",
  "duration_target_seconds": 30,
  "structure": "hook_story_offer",
  "hook_family": "concrete_promise",
  "beats": [
    {
      "id": "hook",
      "purpose": "stop_scroll",
      "voice": "A short spoken opening.",
      "on_screen": "SHORT OPENING TEXT",
      "visual_mode": "avatar_or_kinetic_text",
      "claim_ids": []
    },
    {
      "id": "body",
      "purpose": "explain_or_demonstrate",
      "voice": "The one useful explanation or demonstration.",
      "on_screen": "SHORT SUPPORTING TEXT",
      "visual_mode": "product_demo",
      "claim_ids": ["claim-001"]
    },
    {
      "id": "cta",
      "purpose": "ask_for_action",
      "voice": "One clear next action.",
      "on_screen": "ONE CLEAR CTA",
      "visual_mode": "brand_card_or_avatar",
      "claim_ids": []
    }
  ]
}
```

Script rules:

- write for speech, not essays;
- remove throat-clearing;
- avoid stuffing every feature into one video;
- keep the opening understandable without context;
- make the proof arrive before the viewer loses patience;
- use a single CTA;
- mark every factual claim with a `claim_id`.

**Human gate A:** the script must be approved before paid or slow media generation.

**Done when:** the script has timestampable beats, a clear hook, supported claims, and one CTA.

## 8. Stage 2 — Create the visual script

The visual director maps each beat into a scene.

Every beat specifies:

1. spoken audio;
2. on-screen text;
3. visual mode;
4. motion or change;
5. purpose of the visual;
6. asset references;
7. caption placement;
8. transition or timing notes.

Common scene types:

1. **AvatarHook** — short synthetic presenter clip.
2. **KineticStatement** — animated type and emphasis.
3. **ProductDemo** — UI, screen recording, or product interaction.
4. **Diagram** — process, comparison, or concept visual.
5. **ProofCard** — verified result, quote, number, or before/after.
6. **ListOrSteps** — numbered explanation.
7. **Footage** — optional stock, generated, or user-provided clip.
8. **BrandCTA** — final action card.

The profile controls which scene types are allowed. A workflow can be entirely motion-graphics-based, avatar-led, footage-led, or hybrid.

Visual rules:

- show the promise immediately;
- make captions available from the first spoken word;
- keep important text inside platform-safe margins;
- change the visual state when the idea changes;
- show the product or evidence instead of only naming it;
- never use an avatar or visual asset without a narrative purpose.

**Output:** `visual-script.json`.

**Done when:** every spoken beat has an intentional visual and every required asset is identified.

## 9. Stage 3 — Generate audio

The audio adapter receives the approved voice text and returns:

- audio file(s);
- provider/model metadata;
- duration;
- word- or phrase-level timing data when available.

Chunking audio by beat or scene is preferred because it enables:

- smaller provider requests;
- independent caching;
- targeted regeneration after edits;
- easier avatar lip-sync;
- simpler timeline debugging.

If the TTS provider does not return reliable timing data, use a separate alignment tool. Never estimate caption timing from character count alone.

**Commercial-use gate:** the user must verify that the selected voice model and plan permit the intended use.

**Done when:** every voice beat has playable audio and timing data that matches the spoken words.

## 10. Stage 4 — Generate optional avatar or media clips

This stage is optional. The workflow must be able to produce a complete video without an avatar provider.

An avatar adapter receives:

- a portrait or character input;
- a short audio clip or script;
- requested resolution;
- model/version settings.

A media adapter may receive:

- a text prompt;
- a reference image;
- a source clip;
- a duration and aspect ratio.

Provider rules:

- call only for scenes marked as requiring that provider;
- cache by all meaningful inputs and model version;
- enforce a per-job cost ceiling;
- record provider job IDs and actual costs;
- provide a lower-cost or faceless fallback;
- never regenerate unchanged inputs.

**Done when:** every required media asset exists, passes basic format checks, and has provenance metadata.

## 11. Stage 5 — Compose with Remotion or another renderer

The renderer receives an immutable render payload containing:

- canvas size and frame rate;
- scene list and props;
- audio files and timing data;
- media asset paths;
- caption style;
- brand tokens from the profile;
- transition and motion settings;
- output platform requirements.

The renderer owns deterministic composition. It should not invent marketing claims or silently rewrite the script.

Remotion is a strong default when the workflow needs:

- reusable React scene components;
- data-driven layouts;
- animated captions;
- product UI demonstrations;
- generated charts or diagrams;
- batch rendering from JSON.

FFmpeg or an equivalent media tool performs technical post-processing:

- codec validation;
- audio-stream checks;
- metadata handling;
- compression;
- poster-frame extraction;
- duration and file-size reporting.

**Done when:** the same render payload produces a valid, repeatable video and cover image.

## 12. Stage 6 — Automated QA

QA should fail loudly instead of shipping a suspicious file.

### Technical checks

- video exists and opens;
- dimensions match the platform profile;
- audio exists and is not silent;
- duration is within limits;
- required assets and fonts exist;
- captions remain inside safe areas;
- no blank frames or missing scenes;
- output file size is acceptable;
- cover image exists;
- provider outputs match expected media types.

### Content checks

- opening promise is present;
- every factual claim maps to an approved claim;
- the product, source, or evidence is visible where required;
- CTA is singular and understandable;
- captions are readable without sound;
- visual mode follows the profile policy;
- required disclosures are present;
- no unintended watermarks or provider branding remain.

### Human review checklist

- Would the intended viewer stop for the opening?
- Is the explanation understandable without extra context?
- Does the visual support the spoken line?
- Does anything look synthetic, confusing, or embarrassing?
- Is the promise credible?
- Is the CTA worth acting on?

**Output:** `qa.json` with pass/fail status and failure reasons.

**Done when:** all automated checks pass and the package is ready for human review.

## 13. Stage 7 — Review queue

A folder-first queue is enough for V1:

```text
review/
├── ready/<job-id>/
├── approved/<job-id>/
├── changes-requested/<job-id>/
└── published/<job-id>/
```

A review package contains:

- video;
- cover;
- caption;
- target platform;
- creative summary;
- estimated and actual provider costs;
- QA report;
- approval metadata.

Useful rejection reasons:

- `hook_weak`;
- `claim_unsupported`;
- `visual_unclear`;
- `caption_too_fast`;
- `audio_bad`;
- `avatar_bad`;
- `brand_mismatch`;
- `cta_wrong`;
- `technical_failure`.

A rejection should invalidate only the affected downstream artifacts. A caption fix should not regenerate the avatar.

**Human gate B:** public publishing requires explicit approval.

**Done when:** the reviewer can approve or reject without opening project source files.

## 14. Stage 8 — Publishing adapter

Publishing is platform-specific and optional.

A publisher should:

1. verify the job is approved;
2. verify caption, cover, and video are final;
3. upload or reference a publicly fetchable media URL;
4. create the platform media object/container;
5. wait for processing;
6. publish once;
7. store the platform post ID and URL;
8. record the response and timestamp;
9. schedule metrics collection;
10. protect against duplicate publishing.

Before an official API is configured, the workflow should export a manual publishing package. Manual publishing is a valid V1 mode.

**Done when:** an approved package can be published once or exported for manual posting, with a recoverable record of what happened.

## 15. Stage 9 — Analytics and learning

### V1 — manual snapshots

After posting, record:

- post URL and ID;
- publish time;
- creative metadata from the job manifest;
- metrics at defined checkpoints, such as 24 hours, 48–72 hours, and 7 days.

### V2 — platform insights

When the platform API is available, fetch metrics for the stored post ID. Keep both:

- the raw provider response;
- a normalized provider-independent record.

Metric names and availability vary by platform and API version. Missing data must be recorded as `unavailable`, not silently converted to zero.

### V3 — first-party conversion tracking

Use a unique UTM link or redirect for each creative version and read meaningful actions from the product's own analytics:

- landing-page visits;
- signups;
- activation;
- purchases;
- retained usage;
- other product-specific outcomes.

The workflow should compare one major creative variable at a time where possible:

- hook family;
- audience problem;
- visual mode;
- presenter/avatar usage;
- duration;
- CTA;
- offer.

Useful derived measures include:

- early-hold rate;
- average-watch-to-duration ratio;
- save rate;
- share rate;
- interaction rate;
- profile or landing-page visit rate;
- signup or conversion rate.

The system should suggest the next test and show sample size. It should not announce a universal winner from one or two posts.

Recommended files:

```text
jobs/<job-id>/metrics/
├── raw-<timestamp>.json
├── normalized.json
└── lesson.json
```

**Done when:** every published job has metrics or an explicit `insufficient_data` result and a next-test suggestion.

## 16. State machine

```text
IDEA
  → BRIEFED
  → SCRIPT_DRAFT
  → SCRIPT_APPROVED
  → AUDIO_READY
  → MEDIA_READY
  → RENDERED
  → QA_PASSED
  → READY_FOR_REVIEW
  → APPROVED
  → PUBLISHING
  → PUBLISHED
  → METRICS_PENDING
  → LEARNED
```

Recoverable failure states:

```text
*_FAILED → retry the same stage
*_CHANGES_REQUESTED → edit the relevant input and invalidate downstream artifacts
```

Every job should record:

- stable job ID;
- schema version;
- input hashes;
- provider/model versions;
- estimated and actual costs;
- timestamps;
- approvals;
- errors;
- output locations.

## 17. Tool boundaries

| Component | Owns | Must not own |
|---|---|---|
| Structured-output LLM | brief, hooks, script, visual plan, caption draft | unsupported claims or silent publishing |
| TTS adapter | narration and timing data | visual timing assumptions |
| Avatar/media adapter | optional generated clips | full composition or creative approval |
| Remotion/renderer | deterministic scenes, layout, motion, captions | marketing strategy or claims |
| FFmpeg/media tools | encoding and inspection | creative decisions |
| Local job store | artifacts, manifests, state | plaintext secrets |
| Publishing adapter | approved platform posting | script generation or approval |
| Analytics adapter | metric retrieval and normalization | claiming causality from tiny samples |
| Workflow runner | ordering, state, retries, caching, approvals | being the creative brain |

## 18. Reusable skill/plugin boundaries

A public implementation can expose these generic components:

1. `brief-builder`
2. `hook-generator`
3. `scriptwriter`
4. `visual-director`
5. `brand-and-claim-guard`
6. `audio-producer`
7. `avatar-and-media-producer`
8. `remotion-director`
9. `reel-qa`
10. `review-publisher`
11. `analytics-learner`
12. `workflow-runner`

Each component should have a narrow job and a documented input/output contract. Brand and product details are supplied by the profile, not hard-coded into these components.

## 19. Implementation order

### Slice 1 — dry run

Generate a brief, claims list, script, visual script, caption, and fake render payload without external API calls.

**Acceptance:** a human can review a complete content package.

### Slice 2 — renderer

Build one reusable composition with configurable scenes, captions, brand tokens, and media placeholders.

**Acceptance:** a JSON payload renders a valid vertical video locally.

### Slice 3 — audio adapter

Add TTS generation, caching, and timing validation.

**Acceptance:** voice and captions align reliably.

### Slice 4 — optional media adapters

Add avatar, image, or footage providers behind replaceable interfaces.

**Acceptance:** a provider failure produces a clear error or configured fallback.

### Slice 5 — QA and review

Add automated checks, package generation, and approval folders.

**Acceptance:** a reviewer can approve or reject without editing source code.

### Slice 6 — manual publishing package

Export video, cover, caption, and metadata for manual posting.

**Acceptance:** a person can publish without hunting through project files.

### Slice 7 — platform publisher

Add one official publishing adapter at a time.

**Acceptance:** approved media publishes once with duplicate protection and a stored post ID.

### Slice 8 — analytics

Start with manual snapshots, then add platform insights and first-party conversion tracking.

**Acceptance:** each post is linked to its creative metadata and next test.

### Slice 9 — thin workflow runner

Automate stage ordering, caching, retries, approvals, and recovery only after the individual stages are stable.

**Acceptance:** a job can resume from the last completed stage without duplicating paid work or publishing.

## 20. Explicit non-goals for the first release

- fully autonomous public posting;
- every social platform at once;
- a dashboard before the folder workflow works;
- full-length avatar videos by default;
- expensive generative video for every scene;
- automatic claims about virality;
- training a custom model;
- storing secrets in profile files;
- a giant agent that makes every creative and operational decision.

## 21. Safety and quality safeguards

- use synthetic or properly licensed portraits and footage;
- verify commercial rights for every provider and model;
- show AI/synthetic-media disclosures where required;
- keep claims tied to approved evidence;
- record provider costs and enforce per-job budgets;
- use official publishing APIs where possible;
- protect against duplicate posts;
- retain raw provider responses for debugging;
- allow a complete non-avatar fallback;
- treat small samples as hypotheses, not proof.

## 22. Public plugin packaging

A GitHub-ready implementation can use this structure:

```text
short-form-video-factory/
├── README.md
├── WORKFLOW.md
├── SKILL.md
├── templates/
│   ├── profile.example.yaml
│   ├── brief.example.json
│   ├── script.example.json
│   └── visual-script.example.json
├── references/
│   ├── provider-adapters.md
│   ├── platform-publishing.md
│   └── analytics.md
├── scripts/
│   ├── validate-job.py
│   ├── inspect-render.py
│   └── package-review.py
└── tests/
```

`WORKFLOW.md` is the provider-agnostic process. `SKILL.md` is the agent-facing instruction layer. Templates hold user-specific inputs without contaminating the reusable core. Provider implementations and platform rules belong in references or adapters.

## 23. Definition of done

The reusable workflow is ready for a first public pilot when:

- a new user can fill the profile template without editing the core workflow;
- one brief produces a structured script and visual plan;
- a renderer can produce a valid vertical draft;
- optional providers can be swapped without rewriting the workflow;
- a human can review and approve the package;
- publishing can be manual before an API is configured;
- metrics can be recorded and linked to the creative metadata;
- no personal brand, product, audience, credential, or account detail is required by the core files.
