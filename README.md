# tool-call-replay

`tool-call-replay` converts recorded agent tool-call events into a canonical
local replay file, deterministic assertions, and an optional pytest regression.

## At a Glance

| | |
| --- | --- |
| Job | Turn a recorded tool-call sequence into a repeatable regression artifact. |
| Built for | Agent developers, eval engineers, and CI owners debugging tool-use failures. |
| Differentiator | Replays recorded structure and outputs without a model, provider API, or live tool. |
| Produces | Canonical replay JSON, assertion results, Markdown reports, and generated pytest code. |

## Install

```bash
python -m pip install "tool-call-replay==0.1.1"
```

## Verified Quickstart

Run from a source checkout:

```bash
tool-call-replay ingest examples/recovered_trace.jsonl \
  --out /tmp/refund-replay.json

tool-call-replay run /tmp/refund-replay.json \
  --assert examples/refund_assertions.yaml \
  --report /tmp/refund-replay.md

tool-call-replay pytest /tmp/refund-replay.json \
  --assert examples/refund_assertions.yaml \
  --out /tmp/test_refund_replay.py
```

The recovered refund fixture passes its tool order, terminal answer, retry, and
forbidden-tool assertions. `examples/failed_refund_agent_trace.jsonl` is an
intentional failing fixture and returns exit code `1` with the same assertions.

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
