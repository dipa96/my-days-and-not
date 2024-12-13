# My days and not

Analysis of vulnerabilities from **Security Audit** || **Bug Bounty** || **Security advisories** || **CTF**.

## CVE Analysis list

| Name           | Field            | Vulnerability           | Proof of Concept(PoC)                                                               
|----------------|------------------|-------------------------|-------------------------------------------------------------------------------------
| CVE-2021-43849 | Mobile - Android | Denial of Service (DoS) | [Link 2 PoC](/CVE-2021-43849/README.md)                                             
| CVE-2022-2071  | Web Application  | CSRF + XSS              | [Link 2 PoC](https://wpscan.com/vulnerability/d3653976-9e0a-4f2b-87f7-26b5e7a74b9d) 
| CVE-2022-2072  | Web Application  | XSS                     | [Link 2 PoC](https://wpscan.com/vulnerability/3014540c-21b3-481c-83a1-ce3017151af4) 
| CVE-2022-3241  | Web Application  | SQL Injection(SQLi)     | [Link 2 PoC](https://wpscan.com/vulnerability/a995dd67-43fc-4087-a7f1-5db57f4c828c) 
| CVE-2022-3860  | Web Application  | SQL Injection(SQLi)     | [Link 2 PoC](https://wpscan.com/vulnerability/d99ce21f-fbb6-429c-aa3b-19c4a5eb7557)
| CVE-2023-4724  | Web Application  | SQL Injection(SQLi)     | [Link 2 PoC](https://www.unlock-security.it/it/security-advisory/cve-2023-4724-cve-2023-5882-wp-all-export/)
| CVE-2023-5882  | Web Application  | SQL Injection(SQLi)     | [Link 2 PoC](https://wpscan.com/vulnerability/72be4b5c-21be-46af-a3f4-08b4c190a7e2/)
| CVE-2024-XXXX  | Mobile - Android | Open arbitrary URLs     | WiP
| CVE-2024-23710 | Mobile - Android | Elevation of Priv (EoP) | WiP

## CTF Writeups
| Name           | Field            | Vulnerability           | Writeup                                                                            | Platform
|----------------|------------------|-------------------------|------------------------------------------------------------------------------------|------------
| ConfigEditor   | Mobile - Android | Java Deserialization    | [Link 2 Writeup](/CTFs/ConfigEditor/README.md)                                          | [MHL](https://www.mobilehackinglab.com/course/lab-config-editor-rce) |
| Europa | Web Application | SQLi, preg_replace() | [Link 2 Writeup](https://gist.github.com/dipa96/16fbbc204d8d7daac581ed52c421d363) | [HTB](https://app.hackthebox.com/machines/27) |
| Bank | Web Application | File Upload | [Link 2 Writeup](https://gist.github.com/dipa96/d509ea39d1c00dcf5e736a8b72885ee6) | [HTB](https://app.hackthebox.com/machines/Bank) |
