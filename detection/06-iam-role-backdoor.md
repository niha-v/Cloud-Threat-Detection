# Detection 06: IAM Role Trust Policy Backdoor

**MITRE ATT&CK:** T1098 – Account Manipulation *(verify on attack.mitre.org; a sub-technique may fit better)*
**Log source:** AWS CloudTrail (management events)
**Severity:** High
**Report:** [IR-006](../incident-reports/IR-006-iam-role-backdoor.md)

## What it detects

An IAM role's trust policy edited so another (often outside) account can assume it. The attacker keeps a way back in that does not depend on any user's credentials.

## Attack simulated

```bash
stratus detonate aws.persistence.iam-backdoor-role
```

## Detection

```spl
index=main eventName=UpdateAssumeRolePolicy
| table _time, userIdentity.arn, requestParameters.roleName, requestParameters.policyDocument, sourceIPAddress, userAgent
| sort _time
```

## Tuning

Trust policy changes are rare. Allow-list the pipeline or admin role that legitimately makes them.

## Alert

Every 5 minutes, trigger when results > 0, severity High.

## Response

1. Restore the previous trust policy.
2. Review `AssumeRole` calls on that role from the new principal.
3. Investigate the actor and how they gained `iam:UpdateAssumeRolePolicy`.
4. `stratus cleanup aws.persistence.iam-backdoor-role`
