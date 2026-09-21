---
name: build-client-report
description: Stand up a new client in Oviond and build their first report — create the client, link the datasources they already have connected, add data widgets per channel, and set the reporting period. Use when the user asks to onboard a client, build a report or dashboard, or add widgets to one.
---

# Build a client report

Oviond is a client reporting platform for marketing agencies. This skill covers
going from "we just signed Acme" to a report with real numbers in it.

## The shape

```
account
└── client                the agency's customer
    └── report            a REPORT (many pages, PDF-able) or a DASHBOARD (one page)
        └── page
            └── widget    a DATA widget pulls numbers; the rest are title/text/image/button/embed
```

A **connection** belongs to the account — one Google Ads login. A **link**
points one client at that connection. Several clients can share one login, each
reporting on its own ad account. Connecting and linking are two separate steps.

## Start here

`create_client_with_report` does the whole first step in one call: the client,
and a first report built from a template. Use `list_templates` first if the user
has a house template; otherwise let it pick the default.

For an existing client, `create_report` on its own, or
`create_report_from_template`.

Then `list_pages` for the `page_id` — cheap, and worth calling rather than
assuming a dashboard has only one page.

## Link the datasources before adding widgets

```
connected_datasources     what this account has already connected
link_datasource           point this client at one, and pick the ad account
```

`link_datasource` called with just a client and a datasource lists the accounts
to choose from. Most take one pick. Six take two: Google Ads (manager, then
customer), Meta and Instagram Ads (business, then ad account), GA4 (account,
then property), Business Profile (account, then one or more locations).

**OAuth datasources cannot be connected from here.** Google, Meta, LinkedIn,
TikTok and the rest sign in on the vendor's own website, which needs a browser.
`create_connection` refuses them and says so — tell the user to connect it once
in the Oviond app, then come back. API-key datasources do work:
`create_connection` with just a datasource and a name replies with the exact
fields it needs, and only accepts the keys it named.

## Add the widgets

Never guess field ids:

1. `datasources` → the `datasource_id` (`ga4`, `gadw`, `fb-ads`, …)
2. `describe_datasource` → its data views, and the metrics, dimensions,
   filter operators and required settings inside each

A metric is only valid inside its own data view. A wrong id is rejected with the
valid list, so read the error instead of retrying blindly.

Then `add_data_widget`, one per chart, table or scorecard. Position and size
default to the next free slot, so usually don't pass them; `move_widget`
repositions afterwards.

Four widget types are not vendor datasources:

| Want | `datasource_id` | Carries |
|---|---|---|
| A typed-in number or word | `CUSTOM_DATA` | `advanced.value` |
| A goal's progress | `GOALS` | `advanced.goal_id` from `list_goals` |
| An uploaded dataset | `CUSTOM_IMPORT` | `advanced.custom_data_id` + metrics + a dimension |
| A calculated metric | `CALCULATION` | the metric id, passed as the single **metric** |

`CALCULATION` is the odd one out — its id goes in `metrics`, not `advanced`. A
goal or calculated metric has to exist first (`create_goal`,
`create_calculated_metric`), then the widget that points at it.

`add_report_section` drops in a whole block of widgets at once when a section
already exists as a saved asset (`list_assets`).

## Reading the numbers back

Data is fetched **server-side, right after the widget is written**. There is no
ad-hoc query tool. The write returns before the numbers land, so a widget in
state `loading` straight after a write is normal.

`get_widget_data` takes either `widget_ids` (up to 200 — the widgets you just
added) or `source_id` (every data widget on one report). It waits up to 15
seconds for in-flight fetches by default, so usually one call is enough. Each
entry carries the widget's state: `loading` is still fetching, `demo` means the
datasource isn't linked, `error` means the fetch failed.

Render the result as a visual artifact — one card per widget, plotted as the
chart type that widget is saved as — and update the same artifact on each poll.
Never dump it as a markdown table.

## The date range

A report's range is stored as the **preset itself** — "Last 30 Days" keeps
meaning the last 30 days as the calendar moves. Send the preset as
`date_range.text`, not computed dates; the server resolves it against the
client's timezone. `change_report_date_range` resolves the new range and
re-fetches every data widget, except ones pinned to their own custom range.
Use `text: "Custom"` with `current_start` / `current_end` (YYYY-MM-DD) only for
a fixed window.

A report created from a template does **not** inherit the template's date range
— it starts on the account default. Set it explicitly if the user named a period.

## Finish

`audit_report_health` is read-only and worth running as the last step. It lists
the datasources the client has connected and the report's automations with their
state, and flags the two things that quietly break a delivery: no datasource
connected, and automations left paused. It does not inspect individual widgets
— `get_widget_data` is what tells you whether they returned anything.

## Things that look wrong but aren't

- **Demo data.** A widget pointed at a datasource the client hasn't linked
  renders demo numbers. That's the product behaviour. Link it and refresh.
- **A datasource with no accounts to pick.** Stripe, Shopify and Klaviyo have
  nothing to choose — the connection *is* the link.
- **An empty account list where there should be one.** The connection is broken
  — expired token or revoked access. `test_connection` says which.
- **"Access denied" on a report.** Usually the signed-in user's role, not a
  broken connector. The connection has exactly the permissions of the person who
  signed in.

## Talking to the user

Refer to records by name — "the Acme monthly report" — never by ID. Pass IDs
between tool calls freely; keep them out of the conversation.

Full tool catalog: https://docs.oviond.com/mcp/tools
