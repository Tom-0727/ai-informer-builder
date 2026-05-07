---
doc_id: design.notifications
status: outline
last_edited_at: 2026-05-05
---

# Notifications design — channels, CLI, adapter, config

Lineage: this design evolves the reference impl https://github.com/Tom-0727/AInformer/blob/main/core/utils/inform.py (single `inform(message)` function, env-var URL list, broadcast, platform-by-URL-substring) into a richer adapter pattern with named channels and a config-file source of truth.

## Module location

- Python module path: `ai_informer/notifications/`.
- Per-channel adapters: `ai_informer/notifications/channels/<type>.py` (one file per channel type, e.g. `feishu.py`, `dingtalk.py`).
- CLI entrypoint: `ai_informer/notifications/cli.py`.

## CLI shape

`python -m ai_informer.notifications.cli --message-file <path> [--channel <name>]`

- `--message-file` carries the rendered output (e.g. a window run's `output.md`); reading from a file avoids shell-quoting issues with multiline Chinese content.
- Default behavior (no `--channel`): broadcast — send to every channel listed in the config file.
- `--channel <name>`: restricts the send to a single channel matched by its `name` field in the config.

## Channel-adapter pattern

Each channel is one adapter class in `ai_informer/notifications/channels/<type>.py` exposing a single `send(message: str)` method plus a constructor that takes the per-channel config dict. Discovery is registry-style by the `type` field in config: `type: "feishu"` resolves to `channels/feishu.py`. The CLI iterates configured channels, looks up each adapter by `type`, instantiates it with the entry's config, and calls `send`.

## Config file shape

Location: `config/notifications.json` (NOT created this episode; design only). Top-level shape: an object with a `channels` array; each entry carries:

- `name` — unique identifier the `--channel` flag matches against.
- `type` — selects the adapter file under `channels/`.
- per-type fields (e.g. `webhook_url` for `feishu` and `dingtalk`).

Illustrative entry (placeholder URL only — real URL is authored only in the actual config file at deploy time):

```
{"channels": [{"name": "feishu-default", "type": "feishu", "webhook_url": "<configured-feishu-webhook-url>"}]}
```

## Extensibility recipe

To add a new channel `X`: (1) write `ai_informer/notifications/channels/X.py` with a `send(message)` method; (2) append a config entry with `type: "X"` and X's per-type fields. No edits to `cli.py` or any skill file are needed.

## Integrity invariant

The literal Feishu (or any other) webhook URL is authored in exactly one place — `config/notifications.json` — never in skill files, never in this design file, never in the main design body. Skill Stage 4 invokes the CLI by file path; the CLI reads the config; the config holds the URL.
