<img src="https://r2cdn.perplexity.ai/pplx-full-logo-primary-dark%402x.png" style="height:64px;margin-right:32px"/>

# openai responses api websockets mode codex compatibility with claude code harness

OpenAI’s Responses API WebSocket mode works fine with Codex, and Claude’s “code harness” can interoperate as long as it talks an OpenAI‑compatible HTTP API, but there are a few caveats around transport and endpoints.[^1_1][^1_2][^1_3][^1_4][^1_5]

## Responses API WebSocket basics

- WebSocket mode is just an alternative transport for the Responses API at `wss://api.openai.com/v1/responses`.[^1_6][^1_7][^1_1]
- You send `response.create` events over the socket and use `previous_response_id` to continue the conversation without resending full context.[^1_8][^1_1][^1_6]
- It is designed for **tool‑heavy, long‑running** agent workflows, giving sizable latency gains over repeated HTTPS calls, especially on multi‑tool code jobs.[^1_2][^1_7][^1_9][^1_1]


## Codex + Responses/WebSockets

- Codex (the OpenAI coding agent tooling) can target **any model that supports the Responses API**, and WebSocket is just another transport layer for that.[^1_5][^1_1][^1_2]
- OpenAI is deprecating legacy Chat Completions use in Codex; the recommended path is Responses, so any new work should assume Responses (and optionally WebSocket) rather than chat.[^1_10][^1_5]
- In practice, that means you configure Codex with a model like `gpt-5.3-codex` on the Responses API, and you may opt into WebSocket mode for better performance on large, multi‑file coding tasks.[^1_2][^1_10][^1_5]


### Example mental model

- HTTP: stateless calls to `POST /v1/responses` with full context each turn.
- WebSocket: persistent `wss://.../v1/responses` connection, incremental messages + `previous_response_id`, same Responses schema and models.[^1_1][^1_8][^1_6]


## Claude “code harness” compatibility

- Anthropics’ docs explicitly show an **OpenAI‑SDK‑compatible layer**, where you set `base_url="https://api.anthropic.com/v1/"` and call `openai` SDK methods (e.g., `chat.completions.create`) against Claude models.[^1_4]
- For self‑hosted or third‑party setups, people already use OpenAI‑compatible proxies like LiteLLM as a bridge between Claude‑style backends and OpenAI‑style frontends, including code tools.[^1_3][^1_11]
- The common pattern is: “code harness” (or IDE) → OpenAI‑compatible HTTP API → adapter/proxy → Claude or local model.[^1_11][^1_3][^1_4]


## Where WebSockets fit with Claude harnesses

- Most current Claude/OpenAI‑compat layers (Anthropic base_url compatibility, LiteLLM, etc.) are oriented around **HTTP** endpoints and Chat/Responses‑like schemas, not WebSocket transports.[^1_12][^1_3][^1_4]
- To get *full* Responses WebSocket support with a Claude‑centric harness, you’d need a proxy that:
    - Accepts WebSocket connections at an OpenAI‑style `/v1/responses` endpoint.
    - Speaks the Responses event protocol (`response.create`, `response.output_text.delta`, etc.).
    - Translates those calls into Claude’s API (currently HTTP) and streams back results in the expected event format.[^1_8][^1_12][^1_1]
- Off‑the‑shelf, Claude’s official OpenAI‑SDK compatibility is documented for HTTP, so WebSocket support would depend on your proxy or tooling rather than Claude’s API itself.[^1_12][^1_4]


## Practical guidance

- If your “code harness” (e.g., an IDE plugin or agent framework) already speaks OpenAI **Responses over HTTP**, it should be able to target Claude via an OpenAI‑compatible proxy with minimal changes.[^1_3][^1_11][^1_4]
- If you want to exploit **WebSocket mode’s** latency and incremental‑context benefits, check whether your harness and proxy explicitly support the Responses WebSocket event protocol; if they only support HTTP, you’ll be limited to standard Responses calls.[^1_6][^1_1][^1_2][^1_8]

What specific harness or IDE are you planning to use (e.g., Claude Code, Cursor, VS Code extension), so I can give you wiring steps for that exact setup?
<span style="display:none">[^1_13][^1_14][^1_15]</span>

<div align="center">⁂</div>

[^1_1]: https://developers.openai.com/api/docs/guides/websocket-mode/

[^1_2]: https://x.com/cline/status/2026031848791630033

[^1_3]: https://github.com/anthropics/claude-code/issues/7178

[^1_4]: https://platform.claude.com/docs/en/api/openai-sdk

[^1_5]: https://developers.openai.com/codex/models/

[^1_6]: https://aiengineerguide.com/blog/openai-responses-api-websocket/

[^1_7]: https://www.linkedin.com/posts/jeremy-live_websocket-mode-openai-api-activity-7431841271327047680-Qzt8

[^1_8]: https://learn.microsoft.com/en-us/answers/questions/5788186/is-websocket-mode-working-on-azure-for-openai-mode

[^1_9]: https://apichangelog.substack.com/p/2026-09

[^1_10]: https://community.openai.com/t/codex-openai-completions-responses-migration-pack/1357034

[^1_11]: https://masaic-ai.mintlify.app/use-cases/codex-demo

[^1_12]: https://developers.openai.com/api/docs/guides/realtime-websocket/

[^1_13]: https://community.openai.com/t/websockets-for-responses-api/1374906

[^1_14]: https://www.reddit.com/r/kilocode/comments/1pjseea/openai_responses_api_got_mandatory_all_calls_to/

[^1_15]: https://www.morphllm.com/comparisons/opencode-vs-claude-code


---

# what about the actual Claude Code harness, idk enough about coding or programming terms but whatever Claude Code is, but with websockets and also the responses api compatibility, how is that configured

Here's a plain-English breakdown of what **Claude Code** actually is, how it connects to the OpenAI Responses API, and where WebSockets fit in.

## What Claude Code Is

Claude Code is Anthropic's terminal-based coding agent — you install it via `npm`, run `claude` in your terminal, and it reads/writes files, runs shell commands, and edits code on your behalf by calling an AI model in the background. Think of it as an AI assistant that lives in your command line and can actually touch your project files, not just give advice. By default it's locked to Anthropic's own Claude models, but that can be changed.[^2_1][^2_2]

## How OpenAI API Compatibility Works

Claude Code doesn't speak OpenAI's API natively — it speaks Anthropic's API format (`/v1/messages`). To use it with an OpenAI-compatible endpoint (or to route Claude Code through a proxy to any model), you redirect it with two environment variables:[^2_3][^2_4]

```bash
export ANTHROPIC_BASE_URL="http://your-proxy-address:4000"
export ANTHROPIC_AUTH_TOKEN="your-key-here"
```

The proxy (tools like **LiteLLM** or **Bifrost**) sits in the middle and translates the Anthropic-format requests from Claude Code into whatever the downstream provider expects — whether that's OpenAI's Responses API, Bedrock, Vertex, or Azure. So you never write OpenAI API code directly in Claude Code; the proxy handles the format translation invisibly.[^2_3][^2_1]

### Example: LiteLLM proxy setup

```yaml
# config.yaml
model_list:
  - model_name: claude-sonnet-4-5-20250929
    litellm_params:
      model: anthropic/claude-sonnet-4-5-20250929
      api_key: os.environ/ANTHROPIC_API_KEY
  - model_name: openai-gpt5
    litellm_params:
      model: openai/gpt-5         # OpenAI Responses API on the other side
      api_key: os.environ/OPENAI_API_KEY
```

Then start the proxy with `litellm --config config.yaml` and point Claude Code at it.[^2_3]

## Where WebSockets Fit

Claude Code itself streams responses over **HTTP Server-Sent Events (SSE)**, not WebSocket. SSE is simpler — it's one-way (server to client) and works through standard HTTP proxies without special configuration. This is why Claude Code works through LiteLLM and similar tools without needing WebSocket-aware infrastructure.[^2_5][^2_6]

OpenAI's **Responses API WebSocket mode** lives on the *other side* of your proxy — it's the transport your proxy might use to talk to OpenAI's models for lower latency. Claude Code itself never touches it; your proxy handles that connection entirely behind the scenes.[^2_7][^2_8]

Here's a simple picture of the data flow:

```
Claude Code (terminal)
  → HTTP/SSE to proxy (LiteLLM / Bifrost)
    → proxy translates + connects to OpenAI Responses API (WebSocket optional)
    → proxy translates back → streams result to Claude Code
```


## Routing Claude Code to Any Provider

The cleanest all-in-one approach is **Bifrost CLI**, which auto-configures all the environment variables for you:[^2_1]

```bash
# Step 1: Start the gateway
npx -y @maximhq/bifrost

# Step 2: Launch the interactive CLI
npx -y @maximhq/bifrost-cli
# Then just pick: Claude Code → pick your model (openai/gpt-5, gemini/..., etc.)
```

It handles the Anthropic↔OpenAI format translation, and you can switch models mid-session with `/model openai/gpt-5` inside Claude Code. One hard requirement: **the downstream model must support tool calling**, since Claude Code depends on it for all file and shell operations.[^2_1]

## Quick Config Reference

| What you want | Key env var(s) |
| :-- | :-- |
| Point Claude Code at a local proxy | `ANTHROPIC_BASE_URL=http://localhost:4000` [^2_3] |
| Use AWS Bedrock directly | `CLAUDE_CODE_USE_BEDROCK=1` + `AWS_REGION=...` [^2_9] |
| Override the default model | `ANTHROPIC_DEFAULT_SONNET_MODEL=your-model-name` [^2_3] |
| Skip Bedrock auth (via Bifrost) | `CLAUDE_CODE_SKIP_BEDROCK_AUTH=1` [^2_1] |

The bottom line: you configure Claude Code to point at a proxy via `ANTHROPIC_BASE_URL`, the proxy handles the OpenAI Responses API format and WebSocket transport if needed, and Claude Code itself stays blissfully unaware of all that.[^2_3][^2_1]
<span style="display:none">[^2_10][^2_11][^2_12][^2_13][^2_14][^2_15][^2_16][^2_17]</span>

<div align="center">⁂</div>

[^2_1]: https://www.getmaxim.ai/articles/how-to-run-claude-code-with-openai-gemini-bedrock-and-any-llm-using-bifrost-cli/

[^2_2]: https://www.reddit.com/r/AI_Agents/comments/1s9cnhq/claude_code_literally_got_forked_to_work_with/

[^2_3]: https://docs.litellm.ai/docs/tutorials/claude_responses_api

[^2_4]: https://docs.getbifrost.ai/cli-agents/claude-code

[^2_5]: https://platform.claude.com/docs/en/build-with-claude/streaming

[^2_6]: https://www.claudepluginhub.com/skills/rohitg00-claude-code-toolkit/websocket-realtime

[^2_7]: https://developers.openai.com/api/docs/guides/websocket-mode/

[^2_8]: https://x.com/cline/status/2026031848791630033

[^2_9]: https://code.claude.com/docs/en/amazon-bedrock

[^2_10]: https://docs.portkey.ai/docs/integrations/libraries/claude-code-bedrock

[^2_11]: https://aws.plainenglish.io/configuring-claude-code-extension-with-aws-bedrock-and-how-you-can-avoid-my-mistakes-090dbed5215b

[^2_12]: https://www.harness.io/blog/harness-mcp-server-redesign

[^2_13]: https://www.reddit.com/r/ClaudeAI/comments/1mvxia7/does_anyone_know_how_to_configure_claude_code/

[^2_14]: https://docs.aws.amazon.com/bedrock/latest/userguide/bedrock-runtime_example_bedrock-runtime_Converse_AnthropicClaude_section.html

[^2_15]: https://news.ycombinator.com/item?id=47033722

[^2_16]: https://builder.aws.com/content/39M5ctlGOnaUTx5qNZNgqjh0oou/claude-code-on-bedrock-part-1-getting-started

[^2_17]: https://github.com/affaan-m/everything-claude-code


---

# Alright let's remove the middle layer, no proxies, no HTTP and we definitely need to keep the responses API as well as web sockets mode by openapi: [https://developers.openai.com/api/docs/guides/websocket-mode\#continue-with-incremental-inputs](https://developers.openai.com/api/docs/guides/websocket-mode#continue-with-incremental-inputs)

The honest architectural reality here is important to understand first. Claude Code **cannot** be configured to speak OpenAI's WebSocket Responses API directly — its transport layer is hardwired to send Anthropic-format HTTP requests, and the only knob available (`ANTHROPIC_BASE_URL`) just redirects those same HTTP requests elsewhere. There is no WebSocket transport setting inside Claude Code itself.[^3_1][^3_2]

So to go **truly proxy-free and WebSocket-native** with the Responses API, you have two real options: patch Claude Code's SDK internals (it's open source), or write a **custom agentic harness** that replicates Claude Code's loop but speaks the Responses API over WebSocket directly. The second is far more practical. Here's how it works end-to-end based on the official spec:[^3_3]

## How the Direct WebSocket Harness Works

The OpenAI Responses API WebSocket endpoint lives at `wss://api.openai.com/v1/responses`. You open one persistent connection and drive the entire agentic tool-call loop over it — no HTTP per turn, no proxy in between.[^3_3]

The agentic loop looks like this:

```
connect WSS → response.create (first turn)
     ↓
receive streaming events (response.output_item.done etc.)
     ↓
execute tool locally (read file, run shell cmd, etc.)
     ↓
response.create (previous_response_id + tool output only) ← only incremental input
     ↓
repeat until done → close connection
```

Each continuation sends only the new tool output and the next user message — not the full history — because the server caches the last response state in memory on the active connection.[^3_3]

## Full Working Harness (Python)

```python
import json, os, subprocess
from websocket import create_connection

OPENAI_API_KEY = os.environ["OPENAI_API_KEY"]
MODEL = "gpt-5.4"

# --- Tool definitions (mirrors what Claude Code does natively) ---
TOOLS = [
    {
        "type": "function",
        "name": "read_file",
        "description": "Read a file from the local filesystem.",
        "parameters": {
            "type": "object",
            "properties": {"path": {"type": "string"}},
            "required": ["path"],
        },
    },
    {
        "type": "function",
        "name": "write_file",
        "description": "Write content to a file.",
        "parameters": {
            "type": "object",
            "properties": {
                "path": {"type": "string"},
                "content": {"type": "string"},
            },
            "required": ["path", "content"],
        },
    },
    {
        "type": "function",
        "name": "run_shell",
        "description": "Run a shell command and return stdout.",
        "parameters": {
            "type": "object",
            "properties": {"command": {"type": "string"}},
            "required": ["command"],
        },
    },
]

# --- Local tool executor ---
def execute_tool(name: str, args: dict) -> str:
    if name == "read_file":
        try:
            return open(args["path"]).read()
        except Exception as e:
            return f"ERROR: {e}"
    elif name == "write_file":
        open(args["path"], "w").write(args["content"])
        return f"Written to {args['path']}"
    elif name == "run_shell":
        result = subprocess.run(
            args["command"], shell=True, capture_output=True, text=True
        )
        return result.stdout or result.stderr
    return "Unknown tool"

# --- Collect all events for one response.create turn ---
def collect_turn(ws) -> dict:
    tool_calls = []
    response_id = None
    output_text = ""

    while True:
        raw = ws.recv()
        event = json.loads(raw)
        etype = event.get("type", "")

        if etype == "response.created":
            response_id = event["response"]["id"]

        elif etype == "response.output_text.delta":
            chunk = event.get("delta", "")
            print(chunk, end="", flush=True)
            output_text += chunk

        elif etype == "response.output_item.done":
            item = event.get("item", {})
            if item.get("type") == "function_call":
                tool_calls.append({
                    "call_id": item["call_id"],
                    "name": item["name"],
                    "args": json.loads(item.get("arguments", "{}")),
                })

        elif etype == "response.done":
            print()  # newline after streaming
            break

        elif etype == "error":
            raise RuntimeError(event)

    return {"response_id": response_id, "tool_calls": tool_calls, "text": output_text}

# --- Main agentic loop ---
def run_agent(user_task: str):
    ws = create_connection(
        "wss://api.openai.com/v1/responses",
        header=[f"Authorization: Bearer {OPENAI_API_KEY}"],
    )

    # --- Turn 1: initial task ---
    ws.send(json.dumps({
        "type": "response.create",
        "model": MODEL,
        "store": False,   # ZDR-compatible, no data retained server-side
        "instructions": (
            "You are a coding agent. Use tools to read/write files "
            "and run shell commands to complete tasks."
        ),
        "input": [{
            "type": "message",
            "role": "user",
            "content": [{"type": "input_text", "text": user_task}],
        }],
        "tools": TOOLS,
    }))

    turn = collect_turn(ws)
    previous_id = turn["response_id"]

    # --- Continuation loop: execute tools and feed back results ---
    while turn["tool_calls"]:
        new_inputs = []
        for tc in turn["tool_calls"]:
            print(f"\n[tool] {tc['name']}({tc['args']})")
            result = execute_tool(tc["name"], tc["args"])
            print(f"[result] {result[:200]}")
            new_inputs.append({
                "type": "function_call_output",
                "call_id": tc["call_id"],
                "output": result,
            })

        # Send ONLY the new tool outputs — no full history re-send
        ws.send(json.dumps({
            "type": "response.create",
            "model": MODEL,
            "store": False,
            "previous_response_id": previous_id,  # ← incremental chaining
            "input": new_inputs,
            "tools": TOOLS,
        }))

        turn = collect_turn(ws)
        previous_id = turn["response_id"]

    ws.close()
    print("\n[done]")

if __name__ == "__main__":
    run_agent("Refactor the fizz_buzz function in fizzbuzz.py to be more Pythonic.")
```


## Key Things to Know

- **`store: False`** keeps this ZDR-compatible — no conversation data is persisted on OpenAI's servers, just held in connection-local memory for the active socket[^3_3]
- **Incremental inputs** on continuation turns send only `function_call_output` items + any new user message — not the whole conversation, which is the entire latency win[^3_3]
- **60-minute connection limit** — if your task runs long, handle `websocket_connection_limit_reached`, open a new `ws`, and if you need `store=false` without history, compact first via `POST /v1/responses/compact` (one HTTP call to get a compacted window), then resume on the new socket[^3_3]
- **No multiplexing** — one response in-flight per connection; for parallel sub-agent tasks, open multiple `wss://` connections[^3_3]

This is essentially Claude Code's agentic loop architecture (tool call → execute locally → feed result back → repeat) but running natively over WebSocket against OpenAI's Responses API with zero intermediary layers.[^3_4][^3_3]

<div align="center">⁂</div>

[^3_1]: https://www.getmaxim.ai/articles/how-to-run-claude-code-with-openai-gemini-bedrock-and-any-llm-using-bifrost-cli/

[^3_2]: https://docs.litellm.ai/docs/tutorials/claude_responses_api

[^3_3]: https://developers.openai.com/api/docs/guides/websocket-mode/

[^3_4]: https://www.claudepluginhub.com/skills/rohitg00-claude-code-toolkit/websocket-realtime


---

# i want the priority focus on the claude code harness architeure aspects here, because i want the architectural optimality of the claude code harness but i def dont want it doing the http requests crap. We're just freeform thought experimenting here. Again, the actual Claude Code harness or at least an exact architectural replica, i want its backend to be able to have full responses api websocketmode support, and no proxies or bridges whatsoever.. can it be done? all i need is possible

Yes — fully, completely, and with zero compromises. Here's the architectural thought experiment laid out precisely.

## Claude Code's Actual 8-Step Harness Loop

First, the exact internal sequence Claude Code runs:[^4_1]

```
Step 1  — Assemble context
          (system prompt + CLAUDE.md + memory + FULL conversation history)
Step 2  — Call model API (streaming async generator)
Step 3  — Parse response (text blocks + tool_use blocks)
Step 4  — Permission gate (deny → allow → ask user)
Step 5  — Execute tools (reads parallel, writes serial)
Step 6  — Feed results back (tool results appended to conversation)
Step 7  — Context check (too large? compact → loop. else → loop back to Step 2)
Step 8  — Termination (no tool calls in response = done)
```

The question is which steps are **intrinsically tied to HTTP** versus which are just **harness logic** that can run on any transport.

The answer: **Steps 3, 4, 5, 6, and 8 are pure harness logic — transport-agnostic completely.** Only Steps 1, 2, and 7 are architecturally coupled to HTTP, and they're coupled for one reason: **HTTP is stateless.**

***

## The Real Enemy: Statelessness, Not HTTP

Claude Code's entire complexity around **Step 1** (re-assembling full context every turn) and **Step 7** (the 3-tier compaction system) exists solely because HTTP is stateless. Every call is a fresh request — the server knows nothing. So Claude Code must:[^4_2][^4_1]

- Carry the entire growing conversation history as payload every single turn
- Run a compaction subsystem that kicks in at ~92% context capacity with 3 layered strategies[^4_2]
- Trigger context resets as a last resort for very long sessions[^4_3]

This is **not** an intrinsic requirement of the harness. It's a workaround for transport statelessness.

***

## The WebSocket Architectural Inversion

With a persistent WebSocket connection and `previous_response_id`, the server holds the most recent response state in a **connection-local in-memory cache**. The architectural implication is radical:[^4_4]


| Claude Code (HTTP) | WebSocket-Native Harness |
| :-- | :-- |
| Step 1: Assemble full history every turn | Step 1: **Eliminated** — send `previous_response_id` + new inputs only [^4_4] |
| Step 7: 3-tier local compaction subsystem | Step 7: **Collapses to one API call** — `POST /v1/responses/compact` when needed [^4_4] |
| Context grows client-side | Context lives server-side on the connection |
| Token cost scales with conversation length | Token cost is always just the new incremental input [^4_5] |

Steps 3→6 and 8 map **identically** — the harness logic is the same, just the wire events differ:


| Claude Code event | Responses API WebSocket event |
| :-- | :-- |
| `tool_use` block in response | `response.output_item.done` with `type: function_call` [^4_4] |
| Append `tool_result` to messages list | Send `function_call_output` items in next `response.create` |
| Stream text delta | `response.output_text.delta` |
| End of turn (no tool_use) | `response.done` with empty tool calls |


***

## The Three Structural Differences (Not Problems)

**1. Sub-agent isolation**
Claude Code spawns sub-agents as isolated loops with their own message history and restricted tool sets, returning only a 1,000–2,000 token summary to the parent. In the WebSocket-native version, each sub-agent gets **its own WebSocket connection** with its own `previous_response_id` chain. The parent passes the child a task → child runs its own loop → child sends back its summary as a `function_call_output` into the parent's connection. Structurally identical, just each agent owns a socket instead of a message list.[^4_6][^4_2]

**2. Persistent memory (CLAUDE.md)**
Claude Code re-injects `CLAUDE.md` contents into Step 1 every turn because it has to — HTTP stateless. In the WebSocket version, you inject it **once** in the `instructions` field of the very first `response.create`. It lives on the connection for the session's lifetime. No re-injection needed.[^4_4]

**3. Connection lifetime management**
The one genuinely new architectural concern: connections have a 60-minute ceiling. The reconnection handler needs to thread `previous_response_id` across socket boundaries — open new socket, send new `response.create` with `previous_response_id` from the last turn of the old socket. This is about 10 lines of logic and replaces the entire multi-tier compaction system.[^4_4]

***

## The Architectural Verdict

```
Claude Code's harness = the agentic loop (Steps 3–8)
                      + a context management tax (Steps 1 & 7)
                        paid because HTTP is stateless

WebSocket-native replica = the agentic loop (Steps 3–8) ← IDENTICAL
                         + connection lifecycle handler ← ~10 lines
                         + zero context management tax
```

The harness architecture is not just **replicable** — the WebSocket version is strictly architecturally superior on every axis that mattered enough to Claude Code's engineers that they built an entire compaction subsystem around it. The compaction problem **is** the HTTP problem. Remove HTTP, the problem largely disappears.[^4_7][^4_5][^4_3][^4_4]

The OpenAI Agents Python SDK already formalizes this exact pattern as `responses_websocket_session()` — a shared WebSocket context manager that keeps the connection warm across turns and nested agent-as-tool runs with no additional infrastructure. That's the production-validated proof that the architecture works end-to-end.[^4_6]
<span style="display:none">[^4_10][^4_11][^4_12][^4_13][^4_14][^4_15][^4_8][^4_9]</span>

<div align="center">⁂</div>

[^4_1]: https://sidbharath.com/blog/the-anatomy-of-claude-code/

[^4_2]: https://codewithmukesh.com/blog/anatomy-claude-code-session/

[^4_3]: https://www.anthropic.com/engineering/harness-design-long-running-apps

[^4_4]: https://developers.openai.com/api/docs/guides/websocket-mode/

[^4_5]: https://developers.openai.com/api/docs/guides/conversation-state/

[^4_6]: https://openai.github.io/openai-agents-python/ref/responses_websocket_session/

[^4_7]: https://platform.claude.com/cookbook/tool-use-context-engineering-context-engineering-tools

[^4_8]: https://kenhuangus.substack.com/p/the-claude-code-leak-10-agentic-ai

[^4_9]: https://www.louisbouchard.ai/harness-engineering/

[^4_10]: https://www.reddit.com/r/ClaudeAI/comments/1pk230x/i_built_an_autonomous_harness_for_claude_code/

[^4_11]: https://github.com/strands-agents/sdk-python/issues/1957

[^4_12]: https://x.com/iHarnoorSingh/status/2039461945456427407

[^4_13]: https://community.openai.com/t/responses-api-question-about-managing-conversation-state-with-previous-response-id/1141633

[^4_14]: https://www.humanlayer.dev/blog/skill-issue-harness-engineering-for-coding-agents

[^4_15]: https://developers.openai.com/cookbook/examples/responses_api/responses_example/


---

# What role does CLAUDE.md play in harness memory persistence

CLAUDE.md is not persistent memory in the traditional sense — it is **injected static instruction text** that gets re-loaded into the system prompt on every session, not stored state that the agent modifies or learns from at runtime.[^5_1]

## What CLAUDE.md Actually Is

It's a **hierarchy of plain text files** that get discovered and concatenated into a dedicated section of the system prompt called the `Memory` section during Step 1 of the harness loop. The model doesn't write to them during a session — they're pre-authored by you (or your team), read at session start, and held as cached tokens for the entire run.[^5_1]

## The Discovery Hierarchy

A function called `getMemoryFiles()` walks the filesystem from the root **down** to your working directory, loading in this order:[^5_1]

1. `/etc/claude-code/CLAUDE.md` — managed/corporate-level rules
2. `~/.claude/CLAUDE.md` — your personal rules (apply to every project)
3. `~/.claude/rules/*.md` — per-rule granular personal files
4. `/ancestor-dirs/CLAUDE.md` — each parent directory's project rules, root-first
5. `/your-project/CLAUDE.md` — your immediate project's rules (highest priority)
6. `/your-project/.claude/rules/*.md` — granular project rules
7. `/your-project/CLAUDE.local.md` — gitignored local overrides

The **closer to your working directory, the higher the priority**. An `@include` directive inside any file can pull in referenced docs up to 5 levels deep (e.g., `@./docs/api-conventions.md`), so the actual injected text can be far larger than any single file.[^5_1]

## Caching Behavior

This is architecturally significant: the `Memory` section is marked as a **cached section** in the system prompt. It gets tokenized once at the start of the session and reused every turn via the API's prompt caching, never recomputed mid-session unless you run `/clear` or `/compact`. This is why CLAUDE.md lookup is memoized — it runs exactly once per session.[^5_1]

## What It Is NOT

| What it seems like | What it actually is |
| :-- | :-- |
| Persistent learned memory | Static pre-authored instructions — the model doesn't write to it |
| Runtime state | Compile-time context — fixed at session start |
| Agent scratchpad | Human-controlled config files under version control |
| Session-specific | Hierarchically scoped across the whole filesystem |

## The Actual Runtime Memory System

Claude Code has a **separate** true persistence layer — `~/.claude/projects/<slug>/memory/MEMORY.md`. This is what the agent writes to during a session when it needs to remember something across sessions (facts about the codebase, user preferences, past decisions). It gets injected via `getAttachmentMessages()` alongside conversation history, not baked into the system prompt like CLAUDE.md is.[^5_1]

So in architectural terms:

- **CLAUDE.md** = developer-authored standing instructions, loaded as a cached system prompt section, scoped by directory hierarchy — equivalent to the `instructions` field in an OpenAI Responses API `response.create` call[^5_2]
- **MEMORY.md** = agent-written runtime memory, injected as an attachment message each turn — equivalent to a tool result the agent stuffs into context on every continuation

In a WebSocket-native harness, CLAUDE.md maps cleanly to that one-time `instructions` field on the initial `response.create` — injected once, cached by the server for the lifetime of the connection, never re-sent.[^5_2]

<div align="center">⁂</div>

[^5_1]: https://sidbharath.com/blog/the-anatomy-of-claude-code/

[^5_2]: https://developers.openai.com/api/docs/guides/websocket-mode/


---
