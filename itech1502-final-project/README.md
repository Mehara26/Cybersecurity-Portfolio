# Vulnerability Scanning with Nmap, Nikto and OpenVAS

## Objective

Run a first vulnerability check on an authorised TryHackMe lab machine, the way a security analyst would at the start of a job. Then turn the raw scan output into rated findings and practical fixes. This was my ITECH1502 Cybersecurity Fundamentals final project at Federation University. The full write-up is in [report.pdf](report.pdf).

### Skills Learned

- Finding open ports and service versions on a target
- Checking a web server for misconfigurations and exposed files
- Reading CVE identifiers and CVSS severity ratings
- Ranking findings by risk and recommending fixes
- Treating scanner results as clues to check, not proof
- Scanning only systems I have permission to test

### Tools Used

- Nmap, for host discovery, port and version scanning, and the `ftp-anon` script
- Nikto, for web server checks
- OpenVAS (Greenbone), for CVE and CVSS ratings
- TryHackMe "Vulnerability Scanning Tools" room (all 7 tasks completed, 100%)

## Steps

I started with quiet checks and moved to deeper ones, choosing each step from the last result. Nothing was exploited, so this covers scanning and the first stages of a penetration test (discovery and enumeration). All screenshots are full screen, with the TryHackMe room on the left and the lab machine on the right.

### Part 1: Nmap

1. Checked the lab machine was up with `nmap -sn <target IP>`.

   ![Nmap host discovery](screenshots/01-nmap-host-discovery.jpg)

   *Ref 1: `nmap -sn` shows the host is up.*

2. Found open ports and service versions with `nmap -sV <target IP>`. Five ports were open (21 FTP, 22 SSH, 80 HTTP, 5901 VNC, 8080 HTTP) and 995 were closed.

   ![Nmap service scan](screenshots/02-nmap-service-scan.jpg)

   *Ref 2: `nmap -sV` output listing the five open ports and their versions.*

3. Got more detail with the aggressive scan `nmap -A <target IP>`. It showed that anonymous FTP login is allowed and that FTP sends data in plain text.

   ![Nmap aggressive scan](screenshots/03-nmap-aggressive-scan.jpg)

   *Ref 3: `nmap -A` output with the FTP status and "Anonymous FTP login allowed (FTP code 230)".*

4. Tested FTP directly with `nmap --script=ftp-anon -p21 <target IP>`.

   ![Nmap ftp-anon](screenshots/04-nmap-ftp-anon.jpg)

   *Ref 4: The ftp-anon script confirms anonymous FTP login is allowed (code 230).*

### Part 2: Nikto

5. Nmap found a website on port 8080, so I checked it with `nikto -h http://<target IP>:8080`. Only that one port was scanned. Nikto ran 6,544 checks in about 8 seconds and reported 7 items.

   ![Nikto](screenshots/05-nikto-port-8080.jpg)

   *Ref 5: Nikto output for port 8080 (6,544 checks, 7 items).*

### Part 3: OpenVAS

6. Opened OpenVAS inside the lab machine's browser (127.0.0.1 means "this computer") and signed in with the room's default login.

   ![OpenVAS login](screenshots/06-openvas-login.jpg)

   *Ref 6: OpenVAS sign-in page.*

   ![OpenVAS dashboard](screenshots/07-openvas-dashboard.jpg)

   *Ref 7: OpenVAS dashboard after signing in.*

7. Added my own target (the lab machine's IP) and created a scan task.

   ![New target](screenshots/08-openvas-new-target.jpg)

   *Ref 8: Creating "SecondTarget".*

   ![Targets list](screenshots/09-openvas-targets-list.jpg)

   *Ref 9: The targets list, including SecondTarget.*

   ![Tasks](screenshots/10-openvas-tasks.jpg)

   *Ref 10: The Tasks page showing the pre-scanned MyTarget task (6.4 Medium).*

   ![New task](screenshots/11-openvas-new-task.jpg)

   *Ref 11: Creating "SecondTask".*

8. My scan could not run: OpenVAS said "Feed is currently syncing. Please try again later." The room had warned that scans need internet. So I studied the room's pre-scanned report instead.

   ![Feed syncing](screenshots/12-openvas-feed-syncing.jpg)

   *Ref 12: The "Feed is currently syncing" message on my new task.*

9. Read the MyTarget report: overall 6.4 (Medium), with 0 High, 4 Medium and 3 Low results.

   ![Report results](screenshots/13-openvas-report-results.jpg)

   *Ref 13: Report Results tab, Anonymous FTP (6.4) and phpinfo() (5.3).*

   ![Report CVEs](screenshots/14-openvas-report-cves.jpg)

   *Ref 14: CVEs tab linking each finding to public CVE IDs and scores.*

   ![Report operating systems](screenshots/15-openvas-report-os.jpg)

   *Ref 15: Operating Systems tab showing Ubuntu 24.04, matching what Nmap saw.*

### Completion

10. Finished all 7 tasks and answered the room's questions.

    ![Room completed](screenshots/16-room-completed-page.jpg)

    *Ref 16: "Room completed!" page (7 tasks completed).*

    ![Room 100%](screenshots/17-room-100-percent.jpg)

    *Ref 17: Room page showing "Room completed: 100%" with all tasks ticked.*

## Results

| Finding | Found by | Severity | Recommended fix |
| --- | --- | --- | --- |
| Anonymous FTP login (port 21) | Nmap, OpenVAS | 6.4 Medium | Disable anonymous access; use SFTP or FTPS |
| phpinfo() page at /info.php | Nikto, OpenVAS | 5.3 Medium | Remove it from production |
| Session cookie without HttpOnly | Nmap, Nikto | Not scored | Set HttpOnly and Secure flags |
| Possible RFI in info.php?file= | Nikto only | Unverified | Validate input; check by hand first |

Scores are the CVSS values OpenVAS gave. Where there was no score, the priority is my own judgement.

## Reflection

- **What I learned:** scanners give clues, not proof, and a finding reported by two tools is more likely to be real.
- **Challenges:** OpenVAS would not open from my own browser because 127.0.0.1 means "this computer". The lab machine restarted and its IP changed. My OpenVAS scan could not run without internet.
- **Next time:** run a full OpenVAS scan with internet access, check one finding by hand, re-scan to show the risk dropped, and revise networking basics before starting.
