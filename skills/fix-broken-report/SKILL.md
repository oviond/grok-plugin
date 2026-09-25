---
name: fix-broken-report
description: Diagnose and repair an Oviond report that is showing demo numbers, blank or stale widgets, or failing to send. Use when someone says a report looks wrong, shows the wrong data, has not updated, or its scheduled email did not arrive.
---

# Fixing a broken Oviond report

"This report looks wrong" is almost always one of four things. Work down the
list — each check is cheap, and the order matters because the later causes are
invisible until the earlier ones are ruled out.

## 1. Read the report's own diagnosis first

`audit_report_health` is read-only and answers most of it in one call: which
datasources the report's client has connected, the state of its automations, and
the two faults that quietly kill a delivery — nothing connected, and automations
left paused.

It does not look at individual widgets. For that, `get_widget_data` with
`source_id` set to the report returns every data widget and its state.

## 2. Read the widget states

Three states are the whole diagnosis:

| State | Means | Fix |
|---|---|---|
| `demo` | The datasource is not linked to this client | Link it — section 3 |
| `error` | The last fetch failed | Test the connection — section 4 |
| `loading` | Still fetching | Wait and call again; not a fault |

**`demo` is not an error.** A widget pointed at a datasource the client has not
connected renders placeholder numbers by design. That is the single most common
cause of "the numbers are wrong" — the report was built from a template before
anything was connected.

Anything else, with rows, is working.

## 3. `demo` — the datasource is not linked

Two separate things have to be true. A **connection** belongs to the account —
one Google Ads login. A **link** points one client at it. A report shows demo
data when the second is missing, even though the first exists.

```
connected_datasources     what the ACCOUNT has connected
link_datasource           point THIS client at one of them
refresh_report            re-fetch, passing datasource_id to do just that one
get_widget_data           confirm the widgets left demo
```

`link_datasource` with only `client_id` and `datasource_id` lists the choices
rather than guessing. Most datasources take one pick; six take two (Google Ads
and Local Services Ads: manager then customer; Meta and Instagram Ads: business
then ad account; GA4: account then property; Business Profile: account then
locations).

If `connected_datasources` does not list the platform at all, nothing can be
linked yet. Google, Meta, LinkedIn and TikTok sign in on the vendor's own
website and need a browser — tell the user to connect it in the app, then come
back and link it here.

## 4. `error` — the fetch failed

`test_connection` says whether a stored connection still authenticates. The
usual answer is an expired OAuth token, which the user re-authorises in the app.

A vendor refusing a specific metric also lands here. If one widget errors while
its neighbours on the same datasource are fine, the problem is that widget's
configuration, not the connection — read it with `get_widget` and check the
metric belongs to the data view it is asking for (`describe_datasource`).

## 5. Numbers that are right but old

A widget's saved numbers are from its last fetch. Nothing re-runs on its own
except an automation.

```
refresh_widgets     specific widgets
refresh_report      everything on the report
```

Both return immediately and fetch in the background — read the result back with
`get_widget_data` rather than assuming it finished.

## 6. The email did not arrive

```
list_automations          is one attached to this report at all?
get_automation_history    what previous runs actually did
list_exports              PDF generation attempts, one row per try
```

The two usual answers: there is no automation, or there is one and it is paused.
`unpause_automation` resumes it — **this puts email back on the wire**, so
confirm with the user first and offer `test_automation` to their own address
before it goes to the client.

## What not to do

Do not rebuild widgets that are showing demo data. The configuration is fine;
the link is missing. Deleting and recreating loses the layout and fixes nothing.

Do not `refresh_report` as a first move. It costs a round trip to every vendor
on the report and will not change anything if the cause is an unlinked
datasource or a paused automation.
