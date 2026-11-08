# Chores

Every chore fits in 15 to 30 minutes: one ticket, one page, one alert. Never a project.

## Grounded signals

| Source | Signal | Chore |
|---|---|---|
| Tracker | An issue whose state lags its PRs: in progress with every linked PR merged, or in review with no open PR | Update the ticket |
| Tracker | An open issue assigned to the user, not updated in 30+ days | Update or close it |
| Vault | A note linked from an active task's note, not edited in 6+ months | Tidy the knowledge page |
| Monitoring | An alert that fired 3+ times in the last 7 days and resolved on its own each time | Tune or remove the noisy alert |
| Monitoring | A silence active for more than 7 days | Fix or delete the alert, then drop the silence |
| Monitoring | A firing alert with no runbook link in its annotations | Add the runbook link |
| Slack | The same question asked twice or more in the last 7 days | Write or update the doc page that answers it |
| Slack | A follow-up the user promised in the last 7 days ("I'll update the doc", "will look into it") with no later reply from them | Do it, or close the loop in the thread |

Read monitoring through its alerting API only: alert rules, alert history, silences. No metric
queries.

## Generic chores

When no grounded signal is found, suggest one of these, tied to an active task when one fits
("update the runbook for the OpenTofu migration", not "update a runbook"):

1. Tidy a knowledge page in the vault
2. Update a documentation page, README, or runbook
3. Fix up a noisy or useless alert or dashboard panel
4. Update stale issues or tickets
5. Fix a flaky test or slow CI job
