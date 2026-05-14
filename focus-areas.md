# Areas of focus and rationale

## Issue 1: A key term was used but never defined

The original page uses `untilAttach`, "durable session", "persistence window", "encoder", and "lifecycle tracker" without explaining what they are. For a developer evaluating or getting started with AI Transport, hitting an undefined term like `untilAttach` in step 3 of a "How it works" list creates an immediate comprehension gap. The reader does not know if it is a method, a parameter, or a configuration option, and the page does not link to a definition.

I defined each term inline on first use. `untilAttach` is explained as a history parameter that ensures gapless continuity. The encoder is introduced as part of the server transport's codec pipeline. The lifecycle tracker is described by its function (synthesizing missed events for late-joining clients) rather than by name alone.

## Issue 2: Recovery scenarios lack specificity and code

The original page describes "brief" and "longer period" disconnections but never quantifies either threshold, never specifies the persistence window duration, and provides no code for either path. A developer reading this section cannot answer the most practical question: "How long is too long, and do I need to handle it differently?"

I restructured this into two clearly labeled subsections with specific thresholds (under two minutes for brief, 24-72 hours for persistence window), added the `useView` code sample directly within the extended disconnection path where it belongs, and clarified that brief disconnections require no application code while extended disconnections use `useView` for history loading.

## Issue 3: Code samples are sparse and disconnected from the prose

The original page's only substantive code shows the `useView` React hook in a standalone "Load history on reconnect" section that is disconnected from the recovery scenarios described above. There is no code showing the encoder recovery path, no code for mid-stream joins, and the existing samples do not map to any specific recovery scenario.

I moved the `useView` code into the extended disconnection subsection so it sits next to the scenario it addresses. I added a server-side code sample to the encoder recovery section showing `turn.streamResponse()` in context with `streamText`, so the reader sees where encoder recovery actually operates. I did not add code for mid-stream joins because that behavior is fully automatic and showing code would imply the developer needs to do something, but I expanded the explanation of why it works.
