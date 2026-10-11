# Incident Report IR-006: IAM Role Trust Policy Backdoor

| Field | Value |
|---|---|
| **Incident ID** | IR-006 |
| **Date detected** | 2026-10-11 (TODO: confirm) |
| **Severity** | High |
| **MITRE technique** | T1098 – Account Manipulation *(verify)* |
| **Status** | Resolved (lab) |

## Summary

An IAM role's trust policy was edited so another account can assume it, giving an attacker a way back in that does not depend on any user's credentials. The activity was simulated with Stratus Red Team (`aws.persistence.iam-backdoor-role`) in an isolated lab account and detected in Splunk from CloudTrail logs.

## Timeline (UTC)

| Time | Event | Detail |
|---|---|---|
| TODO | `UpdateAssumeRolePolicy` | TODO: copy `_time`, actor ARN and source IP from the Splunk result |

## Indicators

- **Event:** `UpdateAssumeRolePolicy`
- **Actor:** TODO
- **Source IP:** TODO
- **User agent:** `stratus-red-team_*` (simulation marker)

## Detection

See [detection 06](../detections/06-iam-role-backdoor.md).

## Impact

Lab only. In a real account, an outside account could assume the role and use its permissions.

## Response actions

1. Restore the previous trust policy.
2. Review `AssumeRole` calls on the role from the new principal.
3. Investigate the actor.
4. `stratus cleanup aws.persistence.iam-backdoor-role`

## Lessons

- Trust policy changes are rare, so a simple event-name alert is high fidelity.
- Preventive fix: restrict `iam:UpdateAssumeRolePolicy` and review roles that trust outside accounts.
