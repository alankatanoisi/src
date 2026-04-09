#   
  
Based on the documentation, here's a breakdown of Claude Code's architectural strengths and what to **keep vs. dispose** when building a new AI coding harness:  
  
## Architectural Strengths (Keep These)  
  
1. **Agentic Loop (`query.ts`)** — The `while(true)` think→act→observe cycle with 7 termination conditions and 4 recovery paths (max-output-tokens, prompt-too-long, model fallback, stop-hook retry). This is battle-tested and essential. ([/docs/conversation/the-loop](/docs/conversation/the-loop))  
  
2. **Unified Tool Abstraction** — 35-field `Tool` interface with `buildTool()` factory, covering input validation, permissions, execution, and rendering in one type. Highly composable. ([/docs/tools/what-are-tools](/docs/tools/what-are-tools))  
  
3. **Three-tier Permission Model** — Allow/Ask/Deny with 5-layer rule sources, 3-dimension matching (tool name, command pattern via AST, file path glob), and denial tracking to prevent infinite loops. ([/docs/safety/permission-model](/docs/safety/permission-model))  
  
4. **Context Engineering Pipeline** — The 5-stage pre-processing (`applyToolResultBudget → snipCompact → microcompact → contextCollapse → autocompact`) with prompt-too-long reactive recovery. ([/docs/conversation/the-loop](/docs/conversation/the-loop))  
  
5. **System Prompt Caching Strategy** — Static/dynamic boundary split (`SYSTEM_PROMPT_DYNAMIC_BOUNDARY`) enabling global cache for static content while keeping dynamic sections fresh. ([/docs/context/system-prompt](/docs/context/system-prompt))  
  
6. **Hooks System** — 22 lifecycle events with `if` condition matching, JSON output schema, and async hook protocol. Enables extensibility without modifying core. ([/docs/extensibility/hooks](/docs/extensibility/hooks))  
  
7. **MCP Protocol Integration** — Unified tool interface for external tools with 7 transport types, LRU caching, and graceful reconnection. ([/docs/extensibility/mcp-protocol](/docs/extensibility/mcp-protocol))  
  
## What to Dispose / Rethink  
  
1. **Terminal-only UI (React/Ink TUI)** — Unless you're building a CLI tool, the Ink rendering layer is tightly coupled and unnecessary for web/IDE harnesses.  
  
2. **Feature flag complexity** — The `feature() = false` pattern and extensive `BUILD_TARGET`/`BUILD_ENV` macros are Anthropic-internal concerns. Simplify for your use case.  
  
3. **Ant-only tools and 1P-specific paths** — Internal tools (`REPLTool`, `TungstenTool`, `ConfigTool`) and 1P-specific cache modes add complexity with no external value.  
  
4. **`CLAUDE_CODE_SIMPLE` escape hatch** — Indicates the full system prompt assembly may be over-engineered for simpler use cases. Consider starting minimal.  
  
5. **Streaming tool executor dual-path** — The code maintains both streaming and non-streaming tool execution paths (`StreamingToolExecutor` vs `runTools`). Pick one for your harness.  
  
```suggestions  
(Agentic Loop deep dive)[/docs/conversation/the-loop]  
(Tool system design)[/docs/tools/what-are-tools]  
(Permission model)[/docs/safety/permission-model]  
```  
  
  
  
Based on the documentation, here's a breakdown of Claude Code's architectural strengths and what to **keep vs. dispose** when building a new AI coding harness:  
  
## Architectural Strengths (Keep These)  
  
1. **Agentic Loop (`query.ts`)** — The `while(true)` think→act→observe cycle with 7 termination conditions and 4 recovery paths (max-output-tokens, prompt-too-long, model fallback, stop-hook retry). This is battle-tested and essential. ([/docs/conversation/the-loop](/docs/conversation/the-loop))  
  
2. **Unified Tool Abstraction** — 35-field `Tool` interface with `buildTool()` factory, covering input validation, permissions, execution, and rendering in one type. Highly composable. ([/docs/tools/what-are-tools](/docs/tools/what-are-tools))  
  
3. **Three-tier Permission Model** — Allow/Ask/Deny with 5-layer rule sources, 3-dimension matching (tool name, command pattern via AST, file path glob), and denial tracking to prevent infinite loops. ([/docs/safety/permission-model](/docs/safety/permission-model))  
  
4. **Context Engineering Pipeline** — The 5-stage pre-processing (`applyToolResultBudget → snipCompact → microcompact → contextCollapse → autocompact`) with prompt-too-long reactive recovery. ([/docs/conversation/the-loop](/docs/conversation/the-loop))  
  
5. **System Prompt Caching Strategy** — Static/dynamic boundary split (`SYSTEM_PROMPT_DYNAMIC_BOUNDARY`) enabling global cache for static content while keeping dynamic sections fresh. ([/docs/context/system-prompt](/docs/context/system-prompt))  
  
6. **Hooks System** — 22 lifecycle events with `if` condition matching, JSON output schema, and async hook protocol. Enables extensibility without modifying core. ([/docs/extensibility/hooks](/docs/extensibility/hooks))  
  
7. **MCP Protocol Integration** — Unified tool interface for external tools with 7 transport types, LRU caching, and graceful reconnection. ([/docs/extensibility/mcp-protocol](/docs/extensibility/mcp-protocol))  
  
## What to Dispose / Rethink  
  
1. **Terminal-only UI (React/Ink TUI)** — Unless you're building a CLI tool, the Ink rendering layer is tightly coupled and unnecessary for web/IDE harnesses.  
  
2. **Feature flag complexity** — The `feature() = false` pattern and extensive `BUILD_TARGET`/`BUILD_ENV` macros are Anthropic-internal concerns. Simplify for your use case.  
  
3. **Ant-only tools and 1P-specific paths** — Internal tools (`REPLTool`, `TungstenTool`, `ConfigTool`) and 1P-specific cache modes add complexity with no external value.  
  
4. **`CLAUDE_CODE_SIMPLE` escape hatch** — Indicates the full system prompt assembly may be over-engineered for simpler use cases. Consider starting minimal.  
  
5. **Streaming tool executor dual-path** — The code maintains both streaming and non-streaming tool execution paths (`StreamingToolExecutor` vs `runTools`). Pick one for your harness.  
  
```suggestions  
(Agentic Loop deep dive)[/docs/conversation/the-loop]  
(Tool system design)[/docs/tools/what-are-tools]  
(Permission model)[/docs/safety/permission-model]  
```  
  
  
  
  
  
Let me search the docs for UI-related content and the memory system.Based on the documentation, here's a comprehensive answer:  
  
## UI Alternatives to TUI  
  
The docs reveal Claude Code already has a **Bridge Mode** ([/docs/features/bridge-mode](/docs/features/bridge-mode)) — the CLI registers as a remote execution environment, and **claude.ai acts as the web UI**. This is the architectural pattern to follow:  
  
- **Decouple UI from the agentic core.** The 5-layer architecture ([/docs/introduction/architecture-overview](/docs/introduction/architecture-overview)) shows the interaction layer (React/Ink) is cleanly separated from the orchestration layer (QueryEngine). Replace Layer 1 with a **web frontend or IDE extension** that communicates with the QueryEngine via a transport layer (like Bridge Mode's HTTP long-polling or WebSocket).  
- **Bridge Mode as a blueprint** — It already proves the core loop can be driven remotely, with permission requests flowing to a web UI for approval.  
  
## Memory System Improvements  
  
The current memory system ([/docs/context/project-memory](/docs/context/project-memory)) has clear improvement opportunities:  
  
**Current limitations:**  
- **Pure file-based, no vector store** — Recall relies on a Sonnet side-query scanning frontmatter, limited to ≤5 memories per turn  
- **MEMORY.md index caps** — 200 lines / 25KB hard limits; large projects will hit this  
- **No semantic search** — The `selectRelevantMemories()` call uses an LLM to pick from a manifest, not embeddings  
  
**Improvements to consider:**  
1. **Add vector embeddings** — Replace the Sonnet side-query with a local embedding index (e.g., HNSW) for faster, cheaper, and more scalable recall  
2. **Hierarchical memory tiers** — The 4-type system (user/feedback/project/reference) is good, but add **episodic memory** (what happened in past sessions) beyond just KAIROS daily logs  
3. **Proactive memory decay** — Currently relies on "memory drift defense" (grep to verify claims). Add TTL/staleness scoring so old memories auto-deprecate  
4. **Raise the ≤5 recall limit** — With embeddings, you can retrieve more candidates cheaply and let the model filter  
5. **Session Memory as first-class** — The SM-compact integration already skips the compression API by using Session Memory as summaries. Formalize this as a persistent mid-term memory layer between ephemeral context and long-term MEMORY.md  
  
```suggestions  
(Project Memory system)[/docs/context/project-memory]  
(Architecture overview)[/docs/introduction/architecture-overview]  
(Bridge Mode)[/docs/features/bridge-mode]  
```  
  
  
  
  
  
Let me check what the documentation says about the transport layer, Bridge Mode implementation, and any architectural patterns that could inform this.Now I have a thorough understanding of the architecture. The documentation explicitly notes that Bridge Mode chose **long-polling over WebSocket** as a design decision. Let me provide a well-grounded answer based on what the docs reveal about the architecture.  
  
Great question. The documentation provides strong architectural guidance here. Let me ground this in what Claude Code's architecture reveals:  
  
## Why WebSocket Over Long-Polling  
  
The Bridge Mode docs ([/docs/features/bridge-mode](/docs/features/bridge-mode)) explicitly note a key design decision: **"长轮询而非 WebSocket — pollForWork uses HTTP GET + 10s timeout. Simple and reliable, no need to maintain WebSocket connections."** This was pragmatic for their use case (remote CLI execution), but for a **real-time web frontend**, it's the wrong tradeoff. Here's why:  
  
- The agentic loop is **streaming-first** — `deps.callModel()` returns an `AsyncGenerator<StreamEvent>`, and `StreamingToolExecutor` begins tool execution *during* the stream ([/docs/introduction/architecture-overview](/docs/introduction/architecture-overview)). Long-polling with 10s timeouts adds unacceptable latency to this real-time token stream.  
- Permission requests need **instant bidirectional flow** — the bridge already implements permission round-trips to claude.ai. WebSocket makes this sub-100ms instead of up to 10s.  
  
## Recommended Architecture  
  
Based on the 5-layer architecture, you'd **replace Layer 1 (Interaction)** while keeping Layers 2-5 intact:  
  
```  
┌─────────────────────────────────────┐  
│  Web Frontend (any framework)       │  ← New Layer 1  
│  - Token stream rendering           │  
│  - Permission approval UI           │  
│  - Tool progress visualization      │  
├─────────────────────────────────────┤  
│  WebSocket Transport Layer          │  ← Replaces replBridgeTransport.ts  
│  - Bidirectional event stream       │  
│  - Heartbeat/reconnection           │  
│  - Session multiplexing             │  
├─────────────────────────────────────┤  
│  QueryEngine (unchanged)            │  ← Layer 2: Orchestration  
│  Agentic Loop (unchanged)           │  ← Layer 3: Core Loop  
│  Tool Layer (unchanged)             │  ← Layer 4: Tools  
│  API Communication (unchanged)      │  ← Layer 5: Streaming Provider  
└─────────────────────────────────────┘  
```  
  
## F	  
  
```suggestions  
(Bridge Mode architecture)[/docs/features/bridge-mode]  
(5-layer architecture overview)[/docs/introduction/architecture-overview]  
(Agentic Loop details)[/docs/conversation/the-loop]  
```  
  
  
  
  
Great question. Let me search the docs for multi-provider, model fallback, and provider abstraction patterns.Now I have a comprehensive picture. Let me provide a well-grounded answer.  
  
Excellent question. The documentation reveals that Claude Code **already has multi-provider infrastructure** — and its patterns show both the viability and the architectural seams you'd need to exploit.  
  
## What Already Exists: Multi-Provider Abstraction  
  
The streaming docs ([/docs/conversation/streaming](/docs/conversation/streaming)) show Claude Code already supports 4 providers behind a **unified `Stream<BetaRawMessageStreamEvent>` abstraction**:  
  
| Provider | Protocol | Notes |  
|---|---|---|  
| Anthropic Direct | Native SSE | Lowest latency |  
| AWS Bedrock | AWS SDK streaming | Extra beta headers |  
| Google Vertex | gRPC → event stream | `getMergedBetas()` adapter |  
| Azure | Anthropic-compatible API | Custom base URL |  
  
`getAPIProvider()` in `providers.ts` selects the provider at runtime. **Upper layers (QueryEngine, REPL) don't know or care which provider is active.** This is the key architectural proof: the agentic loop is already provider-agnostic.  
  
## Model Fallback: The Existing Multi-Model Pattern  
  
The agentic loop ([/docs/conversation/the-loop](/docs/conversation/the-loop)) already implements **cross-model switching mid-conversation**:  
  
- When `FallbackTriggeredError` fires, it clears assistant messages, synthesizes `tool_result: "Model fallback triggered"`, **strips thinking signature blocks** (because signatures are model-bound), switches to `fallbackModel`, and re-issues the stream.  
- This proves the loop can survive model switches, but reveals a constraint: **thinking/signature blocks are model-specific** and must be stripped on transition.  
  
## Viability Assessment for Your Proposed Architecture  
  
**Codex for engineering + Claude for agentic loop + Gemini for embeddings/search:**  
  
### What Works (High Viability)  
  
1. **Gemini for embeddings/retrieval** — The memory system currently uses a Sonnet side-query for `selectRelevantMemories()` ([/docs/context/project-memory](/docs/context/project-memory)). Replacing this with Gemini embeddings is a clean swap — it's already a separate call path, not entangled with the main loop. The MCP protocol ([/docs/extensibility/mcp-protocol](/docs/extensibility/mcp-protocol)) could even expose an embedding service as an MCP tool with its own transport.  
  
2. **Claude for the agentic loop** — This is what it's built for. The 7 termination conditions, 4 recovery paths, streaming tool executor, and permission model are all battle-tested. Keep this as-is.  
  
3. **Specialized models for specialized tasks** — The `Tool` interface's `buildTool()` pattern means each tool could internally route to a different model. A code-generation tool could call Codex while the orchestration layer stays on Claude.  
  
### Architectural Challenges  
  
1. **Unified message format** — The loop expects `BetaRawMessageStreamEvent` everywhere. Non-Anthropic models would need **adapter layers** that translate their responses into this event schema. The existing provider abstraction (`Stream<BetaRawMessageStreamEvent>`) is the pattern to follow.  
  
2. **Thinking/signature incompatibility** — The fallback code explicitly strips `signatureBlocks` when switching models. A multi-model harness must handle this at every model boundary, not just on fallback.  
  
3. **System prompt coupling** — The 5-stage system prompt assembly ([/docs/context/system-prompt](/docs/context/system-prompt)) is Claude-optimized (tool definitions, identity blocks, cache boundaries). Other models may need entirely different prompt engineering.  
  
4. **Permission model assumptions** — The 3-tier permission system and auto-mode's `yoloClassifier` assume Claude's safety classification capabilities. Other models may not support equivalent safety classification ([/docs/safety/auto-mode](/docs/safety/auto-mode)).  
  
### Recommended Architecture Pattern  
  
```  
┌─────────────────────────────────────────────┐  
│           Router / Orchestrator              │  
│  (Decides which model handles what)          │  
├──────────┬──────────────┬───────────────────┤  
│ Claude   │ Codex/Other  │ Gemini            │  
│ Agentic  │ Code Gen     │ Embeddings +      │  
│ Loop +   │ (via Tool    │ Retrieval         │  
│ Planning │  interface)  │ (via MCP or       │  
│          │              │  direct API)      │  
├──────────┴──────────────┴───────────────────┤  
│     Unified Event Adapter Layer             │  
│  (All responses → BetaRawMessageStreamEvent │  
│   or your custom unified event schema)      │  
└─────────────────────────────────────────────┘  
```  
  
The key insight from the docs: **the tool layer is your integration point**. Each tool can internally call any model/service, while the agentic loop only sees tool inputs and outputs. This is already how MCP tools work — external services behind a unified interface.  
  
```suggestions  
(Streaming & Provider abstraction)[/docs/conversation/streaming]  
(Agentic Loop & model fallback)[/docs/conversation/the-loop]  
(MCP Protocol for external services)[/docs/extensibility/mcp-protocol]  
```  
  
  
This is a great architectural question. Let me ground this carefully in what the documentation reveals about Claude Code's design boundaries.  
  
## High-Level Viability: Very High  
  
The 5-layer architecture ([/docs/introduction/architecture-overview](/docs/introduction/architecture-overview)) proves this is viable because **Layers 2-4 are already model-agnostic in design**. The model coupling only exists at two specific points:  
  
### Where Claude Models Are Actually Coupled  
  
1. **Layer 5 (Communication)** — `deps.callModel()` returns `AsyncGenerator<BetaRawMessageStreamEvent>`. This is the **only hard coupling** to Anthropic's API format. OpenAI's Responses API would need an adapter here.  
  
2. **System Prompt Assembly** — `fetchSystemPromptParts()` is Claude-optimized (tool definitions use Anthropic's `tool_use`/`tool_result` block format, thinking blocks, `cache_control` markers). OpenAI uses a different function-calling schema.  
  
3. **Thinking/Signature Blocks** — The fallback code strips `signatureBlocks` because they're model-bound. OpenAI doesn't have this concept.  
  
4. **Safety Classification** — `yoloClassifier` in auto-mode assumes Claude's safety behavior. OpenAI models would need a different classifier.  
  
### What You'd Need to Build  
  
```  
┌──────────────────────────────────────────────────┐  
│  OpenAI Responses API Adapter                     │  
│  ─────────────────────────────────────────────    │  
│  Translates:                                      │  
│  • function_call ↔ tool_use blocks                │  
│  • function response ↔ tool_result blocks         │  
│  • OpenAI streaming chunks → BetaRawMessageStream │  
│    EventStream (or your unified event schema)     │  
│  • tool_choice → Anthropic tool routing            │  
└──────────────────────────────────────────────────┘  
```  
  
### What Transfers Cleanly (No Changes Needed)  
  
- **Agentic Loop (`query.ts`)** — The `while(true)` think→act→observe cycle, 7 termination conditions, 4 recovery paths. These are **model-independent state machine logic**. As long as your adapter translates OpenAI responses into the expected `tool_use`/`tool_result` message format, the loop works unchanged.  
  
- **Tool Layer** — The 35-field `Tool` interface, `buildTool()`, `validateInput()` → `checkPermissions()` → `call()` chain. Tools don't know what model invoked them.  
  
- **Permission Model** — The 3-tier Allow/Ask/Deny system with 5-layer rule sources. Entirely model-independent.  
  
- **Context Pre-processing Pipeline** — `applyToolResultBudget → snipCompact → microcompact → contextCollapse → autocompact`. These operate on message arrays, not model-specific formats.  
  
- **Hooks System** — 22 lifecycle events. Completely model-agnostic.  
  
- **QueryEngine orchestration** — Session management, transcript persistence, cost tracking, file history snapshots. All model-independent.  
  
### What Needs Adaptation (Medium Effort)  
  
| Component | Issue | Solution |  
|---|---|---|  
| `BetaRawMessageStreamEvent` | Anthropic-specific event schema | Build a **unified event type** that both OpenAI chunks and Anthropic events map to |  
| `StreamingToolExecutor` | Expects Anthropic `tool_use` blocks mid-stream | Adapter must emit synthetic `tool_use` events from OpenAI's `function_call` chunks |  
| System Prompt | Claude-specific tool formatting, `cache_control` | Rebuild `fetchSystemPromptParts()` with OpenAI function definitions |  
| Prompt caching | Anthropic's `cache_control` breakpoints | OpenAI has no equivalent; remove or replace with OpenAI's prompt caching strategy |  
| Token counting | Uses Anthropic token counts for autocompact thresholds | Swap to `tiktoken` or OpenAI's token counting |  
  
### The Critical Design Decision  
  
The docs reveal that the **provider abstraction already exists** at Layer 5 — `getAPIProvider()` selects between Anthropic/Bedrock/Vertex/Azure and all return `Stream<BetaRawMessageStreamEvent>`. You'd add OpenAI as a 5th provider behind this same abstraction. The key question is whether you:  
  
1. **Adapt OpenAI → Anthropic format** (minimal changes to Layers 1-4, heavier adapter) — recommended if you want to preserve the battle-tested loop logic exactly  
2. **Build a new unified format** (more changes to Layers 2-3, cleaner long-term) — recommended if you're building a truly provider-agnostic harness  
  
**Bottom line:** The architecture is ~70% transferable without Claude models. The agentic loop, tool system, permission model, hooks, and context engineering are all model-independent. The remaining 30% is adapter work at the streaming/event boundary and system prompt assembly.  
  
```suggestions  
(Architecture overview - 5 layers)[/docs/introduction/architecture-overview]  
(Streaming & Provider abstraction)[/docs/conversation/streaming]  
(Agentic Loop state machine)[/docs/conversation/the-loop]  
```  
  
  
  
  
Let me get a deep understanding of the tool system from the docs.Now I have a comprehensive understanding. Let me compose the response.  
  
## The Tool System In Depth  
  
Based on the docs ([/docs/tools/what-are-tools](/docs/tools/what-are-tools), [/docs/tools/shell-execution](/docs/tools/shell-execution), [/docs/tools/file-operations](/docs/tools/file-operations)), the tool system is a **35-field unified `Tool<Input, Output, Progress>` interface** with four core pillars:  
  
### Current Architecture  
  
**1. The 10-Step Execution Chain:**  
```  
AI emits tool_use → StreamingToolExecutor.addTool()  
  → findToolByName() → validateInput() → canUseTool() (permission UI)  
  → checkPermissions() (rule matching) → call() (execute)  
  → onProgress() (stream updates) → mapToolResultToToolResultBlockParam()  
  → append to conversation → next loop iteration  
```  
  
**2. Dual Execution Paths:** `StreamingToolExecutor` begins tool execution *during* the model stream (parallel), while `runTools()` is the sequential fallback. Tools declare `isConcurrencySafe()` to opt into parallel execution.  
  
**3. Registration is Dynamic:** `getTools()` reassembles the tool list on *every API call* — `isEnabled()` checks runtime state, feature flags, platform, and deny rules. Not cached globally.  
  
**4. Budget Control:** Each tool declares `maxResultSizeChars` (Bash: 30K, Skill: 100K, Read: Infinity). Oversized results are persisted to disk via `applyToolResultBudget()`, and the AI receives only a preview + file path.  
  
---  
  
## Innovative Overhauls for a Provider-Agnostic Harness  
  
### 1. Replace `tool_use`/`tool_result` with a Universal Tool Protocol  
  
**Problem:** The entire chain is hardcoded to Anthropic's `tool_use` block format. OpenAI uses `function_call`/`function` messages. Google uses `functionCall`/`functionResponse`.  
  
**Overhaul:** Create a **Canonical Tool Event (CTE)** layer:  
  
```  
┌─────────────────────────────────────────────────┐  
│  Canonical Tool Event Schema                     │  
│  ─────────────────────────────────────────────   │  
│  ToolInvocation { name, id, input, metadata }    │  
│  ToolResult { id, output, isError, isIncomplete }│  
│  ToolProgress { id, delta, elapsed }             │  
├──────────┬──────────────┬───────────────────────┤  
│ Anthropic│   OpenAI     │   Google              │  
│ Adapter  │   Adapter    │   Adapter             │  
│ tool_use │ function_call│ functionCall          │  
│ ↕ CTE    │ ↕ CTE       │ ↕ CTE                │  
└──────────┴──────────────┴───────────────────────┘  
```  
  
The `StreamingToolExecutor` and `runTools()` would operate entirely on CTEs. Each provider adapter translates bidirectionally. This is the **single most impactful change** — it decouples the entire tool chain from any specific model API.  
  
### 2. Replace `inputSchema` (Zod) with a Dual-Schema System  
  
**Problem:** Built-in tools use Zod schemas, but MCP tools already use raw JSON Schema (`inputJSONSchema`). OpenAI and Google also expect JSON Schema for function definitions. The current system has two parallel paths.  
  
**Overhaul:** Make **JSON Schema the canonical format**, with Zod as an optional compile-time convenience:  
- `buildTool()` accepts Zod but compiles to JSON Schema at registration time  
- All runtime validation uses JSON Schema (via Ajv or similar)  
- Provider adapters receive JSON Schema directly — no per-provider schema translation  
- MCP tools and built-in tools become truly identical at the schema level  
  
### 3. Decouple `description()` and `prompt()` from Model-Specific Optimization  
  
**Problem:** Tool descriptions and system prompt injections (`prompt()`) are Claude-optimized. Other models may need different instruction styles, verbosity levels, or formatting.  
  
**Overhaul:** Introduce **prompt profiles per provider**:  
```  
Tool {  
  prompt: {  
    default: "...",           // Baseline instructions  
    anthropic: "...",         // Claude-optimized (current)  
    openai: "...",            // GPT-optimized  
    google: "...",            // Gemini-optimized  
  }  
}  
```  
Or better: a `promptAdapter(provider, basePrompt)` function that transforms base instructions into provider-optimal format. This is critical because Claude and GPT respond very differently to the same tool instructions.  
  
### 4. Unify the Streaming Event Schema  
  
**Problem:** The entire system is built around `BetaRawMessageStreamEvent` with its `content_block_start → content_block_delta → content_block_stop` lifecycle. OpenAI uses `choices[0].delta` chunks. These are fundamentally different event shapes.  
  
**Overhaul:** Define a **Universal Stream Protocol**:  
```  
StreamEvent =  
  | { type: 'text_delta', text: string }  
  | { type: 'tool_call_start', id: string, name: string }  
  | { type: 'tool_call_delta', id: string, partialInput: string }  
  | { type: 'tool_call_end', id: string }  
  | { type: 'thinking_delta', text: string }  
  | { type: 'usage', input: number, output: number }  
  | { type: 'done', stopReason: 'end_turn' | 'tool_use' | 'max_tokens' }  
```  
Each provider adapter transforms its native events into this schema. The `StreamingToolExecutor` consumes only universal events — it can begin parallel tool execution on `tool_call_delta` regardless of provider.  
  
### 5. Abstract the Permission Model Away from Tool Format  
  
**Problem:** `checkPermissions()` and `preparePermissionMatcher()` use AST parsing of Bash commands and Anthropic-specific tool names. The `yoloClassifier` assumes Claude's safety behavior.  
  
**Overhaul:**  
- Make `preparePermissionMatcher()` return a **provider-independent risk assessment** (read-only, destructive, network-access, etc.) rather than pattern-matching on command strings  
- Replace `yoloClassifier` with a **pluggable safety classifier interface** — each provider can supply its own, or use a universal rule-based classifier  
- Permission rules should match on **semantic actions** (file-write, shell-exec, network-request) not tool names  
  
### 6. Memory System: From File-Based to Hybrid Store  
  
**Problem:** As discussed, the memory system uses Sonnet side-queries on frontmatter, capped at ≤5 recalls, with no embeddings.  
  
**Overhaul for provider-agnostic:**  
- **Embedding provider as a pluggable interface** — `EmbeddingProvider.embed(text): Vector` with implementations for OpenAI, Gemini, Cohere, local models  
- **Vector index for recall** — Replace `selectRelevantMemories()` Sonnet call with nearest-neighbor search. This removes the dependency on any specific LLM for memory retrieval  
- **Memory write via any model** — The 4-type classification (user/feedback/project/reference) is model-independent. Any model can write memories if given the frontmatter schema  
  
### 7. Replace Feature Flags with a Capability Registry  
  
**Problem:** The current system uses `feature('KAIROS')`, `feature('COORDINATOR_MODE')`, `BUILD_TARGET` macros, and `process.env.USER_TYPE === 'ant'` checks scattered throughout. This is Anthropic-internal complexity.  
  
**Overhaul:** A **declarative capability registry**:  
```  
capabilities: {  
  streaming: true,  
  thinking: provider.supportsThinking(),  
  parallelTools: provider.supportsParallelToolUse(),  
  caching: provider.supportsCaching(),  
  signatures: false,  // Only Anthropic  
}  
```  
Tools and loop logic check capabilities, not feature flags. This makes the harness self-describing — adding a new provider means declaring its capabilities, and the system adapts automatically.  
  
---  
  
**Bottom line:** The tool system's *execution chain* (validate → permission → execute → budget → render) is provider-agnostic in logic. The coupling is in **data formats** (event schemas, tool schemas, message formats) and **model-specific optimizations** (prompts, safety classifiers, caching). A truly agnostic harness needs adapter layers at these boundaries while preserving the battle-tested orchestration logic.  
  
```suggestions  
(Tool system design)[/docs/tools/what-are-tools]  
(Streaming & event types)[/docs/conversation/streaming]  
(Agentic Loop & tool execution)[/docs/conversation/the-loop]  
```  
