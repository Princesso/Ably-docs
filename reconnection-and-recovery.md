# Reconnection and recovery

AI Transport streams survive connection drops automatically. When a client disconnects, the server-side agent (the component that calls your LLM and publishes responses) keeps streaming tokens to the session channel. When the client reconnects, the Ably realtime client SDK resumes from where it left off. No tokens are lost, no responses break, and no manual retry logic is required.

This works because the session exists independently of any single connection. The agent publishes to a durable Ably channel, not to the client's connection. The client subscribes to that same channel. If the subscription drops, the channel keeps receiving tokens. When the client reattaches, it catches up on everything it missed.

<Aside data-type='note'>
With direct HTTP streaming, a connection drop kills the response stream. The LLM may still be generating, but the tokens have nowhere to go. Resuming requires building custom buffering, sequence numbering, and resume handlers from scratch. AI Transport eliminates this by decoupling the stream from the connection.
</Aside>

## How it works

The durable session channel persists independently of any single connection. When a client's connection drops, the following sequence occurs:

1. The agent continues streaming tokens to the channel. The stream is not tied to the client's connection.
2. The Ably realtime client SDK automatically reconnects.
3. On reconnection, the client transport uses the `untilAttach` history parameter to load any messages published during the gap. This parameter tells Ably to return all messages up to the exact point where the client reattached, closing the gap between historical and live messages.
4. The conversation state is restored. The client sees the complete response with no missing tokens.

No application code is needed. This is built into the transport layer.

## Understand the two recovery paths

There are two recovery paths depending on how long the client was disconnected.

### Brief disconnections (under two minutes)

Ably's connection protocol handles brief disconnections automatically. The Ably realtime client SDK reconnects and retrieves any messages published during the gap using `untilAttach`, as described above. There is no gap between the historical messages and the live stream, and the response resumes exactly where it left off. No application code is required.

### Extended disconnections (beyond channel persistence)

When a client has been offline beyond the channel's persistence window for live recovery (typically 24 to 72 hours, depending on your Ably plan), the connection protocol can no longer replay missed messages. Instead, the client loads the full conversation from channel history on reconnect.

Use `useView` to load history and paginate backward through older messages:

```javascript
const { nodes, hasOlder, loadOlder } = useView({ transport, limit: 30 })

// Load additional pages of older messages if available
if (hasOlder) {
  await loadOlder()
}
```

`useView` loads history on mount and uses the `untilAttach` parameter internally to ensure gapless continuity between historical and live messages. `nodes` contains the decoded conversation. `hasOlder` indicates whether more history is available. `loadOlder()` fetches the next page and expands the view window internally.

## Handle encoder recovery on the server

On the server side, the codec's encoder handles transient failures during streaming. If an append operation fails (for example, due to a network interruption between your server and Ably), the encoder falls back to a full message update rather than losing the token. The recovery follows this sequence:

1. The encoder appends the next token to the message (normal path).
2. If the append fails, the encoder sends a full update containing all accumulated content (recovery path).
3. The encoder continues appending from the recovered state.

This recovery happens automatically inside `turn.streamResponse()`. You do not need to write any retry logic. The accumulated response is never lost, even if individual append operations fail:

```javascript
const turn = transport.newTurn({ turnId, clientId })

const result = streamText({
  model: openai('gpt-4o'),
  messages,
  abortSignal: turn.abortSignal,
})

// Encoder recovery is handled internally - no retry logic needed
await turn.streamResponse(result.toUIMessageStream())
```

## Join a session mid-stream

When a client joins a session channel while a response is already streaming, the client transport's lifecycle tracker ensures it receives the correct sequence of events. Any lifecycle events that the client missed (such as the stream start marker) are synthesized so the client can process the in-progress stream correctly.

This means you can open a second tab or connect a second device while the agent is mid-response, and that client immediately sees the streaming content. This behavior also applies to the reconnection scenarios described above, because the session state does not depend on any single client being connected.

## Related features

- [Token streaming](/docs/ai-transport/features/token-streaming) - what gets recovered during reconnection.
- [Multi-device sessions](/docs/ai-transport/features/multi-device) - how sessions persist across devices and tabs.
- [History and replay](/docs/ai-transport/features/history) - loading and paginating conversation history.
- [Client transport API](/docs/ai-transport/api-reference/client-transport) - reference for `useView`, `loadOlder`, and other client methods.
- [Sessions](/docs/ai-transport/concepts/sessions) - how durable sessions enable recovery.
- [Get started](/docs/ai-transport/getting-started/vercel-ai-sdk) - build your first AI Transport application.
