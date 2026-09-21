---
name: monthly-client-recap
description: Pull last month's marketing numbers into an existing Oviond report and get it to the client — refresh the period, read the widget data back, export a PDF, and schedule the recurring send. Use when the user asks for a monthly recap, a client update, last month's performance, or to schedule or test a report automation.
---

# Monthly client recap

The month has closed and a client needs their report. This skill covers moving
an existing Oviond report to the new period, reading the numbers, and getting it
out — by PDF, by email, or on a schedule that does it every month without you.

Building the report in the first place is a different job — see the
`build-client-report` skill.

## 1. Find the report

`list_clients` → `list_reports` for that client, or `search` when the user names
the report but not the client. Refer to it by name from then on.

## 2. Move it to last month

```
change_report_date_range    resolves the new range and re-fetches every data widget
```

Send the **preset** as `date_range.text` — `Last Month` — not computed dates.
Oviond stores the preset and re-resolves it against the client's timezone, so
the report keeps meaning "last month" next time too. Use `text: "Custom"` with
`current_start` / `current_end` (YYYY-MM-DD) only when the user asked for a
fixed window. Widgets pinned to their own custom range keep it.

If the range is already right and you only want fresh numbers, `refresh_report`
re-runs every widget, and `refresh_widgets` re-runs a chosen few.

Changing only `filters`, `chart`, `sort_order` or `row_limit` does **not**
re-fetch — those are display-side. Changing a metric or dimension does.

## 3. Read the numbers back

The refresh is asynchronous and the tool returns before the data lands.

`get_widget_data` with `source_id` set to the report reads every data widget on
it in one call. It waits up to 15 seconds for in-flight fetches by default, so
usually that's all it takes; widgets still in state `loading` need another call.

Render the result as a visual artifact: one card per widget, in the order
returned, each plotted as the chart type that widget is saved as — a `score`
widget is one large number read from `summary`, a `table` is a table, `map` is
geographic, and the rest are line / area / bar / column / pie / donut / funnel.
`chart` is present while a widget is still fetching, so give a loading widget a
placeholder card of the right shape, then update the **same** artifact on each
poll until nothing is loading.

Once the numbers are in front of you, the recap write-up is the easy part: lead
with what moved, name the channel, and quote the figure.

## 4. Sanity-check before it goes out

`audit_report_health` is read-only. It lists the datasources the client has
connected and the report's automations with their state, and flags no
datasource connected or automations left paused.

For the numbers themselves, read the states from step 3. A widget in state
`demo` means the client hasn't linked that datasource — product behaviour, not
a failure, but it should not reach a client. A widget in `error`, or a
datasource whose account list comes back empty, means the connection is broken;
`test_connection` says which.

## 5. Deliver it

**PDF** — `generate_pdf` is asynchronous: it returns an export id, and
`get_pdf_status` says when the file is ready. Don't promise a link until it
reports done. PDFs are for multi-page REPORTS; a DASHBOARD doesn't export one.

**One-off email** — `send_email` with the report attached or linked.
`list_email_senders` shows which verified sender addresses the account has.

**Every month, automatically** — `create_automation`. Create it **paused**, send
a `test_automation` to the user's own address, and only `unpause_automation`
after they've confirmed what landed. `test_automation` sends a real email.
`get_automation_history` shows what previous runs actually did.

## Confirm before anything outward-facing

Sending an email, activating an automation, deleting, and bulk operations all
reach the client. Ask first, every time.

## Talking to the user

Refer to records by name — "the Acme monthly report" — never by ID. Pass IDs
between tool calls freely; keep them out of the conversation.

Playbooks: https://docs.oviond.com/mcp/playbooks
