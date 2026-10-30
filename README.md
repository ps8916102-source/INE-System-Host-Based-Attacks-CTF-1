# INE – Host & Network Penetration Testing: System-Host Based Attacks CTF 1

## Overview

This repository documents my solution to the **System-Host Based Attacks CTF 1** Skill Check Lab from INE's Host & Network Penetration Testing course.

The lab involved attacking two Windows targets by identifying weak credentials, enumerating web directories, exploiting writable WebDAV access, deploying an ASP webshell, and accessing Windows files through SMB.

## Lab Objectives

| Flag | Objective | Target |
|---|---|---|
| Flag 1 | Discover weak HTTP credentials for user `bob` and retrieve the flag | target1.ine.local |
| Flag 2 | Explore the C: drive and retrieve the flag using an ASP webshell | target1.ine.local |
| Flag 3 | Guess SMB credentials and discover the next flag | target2.ine.local |
| Flag 4 | Enumerate the Administrator's Desktop and retrieve the flag | target2.ine.local |

## Tools Used

- Nmap
- Hydra
- Gobuster
- Cadaver
- SMBClient
- Kali Linux
- ASP Webshell

## Repository Structure

```text
INE-System-Host-Based-Attacks-CTF-1/
│
├── README.md
└── lab-notes.md
```

## Walkthrough

Detailed commands, explanations, enumeration findings, and the flag retrieval process are documented in:

**[lab-notes.md](lab-notes.md)**

## Key Learning Outcomes

- Service and version enumeration using Nmap.
- HTTP authentication testing with Hydra.
- Web directory enumeration using Gobuster.
- WebDAV enumeration and file upload using Cadaver.
- ASP webshell deployment and browser-based command execution.
- SMB credential testing and share enumeration.
- Accessing Windows administrative shares.
- Navigating Windows directories and downloading files through SMB.

## Disclaimer

This repository is intended for educational purposes and documents activities performed in an authorized INE lab environment.

All techniques should only be used against systems for which explicit permission has been granted.
