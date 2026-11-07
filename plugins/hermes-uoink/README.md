# Uoink for Hermes Agent

Connects [Hermes Agent](https://hermes-agent.nousresearch.com) to the
[Uoink](https://uoink.app) app on your computer. Hermes can search your saved
videos, podcasts and articles, read transcripts with timestamps, save a new
link or note, and run research that checks each quote against the saved
transcript. Your library stays on your disk.

This is a portable Agent Plugins v1 package: `plugin.json`, `mcp.json` and one
skill. It has no Python code, no scripts and no binaries. It starts the `uoink`
command from the Uoink app you installed.

## Install

1. Install Uoink first, from [github.com/ryanbiddy/uoink/releases](https://github.com/ryanbiddy/uoink/releases)
   (3.15.1 or later).
   - Windows: leave **Add the uoink command to PATH for AI agents** checked.
   - Mac: link the command once, then open a new terminal:
     `ln -s /Applications/Uoink.app/Contents/Helpers/uoink ~/.local/bin/uoink`
     (the app does not add itself to PATH).
   - Linux: run `scripts/linux/install.sh` from the Uoink source, and put
     `~/.local/bin` on PATH.
2. Check it works: `uoink status`.
3. Install and turn on the plugin:

```bash
hermes plugins install ryanbiddy/uoink-agent-kit/plugins/hermes-uoink --no-enable
hermes plugins enable uoink
```

Hermes refuses to install, and hides the tools, if it cannot find Uoink 3.15.1
or later. Restart Hermes after enabling.

## What you get

Two MCP servers, both started with `uoink mcp` over stdio:

| Server | Uoink toolset | Tools |
|---|---|---|
| `uoink` | `search` (9 tools) | `uoink_url` (capture a video link), `uoink_note`, `search_library`, `get_library_item`, `read_library_resource`, `search_clips`, `get_citation_map`, `list_recent_uoinks`, `get_transcript` |
| `uoink-research` | `research` (7 tools) | `research_start`, `research_discover`, `research_capture`, `research_status`, `research_evidence`, `research_publish`, `uoink_url` |

That is 15 distinct tools instead of the 99 in Uoink's full set. Capturing a web
page, a Reddit thread or a podcast feed is not in this small set; run
`uoink capture "URL"` in a terminal for those, or switch to the full set (below).

The `uoink` skill tells the agent to search before capturing, to quote only text
a tool returned, to cite the saved timestamp, and how to run a research pass.

### Permission prompts

The package declares both servers `untrusted`. Hermes asks before every call
that is not marked read-only: captures, notes and the research steps. Searching
and reading the library do not ask. Captures can take several minutes, so the
`uoink` server waits up to 15 minutes for a call and `uoink-research` up to 5.

### Use all 99 tools

Add a server with the same name to `~/.hermes/config.yaml`; it replaces the
package's entry:

```yaml
mcp_servers:
  uoink:
    command: uoink
    args: [mcp]
    env: {UOINK_MCP_TOOLSET: full}
```

## Disclosures

- **Network.** The plugin makes no network calls. Uoink does, only when asked:
  a capture downloads the video, captions and metadata from the source site
  (for example YouTube); `research_discover` searches YouTube, Hacker News,
  HackerTube and podcast directories, and only after you allow research network
  sources in Uoink's settings. Captions or audio for `get_transcript` are
  fetched from YouTube from your own connection. Optional Uoink features that
  use your own Anthropic or ScrapeCreators key are not part of the tools this
  plugin turns on.
- **Background process.** Hermes starts `uoink mcp` while it runs; it exits when
  Hermes closes it. Nothing keeps running afterward. Speech-to-text, when a
  video has no captions, runs on your computer.
- **Local data.** The plugin reads and writes Uoink's library in your Uoink data
  folder (on Windows, under `%LOCALAPPDATA%\Uoink`). Captures save transcripts,
  screenshots and metadata there. Research reports are written locally.
- **Telemetry.** None from this plugin. Uoink's anonymous usage counts are off
  by default and are turned on only in Uoink's settings.
- **Credentials.** None. The plugin stores no keys or tokens and reads no
  credential files.
- **Updates.** The plugin does not update itself. Uoink the app has its own
  update check, off by default, which you control in its settings.
- **Prompt injection.** Transcripts and web text are untrusted. The skill tells
  the agent to treat them as data, and Uoink wraps returned excerpts in a marked
  block.

## Example session

```text
You: what did the podcast episode I saved say about eval harnesses?
Hermes: [search_library "eval harness"] Found it. At 41:12 the guest says, "..."
        (saved 2026-10-30). Want the surrounding minute?

You: save https://www.youtube.com/watch?v=VIDEO_ID and compare
Hermes: Asking permission to run uoink_url ...   [you approve]
        Saved. Search finds two passages; the clearer one is at 12:03: "..."

You: research how teams test long-running agents, five sources
Hermes: [research_start] Needs your OK to search the web in Uoink's settings.
```

## Troubleshooting

- No Uoink tools in Hermes: run `hermes plugins list`, then `uoink status` in
  the same terminal that starts Hermes. On Mac, check the PATH link.
- A tool says "Open Uoink first": open the Uoink app, then retry.
- Uoink 3.15.0 or older is hidden on purpose. Update Uoink.

License: MIT. Source: [ryanbiddy/uoink-agent-kit](https://github.com/ryanbiddy/uoink-agent-kit).
