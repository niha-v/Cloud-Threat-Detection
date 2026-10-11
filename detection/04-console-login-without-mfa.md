# Detection 04: Console Login Without MFA

**MITRE ATT&CK:** T1078.004 – Valid Accounts: Cloud Accounts *(verify on attack.mitre.org)*
**Log source:** AWS CloudTrail (sign-in events)
**Severity:** Medium (High for privileged users)
**Report:** [IR-004](../incident-reports/IR-004-console-login-without-mfa.md)

## What it detects

A successful AWS console sign-in that did not use multi-factor authentication. A stolen or guessed password is enough to get in, so MFA-less logins are worth reviewing.

## Attack simulated

```bash
stratus detonate aws.initial-access.console-login-without-mfa
```

## Detection

```spl
index=main eventName=ConsoleLogin additionalEventData.MFAUsed=No
| table _time, userIdentity.arn, userIdentity.type, sourceIPAddress, userAgent, responseElements.ConsoleLogin
| sort _time
```

Successful logins only:

```spl
index=main eventName=ConsoleLogin additionalEventData.MFAUsed=No responseElements.ConsoleLogin=Success
| stats count by userIdentity.arn, sourceIPAddress
```

## Tuning

Exclude known service or break-glass accounts by ARN once identified. Do not exclude by source IP alone.

## Alert

Every 5 minutes, trigger when results > 0, severity Medium.

## Response

1. Confirm with the user whether they signed in.
2. Force a password reset and require MFA.
3. Review activity after the login.
4. `stratus cleanup aws.initial-access.console-login-without-mfa`
