# tool-call-replay

Replay recorded AI agent tool calls as pytest regressions.

[![PyPI](https://img.shields.io/pypi/v/tool-call-replay)](https://pypi.org/project/tool-call-replay/)
[![CI](https://github.com/auraoneai/tool-call-replay/actions/workflows/ci.yml/badge.svg)](https://github.com/auraoneai/tool-call-replay/actions/workflows/ci.yml)
[![License: MIT](https://img.shields.io/badge/license-MIT-blue.svg)](https://github.com/auraoneai/tool-call-replay/blob/main/LICENSE)

## Verified Quickstart

```bash
python -m pip install "tool-call-replay==0.1.1"
# from a source checkout (fixtures live in examples/)
tool-call-replay ingest examples/recovered_trace.jsonl --out refund-replay.json
tool-call-replay run refund-replay.json --assert examples/refund_assertions.yaml --report report.md
tool-call-replay pytest refund-replay.json --assert examples/refund_assertions.yaml --out test_refund_replay.py
```

```text
| Result | Assertion | Detail |
| --- | --- | --- |
| pass | `tool_order` | observed=['lookup_order', 'issue_refund'] expected=['lookup_order', 'issue_refund'] |
| pass | `final_answer_contains` | expected substring `once` in `Refund completed once.` |
| pass | `max_retries` | retry counts={'lookup_order': 1, 'issue_refund': 1} max_retries=0 |
| pass | `forbidden_tools` | forbidden used=[] |
```

That table is an excerpt of `report.md`; each command exits `0`.

The recovered refund fixture passes its tool order, terminal answer, retry, and
forbidden-tool assertions. `examples/failed_refund_agent_trace.jsonl` is an
intentional failing fixture and returns exit code `1` with the same assertions.
`test_refund_replay.py` is an ordinary pytest file you can commit next to the
replay JSON.

## Install

```bash
python -m pip install "tool-call-replay==0.1.1"
```

The CLI has no runtime dependencies and requires Python 3.10 or newer.

## At a Glance

| | |
| --- | --- |
| Job | Turn a recorded tool-call sequence into a repeatable regression artifact. |
| Built for | Agent developers, eval engineers, and CI owners debugging tool-use failures. |
| Differentiator | Replays recorded structure and outputs without a model, provider API, or live tool. |
| Produces | Canonical replay JSON, assertion results, Markdown reports, and generated pytest code. |

## What It Does and Doesn't Do

It does:

- normalize a recorded agent trace (JSONL, OpenAI-style events, OTLP or
  Phoenix-style `gen_ai.tool.*` spans) into one canonical replay file;
- check tool order, final-answer text, retries, forbidden tools, and required
  argument keys against that recording;
- write a Markdown report and a generated pytest test for CI.

It doesn't:

- call a model, execute a tool, or make a network request;
- prove that the current live model or tool stack still behaves the same way;
- parse every provider or SDK export variant, or scrub general PII.

## CI Usage

Commit the replay JSON and generated test, then run them with pytest in GitHub
Actions:

```yaml
- uses: actions/checkout@v4
- uses: actions/setup-python@v5
  with:
    python-version: "3.12"
- run: python -m pip install "tool-call-replay==0.1.1" pytest
- run: pytest test_refund_replay.py
```

## Accepted Input Profiles

- `jsonl`: normalized goal, tool call, tool result, and final-answer events;
- `openai`: the same normalized JSONL field subset under an OpenAI-oriented
  adapter name;
- `otlp`: JSON with flat `gen_ai.tool.*` attributes;
- `phoenix`: the same OTLP-style attribute mapping for Phoenix-like exports.

The v1 importers are intentionally small adapters, not complete parsers for
every SDK or telemetry export version.

## Assertion Surface

- exact tool order;
- final-answer substring;
- maximum retries per tool;
- forbidden tools;
- required argument keys for named tools.

## Runtime, Data, and Network Boundary

- Ingest, replay, report, and pytest generation are local file operations.
- The CLI does not call a model, execute a tool, or make a network request.
- Ingest redacts secret-like keys plus `sk-...` and bearer-token strings. It is
  not a general PII scrubber, and generated replay and pytest files embed the
  remaining recorded arguments and outputs.
- Assertions evaluate recorded events only; they do not prove current live tool
  behavior.

## Limitations

- The v1 importers normalize a narrow subset of supported trace shapes; they do
  not parse every provider or SDK export variant.
- Passing assertions confirms only the recorded replay. It does not prove that
  the current live model or tool stack still behaves the same way.

## Compatibility

The published `auraone-agent-studio-open` CLI declares
`tool-call-replay>=0.1.0` as a runtime dependency for trace import, replay, and
CI export flows.

## Publication Status

Verified on 2026-07-13:

- PyPI: [`tool-call-replay==0.1.1`](https://pypi.org/project/tool-call-replay/0.1.1/)
- GitHub release: [`v0.1.1`](https://github.com/auraoneai/tool-call-replay/releases/tag/v0.1.1)
- No benchmark, reliability, or adoption claim is made by the included fixtures.

## Next Action

Normalize one sanitized failure trace, write the smallest assertion file that
captures the regression, and commit the replay plus generated pytest test.

## Part of the AuraOne Agent Reliability Workflow

`tool-call-replay` is the entry point of the AuraOne Open "agent failure to
regression test" workflow:

- [tool-call-replay](https://auraone.ai/open/tool-call-replay): turn a recorded
  agent failure into a pytest regression.
- [Agent Studio Open](https://auraone.ai/open/agent-studio-open): debug, replay,
  and compare MCP and A2A agent traces.
- [mcp-risk-linter](https://auraone.ai/open/mcp-risk-linter): offline MCP
  server security linter.
- [a2a-contract-test](https://auraone.ai/open/a2a-contract-test): A2A agent
  card validator and contract tests.

Turn recurring agent failures into a managed improvement program. [Talk to AuraOne](https://auraone.ai/contact?from=open:tool-call-replay)
