---
name: h3-prompt-master
description: Create MiniMax H3 video prompts and production-ready prompt packages across story animation, brand and product films, co-op game intros, music videos, explainers, live-action fusion, and action scenes. Use mode-specific source guidance for H3 formats, image prompts, shotlists, continuity, and review.
---

# H3-Prompt-Master

Create clear, production-ready prompts and planning materials for MiniMax H3 video generation. This plugin exposes one skill backed by 14 specialist workflows. Use the source material under `references/source-skills/` as the detailed authority for each mode; read only the relevant source guide and linked references for the current request.

## Route the request

1. Identify the requested deliverable: a prompt rewrite, a new H3 prompt, image prompts, a shotlist, a full production package, or prompt review.
2. Select the closest mode below. When modes overlap, combine only the relevant guidance and keep the user's requested output scope.
3. For general H3 authoring, read `references/source-skills/h3-prompt-writing/GUIDE.md` and `references/source-skills/h3-prompt-enhance/GUIDE.md`; consult the latter's references for the selected generation mode, prompt schema, continuity, sound, or validation rules.
4. Read the selected specialist `GUIDE.md` and any references it links before drafting. Source folders are preserved intact so their relative links remain usable.
5. Deliver copy-ready text and state any assumptions that materially affect the result. Do not claim to have generated video, images, or audio unless the active environment actually provides and completes those tools.

## Mode directory

All directory paths below are relative to `references/source-skills/`. Read the selected directory's `GUIDE.md`.

- **General H3 prompt writing, mode selection, rewriting, review, continuity, or multi-clip prompts:** `h3-prompt-writing/`, `h3-prompt-enhance/`.
- **Story-first 3D animated short, from concept through shotlist and production plan:** `3d-animation-short-generator/`.
- **Brand or product promotional film:** `brand-promo-video-generator/`.
- **Co-op game intro with player identities, menu, and opening video:** `co-op-game-intro-generator/`.
- **Hand-drawn animation interacting with live-action footage:** `handdrawn-live-video-generator/`.
- **Dense, continuous fight choreography:** `high-density-fight-prompt/`.
- **Reference-image-driven two-character duel in Ref2VA format:** `ref2va-duel-prompt-generator/`.
- **Image-generation prompt drafting:** `image-prompt-writer-local/`.
- **Minimalist premium product advertisement:** `minimalist-product-ad-generator/`.
- **Music video, lyric typography, or emotional short:** `music-video-subtitle-generator/`.
- **Paper-collage explainer:** `paper-collage-explainer-generator/`.
- **Papercraft stop-motion explainer:** `papercraft-stop-motion-explainer/`.
- **Screenplay breakdown, cinematic shotlist, or self-contained H3 prompt sheet:** `tudou-shotlist-builder-beta/`.

## Shared output practices

- Preserve the selected mode's required prompt fields, language, timing, reference labels, and output structure. Do not flatten a structured prompt into generic prose.
- Keep visual identity, geography, action, timing, camera, performance, dialogue, and sound internally consistent. For multiple clips, carry forward only the continuity state needed by the next clip.
- When references are provided, map them to the correct characters, objects, scenes, or audio roles. Never invent an unseen reference or claim a reference was analyzed if it was not accessible.
- Keep the final prompt easy to copy. Separate director notes, assumptions, negative constraints, and post-production guidance when the selected mode calls for them.
- If the user asks only for a prompt, do not expand the answer into a complete production workflow. For a broad end-to-end request, follow the selected specialist workflow and provide its planned artifacts.

## GPT environment compatibility

This skill is text-first and does not require MiniMax Hub. Some original specialist workflows describe Hub-only image/video generation, canvas operations, or media analysis. In GPT environments without those tools, perform the planning and prompt-writing portions, identify which media step requires an external tool, and provide a ready-to-use prompt or production handoff. Use available image, audio, video, or file tools only when they are present and appropriate.

The 14 source workflows, including their references, templates, and scripts, are retained under `references/source-skills/`. They are supporting material for this single plugin skill. Their former entrypoints are named `GUIDE.md` to keep only one callable skill.



