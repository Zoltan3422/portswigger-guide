# 🧪 PortSwigger Labs

This repo contains the solutions for the PortSwigger Labs available in the **Academy** section of their website: https://portswigger.net/web-security/all-labs

## ❓ Why
This repo has been created to keep in a single place all the solutions of the labs. It should be helpful when preparing for the *Burp Suite Certified Practitioner* (https://portswigger.net/web-security/certification).

## 🛠️ Tools
The tools needed (other than Burp Pro) to complete the labs.

- **SQL Injection**: ``sqlmap``;
- **XSS**: ``dalfox``, ``xsstrike``;
- **Clickjacking**: None;
- **DOM-based**: None;
- **CORS**: None;
- **XXE**: None;
- **SSRF**: None;
- **OS Command Injection**: None;
- **Server-Side Template Injection**: None;
- **Directory Traversal**: None;
- **Access Control**: None;
- **Authentication**: None;
- **WebSockets**: None;
- **Web Cache Poisoning**: None;
- **Information Disclosure**: None;
- **OAuth authentication**: None;
- **File Upload Vulnerabilities**: ``ExifTool``;

## ☑️ Stages checklist

| Category | Stage 1 | Stage 2 | Stage 3 |
| --- | --- | --- | --- |
| [[SQL Injection]] |  | [[SQL Injection \| ✔️]] | [[SQL Injection \| ✔️]] |
| [[XSS]] | [[XSS \| ✔️]] | [[XSS \| ✔️]] |  |
| [[CSRF]] | [[CSRF \| ✔️]] | [[CSRF \| ✔️]] |  |
| [[Clickjacking]] | [[Clickjacking \| ✔️]] | [[Clickjacking \| ✔️]] |  |
| [[DOM-based]] | [[DOM-based \| ✔️]] | [[DOM-based \| ✔️]] |  |
| [[CORS]] | [[CORS \| ✔️]] | [[CORS \| ✔️]] |  |
| [[XXE]] |  |  | [[XXE \| ✔️]] |
| [[SSRF]] |  |  | [[SSRF \| ✔️]] |
| [[HTTP request smuggling]] | [[Http request smuggling \| ✔️]] | [[Http request smuggling \| ✔️]] |  |
| [[Os command  injection]] |  |  | [[Os command injection \| ✔️]] |
| [[Server side template injection]] |  |  | [[Server side template injection \| ✔️]] |
| [[Directory traversal]] |  |  | [[Directory traversal \| ✔️]] |
| [[Access-control]] | [[Access-control \| ✔️]] | [[Access-control \| ✔️]] |  |
| [[Authentication]] | [[Authentication \| ✔️]] | [[Authentication \| ✔️]] |  |
| [[Web cache poisoning]] | [[Web cache poisoning \| ✔️]] | [[Web cache poisoning \| ✔️]] |  |
| [[Insecure deserialization]] |  |  | [[Insecure deserialization \| ✔️]] |
| [[Http Host header]] | [[Http host header \| ✔️]] | [[Http host header \| ✔️]] |  |
| [[OAuth authentication]] | [[Oauth authentication \| ✔️]] | [[Oauth authentication \| ✔️]] |  |
| [[File upload vulnerabilities]] |  |  | [[File upload vulnerabilities \| ✔️]] |
| [[JWT]] | [[Jwt \| ✔️]] | [[Jwt \| ✔️]] |  |

## ‼️ Before the exam

- Install every recommended plugin
- Download the username and password wordlists

## ⚠️ During Exam

1. Visit every endpoint
2. *Active scan* every endpoint
3. Search for *.git* directory and use the *Content Discovery* feature built-in
4. Look for inline JavaScript

Futhermore:
- Use the XSS cheatsheet and Stage checklist
- Use the provided payloads

## 💭 Cheat sheets
- [Cross-Site Scripting (XSS) Cheat Sheet - 2023 Edition | Web Security Academy](https://portswigger.net/web-security/cross-site-scripting/cheat-sheet)
- [PayloadsAllTheThings](https://github.com/swisskyrepo/PayloadsAllTheThings)
- [HackTricks](https://book.hacktricks.xyz/welcome/readme)


## 🛫 Recommended plugins

- HTTP Request Smuggler
- Hackvector
- JWT Editor Keys
- Param Miner
- Java Deserialization Scanner
- ActiveScan++

## GitHub repos and websites used
- [Payloads](https://micahvandeusen.com/burp-suite-certified-practitioner-exam-review/)
- [Burp Suite Certified Practitioner Exam Review](https://micahvandeusen.com/burp-suite-certified-practitioner-exam-review/)
- [BSCP Methodology](http://bscpcheatsheet.gitbook.io/)

