---
name: write-executive-summary
description: Write or update the written commentary on an Oviond report — an executive summary, a monthly recap, per-section notes. Use when someone asks to summarise a report's performance, explain what the numbers mean, or draft the write-up that goes to the client.
---

# Writing an Oviond executive summary

The numbers are already in the report. This is about the prose beside them —
the part a person would otherwise write by hand every month.

## 1. Get the numbers the client will see

Read the report, not the vendor:

```
list_widgets       what is on the report
get_widget_data    the numbers those widgets hold, with source_id set to the report
```

`get_widget_data` returns what each widget actually shows — including its
comparison against the previous period, which is where most of the summary comes
from. Writing from the widgets guarantees the prose agrees with the charts
underneath it. Pull separate figures and you will eventually contradict them.

Two things to respect:

- A widget in state `demo` is **not real data**. Never summarise it — the
  datasource is not linked. Say so instead.
- A widget still `loading` has no rows yet. Call again for it.

Only reach for `query_data` when the report genuinely does not cover what is
being asked — a different period, a metric nobody put on a chart.

## 2. Find where the text goes

```
list_widgets                    look for type TEXT
get_widget                      read what is already there
```

Reading a TEXT widget returns **Markdown**, not the raw editor format, so an
existing summary can be edited rather than replaced blind. If there is no text
widget, `add_text_widget` puts one on the page.

## 3. Write it in Markdown

`update_text_widget` and `add_text_widget` render Markdown:

```markdown
## Executive summary

Sessions grew **18%** to 24,180, driven by paid search.

### What worked
- Paid search beat target by 12%
- Email re-engagement lifted returning users

### Watch items
1. CPC is trending up for a third month
2. Organic is flat
```

Supported: `#` `##` `###` headings, `-` and `1.` lists, `**bold**`, `*italic*`,
`~~strike~~`, `` `code` ``, `>` quotes and `[links](url)`. Headings stop at
`###` because the editor's own toolbar does. Tables and images are not
supported — the renderer drops what it has no plugin for, silently.

Plain prose works too; it becomes paragraphs.

### Merge tags

`{{client_name}}`, `{{report_name}}`, `{{report_date_range}}`,
`{{account_name}}`, `{{client_manager}}`, `{{client_website}}`,
`{{client_first_name}}`, `{{report_type}}` and `{{report_url}}` are substituted
when the report renders. They survive being read back and rewritten, so editing
a summary that contains them is safe. Anything else in braces stays literal.

## 4. Check it

`preview_report` renders the live report in the conversation. Read the summary
in place — beside the charts, at the width the client sees it — rather than
trusting the Markdown. Where the host won't embed an external page, the tool
falls back to the report's shareable URL; open that instead.

## Writing it well

**Lead with the number that moved.** "Sessions grew 18%" beats "This month saw
growth in sessions."

**Say why, or say you cannot.** A summary that only restates the charts adds
nothing. If the cause is not in the data, write that plainly rather than
inventing one.

**Name the period.** The report's date range is on the report; use it rather
than "this month", which is wrong the moment anyone reads it late.

**Keep the client's own words.** If they call it "enquiries" rather than
"conversions", match them.

**Do not invent context.** Budget changes, campaign launches, seasonality and
site outages explain a lot and appear nowhere in the data. Ask, or leave them
out.

## Confirm before it reaches the client

Replacing an existing summary overwrites what was there — `update_text_widget`
replaces the whole body, it does not append. Show the draft first when you are
rewriting rather than filling something empty.
