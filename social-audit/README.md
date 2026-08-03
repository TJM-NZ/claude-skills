# Social Audit

Social hook and stickability audit — checks for missing retention mechanics, weak onboarding, notification problems, dark patterns, and social graph gaps.

## What It Does

Reviews whether an app has the mechanics to make users return and stay. Based on the Hook model (trigger → action → variable reward → investment) and behavioural psychology research.

Checks:

- **Triggers** — external (push/email) vs internal (habit), re-engagement logic, notification personalisation and frequency
- **Action** — friction between trigger and reward, permission gates before value, social graph seeding at signup
- **Variable Reward** — feed algorithm vs chronological, social proof (likes/reactions), streaks, progress/mastery signals
- **Investment** — content history, reputation, social graph — switching cost visibility
- **Onboarding** — aha moment tracking, empty state handling, flow length, social graph population
- **Dark Patterns** — infinite scroll without limits, autoplay, streak pressure without grace (regulatory risk: EU DSA)

## When to Use

- "Users aren't coming back" — retention/habit mechanics missing
- "Engagement is low" — reward or friction issues
- "Notifications aren't working" — spam or personalisation problems
- "Users churn after day 1" — onboarding and empty state gaps
- "Is this too manipulative?" — dark pattern audit

## Severities

- `HOLLOW` — missing mechanic; no reason to return
- `COLD` — empty graph or no value on day 1; users churn before forming habit
- `LEAK` — friction between trigger and reward
- `SPAM` — notification pattern that will destroy permission
- `DARK` — manipulative pattern; regulatory or ethical liability

## Output

```
[SEVERITY] file:line — what's missing. what it costs in retention. what to build.
```

Blunt. Names the component, shows the gap, states the fix with retention context.
