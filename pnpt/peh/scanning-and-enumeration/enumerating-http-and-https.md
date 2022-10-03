---
description: Web Enumeration
---

# Enumerating HTTP and HTTPs

## Methodology

1. Run Nmap
2. What versions are running? -- Information disclosure
3. What programming language is the server running?
4. What does Wappalyzer report?
5. Document findings in notes
6. Vulnerability scanning w/ Nikto -- `nikto -h http://local.htb`
7. Directory Bruteforce -- feroxbuster, gobuster, dirsearch, etc.
8. View source code of all web pages -- Look for anything interesting

## Simplified

* Target the low-hanging fruit first

What to enumerate:

Service versions -> Directories -> Source Code -> Nikto Vulnerability Scan
