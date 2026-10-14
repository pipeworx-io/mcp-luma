# @pipeworx/luma

Public event discovery from Luma (lu.ma): curated city feeds, one event's
full detail, and an organizer's public calendar — the same data a browser
sees on lu.ma/discover and on each event's own public page.

Part of [Pipeworx](https://pipeworx.io) — an MCP gateway connecting AI agents to 1800+ live data sources. This is an independent, unofficial integration — not affiliated with, endorsed by, or published by the upstream provider.

## Tools

- `luma_places(query?, limit?)` — list Luma's curated discover cities, with
  place_id, coordinates, timezone and live event_count. Use to find a
  place_id for `luma_events`, or just pass a city name directly.
- `luma_events(city? | place_id?, query?, limit?, cursor?)` — list public
  upcoming events for a discover city. Pass a city name (resolved for you
  against `luma_places`) or a `place_id`. An optional keyword filters on
  event name/venue/organizer, matched client-side.
- `luma_event(event_id)` — full public detail for one event: description,
  timing, venue, categories, ticket types, FAQs, hosts and calendar.
- `luma_calendar_events(calendar_id, limit?, cursor?)` — public upcoming
  events on an organizer's calendar page (a company's or community's event
  series).

None of these return guest lists or attendee data — see "Privacy" below.

## Auth

Keyless. No account, API key or registration needed — these are Luma's own
public discovery endpoints, the same ones a signed-out browser calls.

## Data sources

- Luma, public event-discovery feed — city "discover" pages, individual
  event pages, and organizer calendar pages, all served to any visitor
  without an account.

## Privacy

Luma's own payload for an event carries per-event attendee signal: a guest
count, and for well-attended events a "featured guests" list naming and
showing avatars for people who RSVP'd, plus (for an authenticated caller)
their own RSVP record. This pack strips all of it before returning anything
— `guest_data`, `featured_guests`, `guest_info`, `host_info` and
`manager_info` are dropped in every tool. Only the event itself, its public
organizer/host listing (the people or orgs who publicly listed themselves as
running the event — a different thing from attendees), and its calendar are
returned.

## Notes

- `luma_events`'s `query` filters client-side. Luma's own `query` parameter
  on the discover-events endpoint does not actually filter server-side
  (verified live: identical result sets with and without it for the same
  place), so this pack paginates the upstream itself (up to 200 events) and
  matches on event name, venue and organizer name instead.
- Place/city resolution (`luma_events` given a `city`) matches against the
  live discover-places list by exact name/slug first, then by substring.
  There is no local cache of places — each resolution is a fresh call, since
  the list is small (under 100 cities) and changes as Luma launches new
  ones.
- Event and calendar IDs (`evt-...`, `cal-...`) are opaque Luma identifiers.
  The easiest way to get one is from a prior `luma_events` or
  `luma_calendar_events` call — every returned event carries its own
  `event_id`, and its `calendar.url` leads to the organizer's public page.

## Quick Start

Add to your MCP client (Claude Desktop, Cursor, Windsurf, etc.):

```json
{
  "mcpServers": {
    "luma": {
      "url": "https://gateway.pipeworx.io/luma/mcp"
    }
  }
}
```

### What this endpoint actually serves

`tools/list` at `https://gateway.pipeworx.io/luma/mcp` returns the tools in the table
above **plus the shared Pipeworx meta-tools** — `ask_pipeworx`,
`discover_tools`, `search_within`, `remember`/`recall` and the rest of the
gateway-wide set. So the tool count you see is larger than this table: a
single-pack endpoint currently lists roughly 30 shared tools alongside the
pack's own. The connection's `initialize` response states its exact scope, and
is the authoritative answer for a given day.

This is deliberate, not multiplexing by accident. The meta-tools are what let a
scoped connection answer a question this pack does not cover — via
`ask_pipeworx`, which routes across the whole catalog — without you adding a
second MCP server. There is currently no way to mount a pack endpoint without
them; if the extra schemas cost you more context than the routing is worth,
connect to the full gateway once rather than to several pack endpoints.

Or connect to the full Pipeworx gateway to get every pack's tools listed
directly, instead of just this one's:

```json
{
  "mcpServers": {
    "pipeworx": {
      "url": "https://gateway.pipeworx.io/mcp"
    }
  }
}
```

Both URLs reach the same gateway and the same 1800+ data sources. The
only difference is which pack's tools are listed **directly**; `ask_pipeworx`
reaches all of them from either one.

## No MCP client? Call it over HTTP

```bash
curl -X POST https://gateway.pipeworx.io/v1/tools/luma_places \
  -H 'Content-Type: application/json' \
  -d '{"query":"san","limit":10}'
```

No account needed for the first calls. Inspect any tool: `GET https://gateway.pipeworx.io/v1/tools/luma_places`. Find one: `POST https://gateway.pipeworx.io/v1/tools/search_packs` with `{"query":"..."}`.

## Standalone (no gateway account)

This package also runs as a local stdio MCP server — no Pipeworx account, no
gateway round-trip:

```json
{
  "mcpServers": {
    "luma": {
      "command": "npx",
      "args": ["-y", "@pipeworx/mcp-luma"]
    }
  }
}
```

Or run it directly to confirm it starts:

```bash
npx -y @pipeworx/mcp-luma
```

It speaks MCP over stdin/stdout and answers `initialize`/`tools/list`/`tools/call`
for **only** this pack's tools — none of the shared meta-tools the gateway
connection above adds. Same source, same tools, no ask_pipeworx routing.

## Using with ask_pipeworx

Instead of calling tools directly, you can ask questions in plain English —
this works on the pack endpoint above as well as on the full gateway:

```
ask_pipeworx({ question: "your question about Luma data" })
```

The gateway picks the right tool and fills the arguments automatically.

## More

- [Docs and guides](https://pipeworx.io/docs)
- [pipeworx.io](https://pipeworx.io)

## License

MIT
