# Personal Memory + Deep Analysis — Product Requirements

Status: Design requirements accepted; **not implemented or tested yet**.

## Nonnegotiable user goals

1. Ray-Ban Meta Wayfarer Gen 2 + iPhone 15 Pro Max provide a voice interface powered by OpenAI models.
2. User can explicitly tell the assistant to **remember** a fact/project decision and retrieve it in a future session.
3. User can find **old information by meaning or keywords**, with source conversation/date when known; use a retrieval step, not a short context-window summary.
4. Import selected, user-reviewed content from a ChatGPT data export, ideally `conversations.json` (and separate memory summary where available).
5. Memory can be accessed and managed from phone and computer; phone and computer use one source of truth after opt-in sync is configured.
6. Fast conversational mode and **deliberate deep-analysis mode**: complex questions use a reasoning-capable model with greater time/cost and deliver a spoken summary when ready.
7. No reliance on undocumented ChatGPT subscription endpoints. Current ChatGPT Pro memory/history is **not accessible through the official OpenAI API**; use a separate personal archive, and where appropriate a user-authorized connector for retrieval in ChatGPT.

## What already exists upstream

- OpenVision iOS app with glasses support, voice commands and OpenAI Realtime vision.
- Local conversation transcripts via `ConversationManager`, bounded by `maxConversations`.
- A `ContextualNoteTool` storing simple notes in UserDefaults and keyword-searching them.
- `MemoriesView` which manages Gemini-specific key/value settings.
- An unofficial ChatGPT subscription sign-in path; **do not depend on it** for persistent account-memory access.

## Proposed architecture

```
Ray-Bans <-> OpenVision on iPhone --TLS--> OpenAI Realtime (fast voice/vision)
                       |
                       +--> Memory API <--> private persistent store (retrieval)
                       |                     |
                       |                     +--> iPhone/Windows memory management UI
                       |                     +--> consent-based ChatGPT connector or files
                       |
                       +--> Reasoning API (deep-analysis route, when requested)
```

- All permanent memories stored privately; **never commit personal information or keys to this public repository**.
- Separate memory kinds: `profile`, `explicit_fact`, `project`, `conversation`, `document`, `decision`.
- Each record needs stable ID, source reference/title, created/updated time, confidence/provenance, tags and full text.
- Semantic retrieval plus full-text/date search; cite matched source titles, avoid inventing past events.
- Never assert that a record was saved until durable store confirms it.
- Explicit user actions: 'remember', 'find', 'correct memory', 'forget', 'show sources', 'export all'.
- Don't silently store camera frames or recordings. Opt-in memory capture only; allow deletion/export.
- Retrieval should be checked by the app before every answer depending on personal history, not only if the model chooses to call a tool.
- A background synchronization queue handles offline writes with conflict resolution and idempotency.

## Incremental milestones

1. **Compile current fork** with no secrets and establish a working iPhone installation flow. Current branch workflow is unsigned only.
2. Add a locally stored, user-editable Personal Archive with import/export, direct search and explicit save/forget.
3. Build a ChatGPT `conversations.json` importer: retain text, titles and timestamps; skip invalid/system messages and attachments unless the user separately opts in; show preview before importing. Memory summary is user-reviewed text, not an official memory API export.
4. Connect retrieval and write tools to OpenVision's OpenAI ordinary-chat and live-realtime backends. Instrument verified save responses.
5. Add private cloud sync and a phone/Windows browser manager. Select a backend only after reviewing authentication, privacy, cost and ChatGPT connector capabilities.
6. Add a separate reasoned-response route for long technical questions and a 'take more time' command.
7. Test on actual Gen 2 glasses: wake word, Bluetooth audio, live camera, locked phone, memory retrieval, save/forget, cross-device sync and billing.

## Acceptance tests

- User: 'Remember I changed the robot controller to an ESP32.' App confirms durable save.
- Next day: 'Which controller did I decide on?' Answer retrieves source with timestamp, even in a fresh conversation.
- User: 'What were we discussing around September about the fish-camera logging program?' Search returns imported transcript source or explicitly says no matching record.
- User: 'Think deeply about this engineering problem' routes to reasoning model, can take longer, then gives a spoken result and a persistent written solution.
- Deleted memories do not reappear in app retrieval without reimport; sync handles tombstones.

## Honest limitations

- Same GPT models do **not** mean the same ChatGPT Pro subscription, private memories or chat sidebar.
- ChatGPT itself can access a shared memory store only through an authorized connector/upload or another explicitly configured interface, not by automatic native sync.
- Cloud AI and deep reasoning may incur usage charges separate from ChatGPT's consumer subscription.
- First successful unsigned build is **not** a ready-to-use glasses application; Apple signing and Meta app registration remain.
