# MITRE ATT&CK Coverage

> Technique IDs were mapped by hand and should be re-verified against [attack.mitre.org](https://attack.mitre.org) before relying on them.

| # | Detection | Tactic | Technique | Status |
|---|---|---|---|---|
| 01 | Backdoor IAM admin user | Persistence | T1136.003 – Create Account: Cloud Account | ✅ Built |
| 02 | CloudTrail logging stopped | Defense Evasion | T1562.008 – Disable or Modify Cloud Logs | ✅ Built |
| 03 | S3 bulk object download | Collection | T1530 – Data from Cloud Storage | ✅ Built |
| 04 | Console login without MFA | Initial Access | T1078.004 – Valid Accounts: Cloud Accounts | ✅ Built |
| 05 | S3 bucket policy backdoor | Exfiltration | T1537 – Transfer Data to Cloud Account | ✅ Built |
| 06 | IAM role trust policy backdoor | Persistence | T1098 – Account Manipulation | ✅ Built |
| 07 | CloudTrail event selectors changed | Defense Evasion | T1562.008 – Disable or Modify Cloud Logs | ✅ Built |
| 08 | VPC Flow Logs deleted | Defense Evasion | T1562.008 – Disable or Modify Cloud Logs | ⬜ Planned |
| 09 | AccessDenied burst from one identity | Discovery | T1580 – Cloud Infrastructure Discovery | ⬜ Planned |
| 10 | Root account activity | Initial Access | T1078.004 – Valid Accounts: Cloud Accounts | ⬜ Planned |
| 11 | Secrets Manager mass retrieval | Credential Access | T1555.006 – Cloud Secrets Management Stores | ⬜ Planned |

**Legend:** ✅ built · ⬜ planned
