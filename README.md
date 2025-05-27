

# Web Security Academy: Blind SQLi - OOB DNS Exploit Script

**⚠️ Important: This script is intended for educational purposes only. Do not use it for any unauthorized or malicious activities. You are solely responsible for your actions. Misuse is strictly prohibited. ⚠️**

## Overview

This Python script is designed to help understand and solve the "Blind SQL injection with out-of-band interaction" lab from PortSwigger's Web Security Academy.

It's particularly aimed at learners who may not have a Burp Suite Professional subscription (which provides the Burp Collaborator client) but wish to understand and complete the lab using an alternative out-of-band (OAST) approach. This script demonstrates how **assigning a random string to a `burpcollaborator.net` subdomain can allow you to clear this lab.**

## How This Script Clears the Lab: An Educational Explanation

The goal of this lab is to exploit a blind SQL injection vulnerability to cause the target database server to perform an out-of-band DNS lookup to a Burp Collaborator domain.

### 1. Understanding Blind SQL Injection
In a typical SQL injection, the attacker might see errors or data directly in the web application's response. With "blind" SQL injection, there's no such direct feedback. The application behaves normally, even if the injection is successful. This means we need another way to confirm that our injected SQL commands were executed.

### 2. Out-of-Band Application Security Testing (OAST)
OAST is a technique used to detect and confirm blind vulnerabilities. It involves sending a payload that causes the application to interact with an external system that we can monitor. If we see an interaction (like a DNS query or HTTP request) on our external system, it confirms the vulnerability.

### 3. The DNS-Based OAST Technique Used by This Script
* **Crafting the SQL Injection Payload**: The script generates a specific SQL injection payload. This payload is designed for Oracle databases (which is commonly used in labs demonstrating this technique) and uses the `EXTRACTVALUE(xmltype(...))` function.
* **Triggering the DNS Lookup**: Inside the `xmltype` constructor, a specially crafted XML string is included. This XML string contains an external entity reference (e.g., `<!ENTITY % remote SYSTEM "http://[RANDOM_SUBDOMAIN].burpcollaborator.net/">`). When the database processes this malicious XML (due to the SQL injection), it attempts to resolve the external entity by making an HTTP request. The very first step to making an HTTP request to a domain is to resolve its name to an IP address – this triggers a **DNS lookup** for `[RANDOM_SUBDOMAIN].burpcollaborator.net`.
* **The Role of `burpcollaborator.net` and the Random Subdomain**:
    * The lab explicitly requires interaction with "Burp Collaborator's default public server."
    * This script automatically generates a **random string** (e.g., `a1b2c3d4e5f6.burpcollaborator.net`) to use as the subdomain.
    * **Why this works without a Burp Pro subscription for *this* lab**: The Domain Name System (DNS) is hierarchical. Any DNS query for *any* subdomain of `burpcollaborator.net` (like our randomly generated one) will ultimately be routed to PortSwigger's authoritative DNS servers for the `burpcollaborator.net` domain. The Web Security Academy's lab environment is likely configured to monitor for *any* DNS lookups originating from its lab instances that are directed towards *any* host within the `*.burpcollaborator.net` domain. If such a lookup is detected (and possibly if the payload structure matches the lab's expected solution), the lab platform registers it as a successful attempt. For this particular lab's validation mechanism, it seems the *act* of causing a DNS lookup to the `burpcollaborator.net` superdomain via the correct SQLi technique is sufficient, rather than requiring interaction with a unique subdomain specifically registered and monitored via a Burp Pro Collaborator client session.
* **What This Script Automates**:
    * Attempting to automatically fetch initial session and `TrackingId` cookies from the lab URL.
    * Generating a random subdomain for `burpcollaborator.net`.
    * Constructing the specific SQL injection payload incorporating this OAST domain.
    * Sending the HTTP request with the malicious `TrackingId` cookie to the lab.

### 4. Confirmation of Lab Completion
* This script *sends* the payload designed to trigger the out-of-band DNS lookup. However, **the script itself does not have a built-in OAST listener** to confirm that the DNS lookup actually occurred on the Collaborator server.
* You can confirm that you have successfully solved the lab when you see the message **"Congratulations, you solved the lab!"** displayed on the Web Security Academy lab page itself. This indicates the lab platform detected the DNS interaction triggered by your script.

## Prerequisites
* Python 3 installed.
* The `requests` library installed. If not, run:
    ```bash
    pip install requests
    ```

## How to Use
1.  Save the script to a file (e.g., `lab_solver.py`).
2.  Open your terminal or command prompt.
3.  Navigate to the directory where you saved the file.
4.  Run the script using:
    ```bash
    python lab_solver.py
    ```
5.  When prompted, enter the full URL of the Web Security Academy lab you are attempting.

## Final Reminder
This script is provided for educational purposes to help you understand the mechanics of out-of-band SQL injection. The concepts learned here are valuable, but always use such knowledge responsibly and ethically. Never test on systems you do not have explicit permission to assess.
