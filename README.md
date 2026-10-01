# 🛡️ AWS Cowrie SSH Honeypot

> Deploying a Cowrie SSH honeypot on AWS EC2 to capture, analyze, and document real-world SSH scanning and authentication activity.

## Project Overview

As part of my cybersecurity learning journey, I wanted to move beyond analyzing prepared datasets and build a live environment where I could observe how Internet-facing systems are discovered and probed.

The idea was inspired by a honeypot analysis project I completed during my cybersecurity internship with the Ubuntu Bridge Initiative.

I initially attempted to deploy T-Pot on AWS. However, the limited resources available on the EC2 instance caused problems with Docker and the services required by T-Pot.

Rather than abandoning the project, I changed my approach and deployed **Cowrie**, a lightweight SSH/Telnet honeypot designed to log brute-force attempts and attacker interactions.

The final environment allowed me to observe unsolicited SSH activity, perform controlled testing, analyze Cowrie JSON logs, and practice evidence preservation.


## Project Objectives

The objectives of this project were to:

- Deploy an Internet-facing honeypot in AWS.
- Secure administrative access to the underlying server.
- Separate real SSH administration from the honeypot service.
- Capture SSH connection and authentication activity.
- Observe commands executed inside the decoy environment.
- Analyze Cowrie JSON logs.
- Distinguish controlled testing from unsolicited Internet activity.
- Preserve collected evidence safely.
- Develop practical SOC and threat-analysis skills.



## Architecture

The environment was designed so that the real Ubuntu server and the Cowrie honeypot used different SSH ports.


                    Internet
                       |
                       |
                AWS Security Group
                  /           \
                 /             \
        TCP Port 22          TCP Port 2222
             |                    |
             |                    |
      Real Ubuntu EC2       Cowrie Honeypot
             |                    |
     Administrative          Decoy SSH Service
         Access             / Attacker Activity
             |
      Restricted to
    Administrator IP
