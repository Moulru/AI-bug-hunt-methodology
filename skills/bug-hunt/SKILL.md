---
name: bug-hunt
description: Whole-project bug hunt that finds and reproduces real bugs using a coverage table and 20 methods (pyramid, inverted pyramid, scenarios, state transitions, boundaries, external behavior, trust boundaries, navigation lifecycle, UI consistency, and more). Use when asked to hunt for bugs, audit an app before a release, or do a full bug sweep.
---

# AI Full Bug Hunt Test Methodology

Apply the methods below to the entire project and find bugs.

The goal is to find and reproduce real bugs, not to do a plain code review. Do not report a problem that has already been confirmed.

Work in this order:
1. Build the coverage table first (step 0).
2. Apply methods 1 through 18.
3. Run one independent hunt with different eyes (method 19).
4. Cross-verify, then report.
5. When a missed bug turns up later, run a retrospective (method 20).

## 0. Build the coverage table

Applying every method does not mean every screen and every path was checked. So before hunting, build a table, and measure coverage by whether every cell of the table is filled.

- Screen list: enumerate every route, tab, bottom sheet, and dialog. Include rarely opened screens (settings, edit screens, update prompts, error screens).
- Entry paths per screen: tab, opened from another screen, bottom sheet, notification, deep link, restore after app restart, right after sign-in.
- Exit paths per screen: back, close, switch to another tab, jump directly to another screen (for example home), sign-out or session expiry, app termination.
- States per screen: loading, load failed, empty, saving, save failed, cancelled.
- Do not include only recently changed code. Give screens that have not been touched in a long time (sign-in, account deletion, settings, and so on) the same weight.

When the hunt is done, fill each cell with "checked / not checked / verification level" and attach the table to the report.

## 1. Pyramid exploration

Start from the core feature with the most connections to other features.

```text
core feature → connected features → sub-features of the connected features
```

Expand the scope in that order. When the chain of connections ends, restart from the remaining feature with the most connections.

Focus on calls between features, data passing, state changes, API integration, and exception handling.

## 2. Inverted pyramid exploration

Start from the leaf features with the fewest connections to other features.

```text
leaf feature → features that call it → higher-level features → user-facing features
```

Work upward in that order.

In particular, check that return values, null handling, exception propagation, types, and the caller's expectations all match.

Mark every place where a leaf function swallows an error and returns an empty value. Follow that empty value all the way up and check whether a higher layer treats it as a "normal empty value" and shows it or saves it.

## 3. User scenario testing

Build the main scenarios a real user is likely to go through and test them from start to finish.

Focus on problems that appear when several features are used in sequence, rather than a single feature alone.

Include not only normal use but also retry after failure, leaving midway and coming back, and similar scenarios.

Also vary how each scenario ends. Even in the same flow, the leftover state can differ when the user finishes with back, switches to another tab, or taps a button that jumps directly to another screen.

## 4. State transition testing

Track the state changes of each main feature. Do not stop at normal → success. Include these situations:

- failure → retry
- leave midway → come back
- refresh
- back navigation
- duplicate execution
- session expiry
- re-running a task that is already completed or deleted
- the app goes to the background and comes back (a permission dialog or an external sign-in window counts as this)
- the same data is changed or deleted from another device

Find problems where state is wrongly kept, or where a previous state affects a new one.

## 5. Boundary value / exception testing

Besides normal input, deliberately test these conditions:

- null / undefined
- empty string / whitespace
- 0 / negative numbers
- minimum / maximum values
- very large values
- empty array / very large array
- duplicate data
- non-existent IDs
- wrong type or format

Also include API and external service timeouts, network errors, 4xx/5xx responses, and missing fields in response data.

## 6. Breaking implicit assumptions

Find conditions the developer implicitly assumes in the code. For example:

- The value always exists.
- The API always succeeds.
- The user taps only once.
- Requests complete in order.
- The data always exists in the DB.
- The user is always authenticated.
- A screen that was opened always closes and returns.
- If the app decides an operation failed, it also failed on the server.
- The same status code always means the same thing.

Verify whether a bug occurs when each assumption breaks.

## 7. Data flow exploration

Trace the full lifecycle of the main data.

```text
input → API → service → DB → response → frontend → state → UI
```

At each step, check:

- missing data
- wrong field mapping
- type conversion
- null handling
- default values (if an empty value is filled with a default and saved, the user's data changes; also check whether that value is used as input by another feature)
- format conversion
- timezone
- cache / stale data
- data mismatch between layers

If there are several branches that lead to a result, trace each branch separately.

## 8. Contract verification

Check that functions, APIs, the DB, and the frontend agree on what they expect from each other. In particular, check:

- input types
- output types
- nullability
- required / optional values
- error handling
- HTTP status
- the DB schema versus how it is actually used

Find places where the implementation and the caller expect different things. Check contracts with external systems as well as contracts between our own code (see method 13).

## 9. Duplication / consistency exploration (logic)

Find business rules that are implemented in several places in different ways. In particular, check that these do not use different criteria in different places:

- validation
- permission checks
- date/time handling
- status values
- data conversion
- calculation logic

Consistency of what is visible on screen is checked separately in method 18.

## 10. Async / race condition testing

Test situations where requests or events run in an unexpected order.

```text
request A starts → request B starts → B completes → A completes
```

Or the same button is tapped rapidly several times. Focus on these problems:

- race conditions
- duplicate requests / duplicate saves
- an old response overwriting the latest state
- wrong loading state
- optimistic update errors
- debounce / throttle errors
- realtime event ordering
- a delayed save (one that runs a few seconds later) overlapping with a save from another screen

## 11. Reverse permission testing

Instead of a normal user using the app the normal way, test from the point of view of a user directly attempting things they must not be able to do.

- a regular user calling admin features
- reading or changing data with another user's ID
- delete requests without permission
- calling APIs without authentication
- calling APIs with expired authentication
- calling features hidden in the frontend directly through the API

Also check that frontend restrictions match the backend's actual permission checks. Rules other than permissions (consent, limits, states) are checked in method 15.

## 12. Change point / regression exploration

If there is recently changed code, start from the change and trace its impact.

```text
changed code → callers → higher callers → shared data → API → UI → related features
```

In particular, when a shared module, global state, DB schema, or API interface changed, re-check every related feature.

Do not spend more than half of the total time on change-point exploration. Spend the rest on the unchanged screens in the step 0 coverage table.

## 13. Measuring external behavior

Build a table of how the parts we did not write (auth server, database service, server functions, social sign-in SDKs, OS permissions, notifications, app stores) actually respond. Build it by calling them directly or reading their official code, not from documentation or memory.

```text
Table columns: external system / situation / actual response (code, message, latency) / how the app interprets it
```

- Check whether one status code has several meanings. (For example, 403 meaning expired token, missing session, and missing user.)
- Check whether a user cancellation can be told apart from a real error.
- Find places where raw external error text (English developer messages) is shown on screen as is.
- Feed each value from the table into the app one by one and record the result.

## 14. Timeouts and ambiguous outcomes

Enumerate every place with a timeout and check each one:

- Does the timed window include time the user spends typing or choosing? (For example, choosing an account in an external sign-in window.)
- Are timeouts nested so that an outer one fires before the inner one finishes?
- Can the app end an operation as "failed" while the server actually finished it? If so, does the app re-check the result?
- If the user taps again after a failure, is the work processed twice?
- Is there any call with no timeout that can wait forever?

## 15. Does the server enforce it too? (trust boundary)

List every rule the app enforces: consent, permissions, count and size limits, ownership, state (for example, a task that has already finished).

For each rule, call the server API directly, without the app, and check that the request is really blocked. If only the app blocks it and the server allows it, the rule breaks for older app versions, for screens that were already open, and on other devices.

## 16. Navigation lifecycle

Find every piece of code that opens a screen and "cleans up after that screen closes": code that runs after awaiting a pushed screen, and places that wait for a value returned by a screen.

For each one, check whether the cleanup still runs when the user does not close that screen but goes somewhere else directly (jumping to home, switching tabs, opening another screen from a notification). If it does not run, banners, badges, locks, or pending states are left behind.

When you find one instance of this root cause, search the whole codebase for code of the same shape to find the rest.

## 17. Writes after failure, and the side-effect ledger

### Writes after failure
Find places where loading fails, the screen shows empty values, and saving in that state overwrites existing data. Examine every screen that has a catch block which swallows errors from this angle.

### Side-effect ledger
For each finishing action (save, add, delete, finish a check, sign out, delete account), write down what must happen along with it.

```text
Example: finish a check → write the history record, refresh the recent list
Example: sign-out and session expiry → cancel scheduled notifications, clear local state
```

Check that everything in the ledger happens on every path that leads to the result. Also check what the user sees when only part of the steps fails (for example, the save succeeded but scheduling the notification failed), and what happens if they tap again.

## 18. Screen consistency and wording

Separately from functional bugs, compare all screens against each other:

- font, position and size of tab titles, app bar title alignment, button shape and height, bottom sheet background, loading indicators
- time format, date format, number format
- words used for the same thing (build a glossary and mark every mismatch)
- wording that sounds like failure (for example "we couldn't check", dialog titles containing "failed")
- developer terms, internal names, and old feature names visible on screen
- tap targets smaller than 44px
- text that breaks in the middle of a word or overflows

Check this by actually rendering the screens. Render a matrix of screen widths (390, 360, 320) and in-app text sizes (default, large, extra large). Lock any rule that can be checked automatically into a test.

## 19. Independent hunt with different eyes

Adding one round where a different model, or a fresh agent with no prior context, hunts independently is better than attaching the same model as a reviewer several times.

- Do not show the main hunt's findings. Give only the step 0 coverage table. If the findings are shown first, the reviewer looks only at the same places.
- Merge what the fresh eyes found with what the main hunt found.
- Only then verify each item by reproduction and code evidence.

## Final cross-verification

Re-verify each suspicious point found by one method using the other methods as well.

Merge problems with the same cause into one, and distinguish problems that are actually different. If the same root cause appears in several places, search the whole codebase for that root pattern to find places not yet found.

Where possible, run real tests to confirm reproduction. Where that is not possible, state clearly that the judgment comes from code analysis.

## Reporting

Organize the problems found using these categories.

### Confirmed
A bug that was actually reproduced, or that has clear evidence in the code.

### Suspected
A problem that is likely a bug but could not be reproduced in the current environment.

For each bug, include:

- Severity: Critical / High / Medium / Low
- Title
- Trigger conditions
- Steps to reproduce
- Actual result
- Expected result
- Cause
- Related files / functions
- Impact scope
- Reproduced or not
- Verification level: code only / unit test / screen render / live server or DB call / real device
- Number of the method that found it

At the end, attach:

- The step 0 coverage table (each cell: checked / not checked / verification level)
- For each method, how far it was applied and which areas could not be tested
- The method 20 retrospective table (if anything was missed in a previous hunt)

Separate or exclude simple code style, refactoring suggestions, and performance improvements from bugs. Report method 18 (screen consistency and wording) in its own section, separate from functional bugs.

Do not invent bugs. If the evidence is weak, always classify the item as Suspected.

## 20. Missed-bug retrospective

When a user or another hunt finds a new bug after this hunt, write one line per bug:

```text
bug / which method should have caught it / why it was missed / what to change in this methodology
```

Use this record to revise this methodology. Fill in every column.
