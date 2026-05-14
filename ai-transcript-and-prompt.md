# AI transcript and prompt notes

## AI tool used

I used Claude (Anthropic), via Claude.ai with a custom `/ably-docs` skill that encodes Ably's official writing style guide, terminology standards, and documentation quality checklist.

## Prompt used

The following prompt was used to generate the initial draft, which I then manually revised:


```
You are rewriting an Ably AI Transport documentation page to publishable standard. 
Follow the Ably writing style guide exactly. Here are the enforced rules:

VOICE AND LANGUAGE:
- Second person, present tense ("You create a channel")
- Active voice (Subject-Verb-Object)
- No filler phrases ("it should be noted", "in order to", "simply")
- No AI-generated patterns ("delve", "dive into", "landscape", "it's worth noting")
- No em-dashes. Use standard hyphens or restructure the sentence
- No bold prefixes in bullet points (no "**Feature:** Description" patterns)
- No Latin abbreviations. Use "for example" not "e.g.", "that is" not "i.e."
- No subjective phrases ("easily", "simply", "just")
- No vague language ("would", "should", "might", "maybe")
- Write small numbers as words ("two minutes" not "2 minutes")

TERMINOLOGY:
- "realtime" (one word, lowercase unless starting a sentence)
- "publish" and "subscribe" (not send/listen/emit)
- "channel" (not topic/stream/room)
- JavaScript (not Javascript), GitHub (not Github)

HEADINGS:
- Sentence case (first letter capitalized)
- Imperative form ("Configure the API client" not "Configuring the API client")
- All headings must be followed by introductory text before any code or lists

CODE SAMPLES:
- Every sample must be complete and runnable
- Single quotes for JS/TS strings (not double quotes, except JSON)
- Realistic values (not "foo", "bar", "test")
- Precede code blocks with a colon, not a period
- Specify the language attribute on every code block

STRUCTURE:
- Lead with what the reader accomplishes, not background theory
- Explain the "why", not just the "what"
- Be generous with paragraph breaks
- End with 2-3 related feature links

PAGE TO REWRITE:
[Full markdown content of https://ably.com/docs/ai-transport/features/reconnection-and-recovery.md was provided]

ISSUES TO FIX (maximum three):

1. KEY TERMS USED BUT NEVER DEFINED: The page uses "untilAttach", "durable session", 
   "persistence window", "encoder", and "lifecycle tracker" without explaining what 
   they are or linking to definitions. A developer new to AI Transport hits "untilAttach" 
   in step 3 of "How it works" with no context for what it means.

2. RECOVERY SCENARIOS LACK SPECIFICITY: The section describes "brief" vs "longer period" 
   disconnections but never quantifies the threshold, never specifies the persistence 
   window duration, and provides no code for either path. The reader cannot answer: 
   "How long is too long, and what do I need to handle?"

3. CODE SAMPLES ARE SPARSE AND DISCONNECTED: The only substantive code shows the 
   useView React hook for history loading, not reconnection or recovery. There is no 
   code for the encoder recovery path on the server, no code for mid-stream joins, and 
   the existing samples do not connect to the recovery scenarios described in prose.

ADDITIONAL CONTEXT:
- untilAttach is a boolean parameter on Ably history requests. When true, it returns 
  messages up to the point of channel attachment, ensuring gapless continuity between 
  historical and live messages.
- Brief disconnections are under two minutes; Ably's connection protocol replays 
  missed messages automatically.
- Channel history persists for 24-72 hours depending on the Ably plan.
- The encoder is part of the server transport's codec pipeline, used inside 
  turn.streamResponse().
- The lifecycle tracker is part of the client transport that manages turn lifecycle 
  events for late-joining clients.

Rewrite the page now. Output clean MDX.
```


## What I manually changed after the AI draft

1. Defined "agent" on first use as "the server-side agent (the component that calls your LLM and publishes responses)" so a developer new to AI Transport knows what this term refers to. Similarly, specified "Ably realtime client SDK" rather than the vague "the SDK" throughout.

2. Restored the "How it works" section with a step-by-step sequence. The original page had this and it provides a valuable mental model of the reconnection flow before the reader hits the detailed recovery paths. Removing it was a mistake in the initial draft.

3. Restructured "Recovery scenarios" into two clearly labeled subsections with H3 headings ("Brief disconnections" and "Extended disconnections") instead of prose-only paragraphs, so the reader can scan directly to their situation.

4. Added the specific persistence window range ("24 to 72 hours, depending on your Ably plan") sourced from the ably-ai-sdk-transport GitHub repository documentation rather than leaving it vague.

5. Added a server-side code sample to the "Encoder recovery" section showing `turn.streamResponse()` in context with `streamText`, so the reader sees where encoder recovery actually happens. The original page described the recovery in prose but showed no code.

6. Expanded the "Mid-stream joins" section to explain *why* it works (the session state does not depend on any single client being connected) rather than just stating what happens.

7. Removed the standalone "Load history on reconnect" section and moved its content into the "Extended disconnections" subsection, because loading history is the recovery mechanism for extended disconnections. Having it as a separate section disconnected from the recovery scenario it serves.

8. Defined `untilAttach` inline on first use as "a parameter that tells Ably to return all messages up to the exact point where the client reattached" rather than assuming the reader knows what it means.

9. Applied Ably style guide throughout: removed all em-dashes, used single quotes in JS code samples, wrote numbers as words ("two minutes"), used imperative headings in sentence case, and ensured every heading is followed by introductory text before code or lists.
