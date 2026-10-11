# Incident Report IR-004: Console Login Without MFA

| Field | Value |
|---|---|
| **Incident ID** | IR-004 |
| **Date detected** | 2026-10-11 (TODO: confirm) |
| **Severity** | Medium |
| **MITRE technique** | T1078.004 – Valid Accounts: Cloud Accounts *(verify)* |
| **Status** | Resolved (lab) |

## Summary

An IAM user signed in to the AWS console without multi-factor authentication, so a password alone was enough to gain access. The activity was simulated with Stratus Red Team (`aws.initial-access.console-login-without-mfa`) in an isolated lab account and detected in Splunk from CloudTrail logs.

## Timeline (UTC)

| Time | Event | Detail |
|---|---|---|
| TODO | `ConsoleLogin` | TODO: copy `_time`, actor ARN and source IP from the Splunk result |

## Indicators

- **Event:** `ConsoleLogin`
- **Actor:** TODO
- **Source IP:** TODO
- **User agent:** `stratus-red-team_*` (simulation marker)

## Detection

See [detection 04](../detections/04-console-login-without-mfa.md).

## Impact

Lab only. In a real account, an MFA-less login to a privileged user is an easy path to account takeover.

## Response actions

1. Confirm with the user whether they signed in.
2. Force a password reset and require MFA.
3. Review activity after the login.
4. `stratus cleanup aws.initial-access.console-login-without-mfa`

## Lessons

- Sign-in events are a strong early signal; MFA status is recorded in `additionalEventData.MFAUsed`.
- Preventive fix: require MFA by policy for all console users.
