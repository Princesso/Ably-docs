# Ably docs assessment on AI Transport reconnection and recovery

This repo contains my rewrite of the [Reconnection and recovery](https://ably.com/docs/ai-transport/features/reconnection-and-recovery) page from Ably's AI Transport documentation. The task was to identify up to three issues with the original page and produce an improved version.

## How to read this repo

The files below follow the order of the work, from analysis through to the final output.


### 1. `focus-areas.md` — What I found and why I changed it

Start here. This document identifies the three issues I found in the original page and explains the rationale behind each change. Reading this first gives context for every decision made in the rewrite.


### 2. `ai-transcript-and-prompt.md` — The AI-assisted drafting process

This documents the AI tool used (Claude, via a custom `/ably-docs` skill), the full prompt that generated the initial draft, and a numbered list of every manual revision I made after the AI output. It shows where AI accelerated the work and where my judgment overrode it.


### 3. `reconnection-and-recovery.md` — The rewritten page

This is the deliverable. It is a full MDX rewrite of the original documentation page, ready to drop into the Ably docs site. It addresses the three issues from `focus-areas.md`: undefined terms, unquantified recovery thresholds, and sparse/disconnected code samples.


### 4. `ably-docs-updated.skill` — The Claude skill used during drafting

This is the custom Claude Code skill file that encodes Ably's writing style guide, terminology standards, and documentation quality checklist. It was loaded into the Claude session that generated the initial draft. It is included here for transparency and reproducibility.
