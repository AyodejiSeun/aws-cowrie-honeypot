# aws-cowrie-honeypot
Deploying a Cowrie SSH honeypot on AWS EC2 to capture, analyze, and document real-world SSH scanning and authentication activity.
# AWS Cowrie SSH Honeypot

## Project Overview

I built and deployed a Cowrie SSH honeypot on AWS EC2 to observe
real-world SSH scanning, credential attempts, reconnaissance activity,
and attacker interaction with a controlled decoy environment.

## Architecture

Internet
    |
    | TCP 2222
    v
AWS Security Group
    |
    v
Ubuntu EC2
    |
    +---- TCP 22 ----> Real SSH Administration
    |
    +---- TCP 2222 --> Cowrie Honeypot

## Security Configuration

TCP 22:
Restricted to my administrative IP address.

TCP 2222:
Exposed to the Internet for Cowrie SSH honeypot traffic.

## Deployment

1. Launched Ubuntu EC2 instance
2. Configured AWS Security Group
3. Connected to EC2 using SSH and PEM key
4. Installed Python and Git
5. Created dedicated Cowrie user
6. Cloned Cowrie
7. Created Python virtual environment
8. Installed and initialized Cowrie
9. Configured Cowrie on TCP port 2222
10. Performed controlled SSH testing
11. Collected and analyzed Cowrie JSON logs
12. Preserved evidence and calculated hashes

## Connecting to the Real EC2 Server

Administrative access was performed over TCP port 22:

ssh -i "myhoneypotkey.pem" ubuntu@<EC2-PUBLIC-IP>

TCP 22 was restricted through the AWS Security Group.

## Cowrie Honeypot

Cowrie was configured to listen on:

tcp:2222:interface=0.0.0.0

A controlled test was performed using:

ssh -p 2222 admin1@<EC2-PUBLIC-IP>

## Observations

The honeypot recorded:

- 54 connection events
- 25 successful decoy logins
- 40 commands executed
- 25 unique source IP addresses
- 0 file downloads

Observed behaviour included credential scanning,
system reconnaissance, SSH probing, and service discovery.

## Evidence Preservation

Cowrie logs, TTY sessions, and downloaded-file directories were
archived and hashed using SHA-256.

## Key Lessons

This project helped me gain practical experience with:

- AWS EC2
- AWS Security Groups
- Linux administration
- SSH
- Cowrie
- Honeypot deployment
- JSON log analysis
- Threat hunting
- Evidence preservation
- Basic attacker-behaviour analysis

## Disclaimer

This project was conducted in an isolated cloud environment for
educational and cybersecurity research purposes.

Sensitive information, administrative IP addresses, and identifying
information have been removed or sanitized.
