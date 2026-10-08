# From Honeypot Deployment to SOC Analysis

## AWS Cowrie SSH Honeypot - Threat Activity Review

**Observation window:** 7 August 2026 17:13:58 UTC - 11 August 2026 02:21:36 UTC  
**Evidence reviewed:** Cowrie JSONL telemetry  
**Honeypot service:** SSH on TCP/2222

> This is a follow-up analysis to the AWS Cowrie honeypot deployment. Source IP addresses are partially masked for public sharing, while SRC identifiers keep each source consistent throughout the report.

## Executive Summary

The collected dataset contains **33,181 valid events** across **4,154 Cowrie sessions** from **21 unique source IP addresses**.

The clearest pattern came from **SRC-01 (47.100.xxx.xxx)**, which generated **4,128 sessions (99.37% of all sessions)** and cycled through **4,127 unique password strings** against the `root` account. The median time between connections was approximately **1.70 seconds**, which is strongly consistent with automated password guessing.

A separate source, **SRC-20 (8.138.xxx.xxx)**, continued interacting after authentication. It tested shell behaviour and created temporary-file content under `/tmp`. Cowrie preserved two file-related events with the same SHA-256. These were treated as evidence for further review rather than automatically classified as malware.

Several Windows OpenSSH sessions were slower and more interactive than the automated traffic. Because these may include controlled testing, they are marked **REVIEW** rather than treating every connection as hostile.

**Assessment:** The honeypot captured meaningful suspicious Internet activity, including automated password guessing and post-login interaction. The dataset does not show evidence that the underlying Ubuntu EC2 host was compromised.

## Dataset Snapshot

| Metric | Result |
|---|---:|
| Valid JSON events | 33,181 |
| Cowrie sessions | 4,154 |
| Unique source IPs | 21 |
| Cowrie login successes | 4,133 |
| Cowrie login failures | 1 |
| Command-input events | 4,182 |
| Unique usernames observed | 2 |
| Unique password strings observed | 4,132 |
| Cowrie file events | 2 |

> A **Cowrie login success** means the decoy accepted a credential. It does **not** mean the real Ubuntu SSH service was successfully accessed.

## Finding 1 - Automated Password Guessing

**Source:** SRC-01 (47.100.xxx.xxx)  
**Sessions:** 4,128  
**Unique password strings:** 4,127  
**Median time between connections:** 1.70 seconds  
**Client signature:** `SSH-2.0-Go`  
**Account targeted:** `root`

The first observed credential attempt was `root / 123456`. That attempt was rejected and the SSH session closed.

A new connection then tried a different password, which Cowrie accepted. Only after that accepted decoy login did the source execute:

```bash
echo -e "\x6F\x6B"
```

The command prints `ok`.

The observed sequence was:

**connect -> try a password -> reconnect with another password -> obtain a decoy login -> check the shell -> continue**

Combined with the 1.70-second median connection interval, this is strongly consistent with automated password guessing rather than manual typing.

**MITRE ATT&CK:** T1110.001 Password Guessing and T1059.004 Unix Shell.

## Finding 2 - Post-Authentication Activity

**Source:** SRC-20 (8.138.xxx.xxx)  
**Client:** `SSH-2.0-russh_0.51.1`  
**Session duration:** approximately 301 seconds

After Cowrie accepted the decoy login, the following activity was recorded:

```bash
echo 1 > /dev/null && cat /bin/echo
echo <redacted> > /tmp/.opass
head -c 3800636 > /tmp/DAx98f5J7x
echo <redacted> > /tmp/.opass#UPX!
```

Cowrie recorded two file-creation events with the same SHA-256:

```text
a19db3d3c0a66608e9ab3652aaf2cfde36328111f9b3423d999e2ecdbc9e0ffe
```

Because the content was captured through shell redirection, these are not described as two confirmed malware downloads. The safer conclusion is that the session created file content requiring additional analysis before classification.

## Finding 3 - Possible Controlled Testing

Some Windows OpenSSH sessions were slower and looked more like manual interaction. Commands included `pwd`, `ls`, `whoami`, reading `.bashrc` and `.profile`, creating an `attacker` directory, and an `example.com` wget command.

Those sessions may include controlled testing. Since the logs alone cannot prove ownership of the source addresses, they are marked **REVIEW**. This avoids inflating attack statistics by counting possible lab activity as confirmed malicious behaviour.

## Key Attack Timeline

| UTC | Source | What happened | Priority |
|---|---|---|---|
| 07 Aug 17:13 | SRC-09 | Windows OpenSSH connection; later discovery commands observed | REVIEW |
| 07 Aug 17:14 | SRC-09 | `whoami`, `ls`, `ls -la`, later `ifconfig` | REVIEW |
| 08 Aug 21:43 | SRC-02 | Interactive Windows OpenSSH session begins | REVIEW |
| 08 Aug 21:55 | SRC-02 | Creates `attacker/` directory and `attackplan` file in Cowrie | REVIEW |
| 09 Aug 11:16 | SRC-03 | HTTP request sent to SSH honeypot port - cross-protocol probing | LOW |
| 10 Aug 01:50 | SRC-05 | TLS-like traffic sent to TCP/2222 - service fingerprinting | LOW |
| 10 Aug 08:42 | SRC-20 | Cowrie accepts a root decoy login | HIGH |
| 10 Aug 08:42 | SRC-20 | Shell check and temporary-file activity under `/tmp` | HIGH |
| 10 Aug 20:17 | SRC-01 | First observed `root / 123456` attempt is rejected | HIGH |
| 10 Aug 20:17 | SRC-01 | New connection uses a different password; Cowrie accepts the decoy login | HIGH |
| 10 Aug 20:18 | SRC-01 | Accepted session executes the shell-check command and returns `ok` | HIGH |
| 10 Aug 22:18 | SRC-01 | Automated burst ends after 4,128 sessions | HIGH |

## MITRE ATT&CK Mapping

| Technique | Name | Tactic | Evidence |
|---|---|---|---|
| T1110.001 | Password Guessing | Credential Access | SRC-01 rapidly changed password values against `root` |
| T1059.004 | Unix Shell | Execution | Shell commands were recorded after Cowrie decoy authentication |
| T1033 | System Owner/User Discovery | Discovery | `whoami` was observed |
| T1083 | File and Directory Discovery | Discovery | `ls`, `ls -la` and shell-profile reads were observed |
| T1016 | System Network Configuration Discovery | Discovery | `ifconfig` was observed |

## Top Source Indicators

| IOC | Masked IP | Sessions (Connections) | Cowrie Login Successes | Commands | Priority |
|---|---|---:|---:|---:|---|
| SRC-01 | 47.100.xxx.xxx | 4,128 | 4,126 | 4,126 | HIGH |
| SRC-02 | 102.93.xxx.xxx | 2 | 2 | 44 | REVIEW |
| SRC-03 | 167.99.xxx.xxx | 2 | 0 | 0 | LOW |
| SRC-04 | 40.119.xxx.xxx | 2 | 0 | 0 | LOW |
| SRC-05 | 94.231.xxx.xxx | 2 | 0 | 0 | LOW |
| SRC-06 | 20.98.xxx.xxx | 2 | 0 | 0 | LOW |
| SRC-07 | 20.65.xxx.xxx | 2 | 0 | 0 | LOW |
| SRC-08 | 45.156.xxx.xxx | 1 | 0 | 0 | LOW |
| SRC-09 | 105.127.xxx.xxx | 1 | 1 | 7 | REVIEW |
| SRC-10 | 120.26.xxx.xxx | 1 | 1 | 0 | MEDIUM |

**Sessions (Connections)** = separate connections made to Cowrie.  
**Cowrie Login Successes** = sessions where the Cowrie decoy accepted a credential.  
**Commands** = command-input events recorded by Cowrie.

## Recommended Actions and Escalation

Based on the observed activity, the following actions are recommended for consideration by Security Operations and Infrastructure teams:

1. Continue monitoring the honeypot for repeated login attempts, unusual commands, new source indicators and file activity.
2. Create or tune alerting for unusually high authentication volumes.
3. Keep a record of approved test IP addresses and test periods so future lab activity can be separated from unsolicited Internet traffic.
4. Keep administrative SSH on port 22 restricted to authorised sources and maintain separation between the real server and the honeypot.
5. Preserve relevant logs, timestamps, session IDs and file hashes when suspicious activity is identified.
6. Refer suspicious captured files for further analysis in a safe environment rather than executing them directly.
7. Escalate sustained credential attacks or unusual post-login behaviour to a senior analyst for deeper investigation and correlation with other security logs.

## Limitations

- This review was based mainly on Cowrie telemetry. External threat-intelligence services were not used to enrich the source IP addresses.
- Some sessions may include controlled testing; those were marked REVIEW where behaviour suggested manual activity.
- A Cowrie login success is a decoy authentication result and does not prove access to the real EC2 host.
- The two Cowrie file events share one hash and were produced through shell redirection; they were not verified as malware downloads.
- Source IP addresses are partially masked in this public version.

## Conclusion

The main lesson from the analysis was that connection volume alone is not enough. Following activity from connection, to authentication, to command execution provided a clearer picture of what the logs actually supported.

Separating possible controlled testing from unsolicited traffic was also important. Without that step, lab activity could have been incorrectly reported as attacker behaviour.

**Final conclusion:** The suspicious activity reviewed here was directed at the Cowrie honeypot. Based on this dataset, there is no evidence that the underlying AWS EC2 host was compromised.
