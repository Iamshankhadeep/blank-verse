# Open Source Claude Co-Work Architecture

## Goal

Build an open source **desktop-first co-work product** inspired by Claude Co-Work.

The product should have two operating modes:

1. **Desktop-local co-work**
   - The primary surface is a desktop app
   - The user works with one or more agents inside a local workspace
   - Local LLMs run on the user machine through **LM Studio** or compatible local servers
   - Hosted models are available through provider APIs
2. **Cloud-backed co-work**
   - Shared sessions, presence, run history, logs, and artifacts sync through a backend
   - The same agent/runtime model works whether a run uses a local model or a hosted API
   - Remote execution can be added later, but it is not the defining architecture boundary

The key design rules are:

- **The desktop app is the primary runtime host.**
- **Every model provider, including local models, must fit behind one internal model interface.**

That is what keeps the product from collapsing into a server-first app with local support bolted on later.

---

## Product Shape

### User-facing capabilities

- Work inside a desktop app with one or more agent threads tied to a workspace
- Run coding tasks with tools against local files, terminal, and git
- Stream thoughts, tool calls, logs, patches, and approvals in real time
- Switch models per agent, per run, or per workspace
- Use local models on-device when privacy, latency, or cost matters
- Use hosted provider APIs when model quality or scale matters
- Sync session state, artifacts, and history to the cloud for co-work continuity

### Collaboration model

- Primary mode: one human user co-working with multiple agents in a desktop workspace
- Shared-cloud mode: session state, artifacts, and event history are synchronized through a backend
- Multi-human shared sessions should be possible later, but the first architecture target is not “Google Docs for prompts”; it is a strong single-user desktop product with cloud-backed continuity

### Supported provider classes

- **Local**
  - LM Studio
  - Ollama later if needed
- **Hosted direct APIs**
  - OpenAI
  - Anthropic
  - Google Gemini
  - xAI
  - Mistral
- **Hosted aggregator or enterprise APIs**
  - OpenRouter
  - Azure OpenAI
  - AWS Bedrock
  - Together

---

## Core Architecture

### 1. Desktop app

This is the main product surface, not a thin wrapper around a web app.

Responsibilities:

- Host the workspace UI, agent threads, approvals, and settings
- Keep a local cache of sessions, artifacts, and run history
- Connect to local model endpoints on `localhost`
- Call hosted provider APIs directly when the user chooses API mode
- Run or delegate local tool execution
- Sync normalized events and artifacts to the cloud backend

### 2. Local runtime host

This is the desktop-side execution engine.

Responsibilities:

- Maintain conversation state
- Build prompts from repo context and instructions
- Decide when to call tools
- Route model calls through the provider adapter interface
- Execute local tools with user approval boundaries
- Emit normalized run events to both the UI and the sync layer

The runtime should not care whether the selected model is Anthropic, OpenAI, or LM Studio. It should only care about the internal model contract.

### 3. Cloud sync backend

The backend is mainly a collaboration and persistence layer for the desktop product.

Responsibilities:

- auth
- sessions
- presence
- run event ingestion
- artifact download
- artifact storage
- model registry lookup
- optional remote execution later

This should expose:

- REST endpoints for stateful operations and sync
- WebSocket for live session presence and event fan-out

### 4. Execution modes

The system should support the same co-work UX across three execution paths:

- **Local model mode**
  - Desktop app calls a local model server such as LM Studio
  - Model inference stays on the user machine
  - Tool execution stays on the user machine
- **Direct API mode**
  - Desktop app calls a hosted provider API using the user’s API key
  - Tool execution still stays local unless explicitly offloaded later
- **Remote run mode**
  - A future extension where the cloud backend runs the agent in isolated workers against a synced workspace snapshot
  - This should reuse the same event protocol and model adapter contracts

### 5. Tool execution layer

Responsibilities:

- File operations
- Search
- Terminal commands
- Git actions
- Web fetches if enabled
- Structured app integrations later

Run tools in isolated workers:

- local subprocess sandbox in local mode
- containerized workers only if remote execution is enabled later

### 6. Provider adapter layer

Each provider gets its own adapter, but all adapters implement the same interface.

Responsibilities:

- Convert internal messages to provider-specific format
- Stream deltas back into the internal event format
- Handle tool-call schema differences
- Map usage, errors, timeouts, and rate limits

### 7. Storage layer

Persist:

- users
- workspaces
- devices
- provider configs
- sessions
- runs
- events
- tool logs
- artifacts
- prompt snapshots
- local sync checkpoints
- billing metadata if needed later

---

## Internal Model Contract

This is the most important decision in the system.

Define one internal request/response model such as:

```ts
type ModelRequest = {
  model: string;
  provider: string;
  messages: InternalMessage[];
  system?: string;
  tools?: ToolDefinition[];
  toolChoice?: "auto" | "none" | { name: string };
  maxTokens?: number;
  temperature?: number;
  metadata?: Record<string, string>;
  stream?: boolean;
};

type ModelEvent =
  | { type: "message_start"; id: string }
  | { type: "text_delta"; text: string }
  | { type: "tool_call_start"; callId: string; name: string; inputJson: string }
  | { type: "tool_call_delta"; callId: string; inputJsonDelta: string }
  | { type: "tool_call_end"; callId: string }
  | { type: "message_end"; stopReason: "end_turn" | "tool_use" | "max_tokens" }
  | { type: "usage"; inputTokens: number; outputTokens: number }
  | { type: "error"; code: string; message: string };
```

If this contract is stable, the rest of the platform stays clean.

### Why this matters

Different providers disagree on:

- message schema
- system prompt handling
- tool call encoding
- streaming chunk shape
- JSON mode
- token accounting
- error payloads

Your backend should absorb those differences once in adapters, not leak them into the agent runtime.

---

## Local LLM And Hosted API Connectivity

### Recommended approach

Treat **LM Studio** as an **OpenAI-compatible endpoint**, not as a runtime special case.

That means a provider config like:

```json
{
  "provider": "openai-compatible",
  "label": "lm-studio-local",
  "baseUrl": "http://127.0.0.1:1234/v1",
  "apiKey": "lm-studio",
  "model": "local-model-id"
}
```

Hosted APIs should use the same provider model:

```json
{
  "provider": "anthropic",
  "label": "claude-api",
  "apiKeyRef": "keychain://anthropic/default",
  "model": "claude-sonnet"
}
```

### Why this is the right default

- One adapter can cover LM Studio and many other OpenAI-compatible servers
- Local models become a config problem instead of a code fork
- Hosted APIs become a config problem instead of a separate execution stack
- You can swap between local and hosted endpoints without changing runtime logic

### Practical constraints

Local models vary a lot in:

- tool calling quality
- context length
- structured JSON reliability
- latency
- multi-step agent performance

So the system should track per-model capabilities:

```ts
type ModelCapabilities = {
  supportsTools: boolean;
  supportsVision: boolean;
  supportsJsonMode: boolean;
  supportsReasoning: boolean;
  maxContextTokens?: number;
};
```

Do not assume every LM Studio model can reliably run the full coding-agent loop. The backend should degrade gracefully:

- disable tool calling if unsupported
- require explicit confirmation for risky actions
- fall back to smaller task scopes

### Privacy boundary

For local model mode:

- prompts, context, and inference stay on the user machine
- the cloud backend should receive only normalized run events and artifacts needed for sync unless the user opts into richer telemetry

For API mode:

- the desktop app may call provider APIs directly with user-managed credentials
- an enterprise relay can exist later, but it should be optional, not the default architecture

---

## API Support Strategy

“Support all APIs” only works if you define tiers.

### Tier 1: First-class

Ship first-party adapters for:

- Anthropic
- OpenAI-compatible
- Gemini

This already covers:

- Claude
- OpenAI
- LM Studio
- OpenRouter-compatible OpenAI mode in many cases
- Azure OpenAI with a small compatibility layer

### Tier 2: Compatibility adapters

Support providers through:

- OpenAI-compatible protocol
- LiteLLM-style gateway if users already run one

This reduces the number of direct integrations you must maintain.

### Tier 3: Custom providers

Allow user-defined providers with:

- base URL
- auth header template
- model list
- capability flags

This matters for self-hosted and enterprise environments.

---

## Cloud Backend Responsibilities For Co-Work

### Session service

Tracks:

- workspace
- participants
- active branch or synced snapshot
- current run
- permissions

### Presence service

Tracks:

- connected devices
- active viewers
- typing or active-run state
- last sync position

### Run event service

Ingests and fans out:

- local run events from desktop clients
- remote run events later if cloud execution is added
- approval requests
- run status changes

### Artifact service

Stores:

- diffs
- patches
- logs
- generated files
- session exports

### Sync service

Manages:

- desktop checkpoints
- conflict resolution
- offline replay
- per-device cursors

### Secrets policy service

Defines how credentials are stored and synced.

Important distinction:

- local model configs and API keys should live in OS keychain or local secure storage by default
- cloud sync should not require uploading provider secrets unless the user explicitly opts into encrypted secret sync

### Remote execution service

Optional later:

- isolated workers for long-running or shared runs
- synced workspace snapshots
- policy-controlled tool execution
- audit trails

---

## Recommended Data Model

Core entities:

- `User`
- `Workspace`
- `Device`
- `SessionParticipant`
- `PresenceState`
- `ProviderConfig`
- `ModelProfile`
- `Session`
- `Run`
- `RunEvent`
- `ToolInvocation`
- `Artifact`
- `ApprovalRequest`

This is enough for an MVP without over-designing.

---

## Suggested Implementation Shape

### Backend

- **TypeScript**
- **Node.js**
- **Fastify** or **NestJS**
- **Postgres**
- **Redis** for realtime fan-out and ephemeral run coordination
- **S3-compatible blob storage** for artifacts
- WebSocket-based sync layer

### Desktop app

- **Electron** or **Tauri**
- local SQLite store for cache and offline state
- OS keychain integration for credentials
- local workspace bridge for files, terminal, and git

### Workers

- Node workers only for cloud-side sync jobs and optional remote execution

### Agent SDK layout

Suggested packages:

- `packages/core-agent`
- `packages/model-adapters`
- `packages/tool-runtime`
- `packages/sync-protocol`
- `packages/shared-types`
- `apps/api`
- `apps/desktop`
- `apps/worker`

---

## Provider Adapter Layout

Keep adapters small and boring:

- `anthropic.adapter.ts`
- `openai.adapter.ts`
- `openaiCompatible.adapter.ts`
- `gemini.adapter.ts`

The `openaiCompatible` adapter should handle:

- LM Studio
- local gateways
- some enterprise proxies
- many third-party compatible endpoints

That adapter is the fastest path to broad provider support.

---

## Security Model

Minimum baseline:

- workspace-scoped permissions
- tool allowlists
- command approval flow
- redaction of secrets from logs
- provider key storage in OS keychain by default
- isolated execution for cloud workers if remote runs are enabled

For local mode:

- make destructive tools opt-in
- surface exact commands before execution
- never proxy local-model prompts through the cloud unless the user explicitly enables it

For cloud mode:

- do not run arbitrary shell commands in the API process
- keep sync and collaboration services separate from any remote execution workers

---

## MVP Scope

If you want this to ship, keep v0 narrow.

### MVP

- desktop app
- local runtime host
- single agent runtime
- OpenAI-compatible adapter
- Anthropic adapter
- LM Studio via OpenAI-compatible config
- direct hosted API mode from the desktop app
- file/search/shell/git tools
- streaming events
- run history
- cloud sync for sessions and artifacts

### v1

- cloud co-work sessions
- presence
- shared artifacts
- approvals
- role-based workspace access
- Gemini adapter
- model capability registry

### v2

- multi-human shared sessions
- hosted execution fleets
- billing
- org admin controls
- plugin/app ecosystem
- fine-grained audit trails

---

## Recommended Build Order

1. Build the internal model contract.
2. Build the desktop shell and local runtime host.
3. Implement the `openaiCompatible` adapter first.
4. Prove local LM Studio runs end-to-end through that adapter.
5. Add Anthropic as the first non-compatible direct adapter.
6. Build the sync/event protocol used by desktop and cloud.
7. Add cloud session storage, presence, and shared artifacts.
8. Add optional remote execution only after the local and direct-API paths are stable.

If you invert this order, the backend will become too central and the desktop product will turn into a thin client.

---

## What “Done Right” Looks Like

You know the architecture is correct if:

- switching from Claude to LM Studio is a config change
- tool calling uses the same runtime path across providers
- streaming events look identical regardless of whether the run used a local model or a hosted API
- cloud sync reuses the same agent core as local mode
- adding a new provider mostly means writing one adapter
- losing network connectivity does not break the core desktop workflow

---

## Immediate Next Step

Create an implementation repo with:

- monorepo package structure
- desktop app shell
- local SQLite and keychain wiring
- internal model types
- `openaiCompatible` adapter
- basic local agent loop
- LM Studio configuration example
- direct Anthropic API configuration example
- sync event protocol and session ingest endpoint

That is the smallest real slice that proves the concept.
