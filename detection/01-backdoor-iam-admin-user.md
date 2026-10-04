# Detection: Backdoor IAM Admin User Created

**MITRE ATT&CK:** T1136.003 – Create Account: Cloud Account
(supporting: T1098 – Account Manipulation)

<img src = "https://github.com/niha-v/Cloud-Threat-Detection/blob/main/Image/01.png" width = 1000>

**Log source:** AWS CloudTrail
**Severity:** High

---

## What it detects

An attacker establishing persistence by creating a new IAM user, attaching
`AdministratorAccess`, and generating an access key — all within seconds. This is a
classic cloud persistence pattern: a quiet backdoor identity that survives even if the
original foothold is lost.

## Attack simulated

```bash
stratus detonate aws.persistence.iam-create-admin-user
```

Observed CloudTrail events (single actor, seconds apart):
- `CreateUser` → `malicious-iam-user`
- `AttachUserPolicy` → `arn:aws:iam::aws:policy/AdministratorAccess`
- `CreateAccessKey`

---

## Detection — investigation table

```spl
index=main eventName=CreateUser OR eventName=AttachUserPolicy OR eventName=CreateAccessKey
| table _time, eventName, userName, sourceIPAddress, userAgent
| sort _time
```

## Detection — high-fidelity (admin policy attached)

```spl
index=main eventName=AttachUserPolicy
    requestParameters.policyArn="arn:aws:iam::aws:policy/AdministratorAccess"
| spath output=target_user path=requestParameters.userName
| spath output=actor path=userIdentity.arn
| table _time, actor, target_user, sourceIPAddress
```

> Note: a real attacker won't use a `stratus-red-team` user agent. The durable signal is
> the **behavior** — admin policy attached immediately after account creation — not the tool name.

---

## Alert configuration

- **Save As → Alert**
- Schedule: every 5 minutes
- Trigger: number of results > 0
- Severity: High
- Action: (lab) log / notify

## Response

1. Disable/delete the created IAM user.
2. Revoke the generated access key.
3. Review what the key accessed before revocation.
4. Confirm teardown: `stratus cleanup aws.persistence.iam-create-admin-user`
